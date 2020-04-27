---
layout: post
title:  "Scouter Server Plugin를 통한 Xlog 수집"
description: "Scouter Server Plugin으로 Scouter Server의 Xlog 정보를 수집하고 구글 시트로 작성하기"
date:   2020.04.27.
writer: "하선호"
categories: Common
---

## 개요
현재 다나와에서는 JAVA WAS에 대한 APM으로 스카우터를 사용하고 있습니다.
그 중 스카우터의 XLOG 차트는 해당 WAS의 상황을 바로 알 수 있는데요.
하지만 XLOG 차트를 계속 보고 있을 수는 없고 일일히 XLOG를 조회해서 보기도 불편할 것 같습니다.
그래서 XLOG 정보를 수집하고 문서화 하는 플러그인을 만들어보았습니다.

[스카우터 XLOG]
![/images/2020-04-27-Scouter-Plugin/Xlog.png](/images/2020-04-27-Scouter-Plugin/Xlog.png)

## 스카우터 서버 플러그인
스카우터 서버 플러그인은 Scripting plugin과 Built-in Plugin으로 나눠지는데 
Scripting plugin은 수집데이터가 DB에 저장되기 전 호출되며 코드 변경이 동적으로 반영되어 운영중인 서버에 즉시 반영할 수 있지만 간단한 전처리 기능정도만 활용이 가능합니다.
Built-in Plugin은 Scouter에서 미리 제공하는 annotation을 사용하여 더 많은 기능들을 구현할 수 있습니다.
여기서는 Built-in Plugin 방식으로 개발하였습니다.

## Built-in Plugin 작성하기

### 주의점
- Annotation scan은 scouter.plugin.server 패키지 하위에서 수행하므로 scouter.plugin.server 하위에 작성되어야 합니다.
- Scouter server의 configuration을 이용할 수 있으며 이때 ext_plugin_xxx 로 시작되어야 합니다.

### 목표
1. 에러와 N초 이상이 걸린 요청들을 수집한 후 특정 주기마다 구글 시트로 작성합니다.
2. 구글시트는 각 서비스별로 탭이 생성되고 시간, 에러여부 타입, 요청된 URL, 응답시간, 페이퍼링크가 기록됩니다.


### 플러그인 기본 구조
기본적으로 scouter.common, scouter.server가 dependency되어야 합니다.

```
<dependencies>
    <dependency>
        <groupId>io.github.scouter-project</groupId>
        <artifactId>scouter-common</artifactId>
        <version>1.8.3</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>io.github.scouter-project</groupId>
        <artifactId>scouter-server</artifactId>
        <version>1.8.3</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

아래는 스카우터 Github에 있는 예제입니다.
스카우터 서버에서 Alert이 발생할 때마다 로그가 찍히는 예제이고 저희가 개발할 플러그인의 목적은 XLOG를 수집하는 것이니 XLOG Annotation을 사용해야 합니다.
```
package scouter.plugin.server.none;
public class NullPlugin {
    Configure conf = Configure.getInstance();

	@ServerPlugin(PluginConstants.PLUGIN_SERVER_ALERT)
    public void alert(AlertPack pack){
        if(conf.getBoolean("ext_plugin_null_alert_enabled", true)) {
            println("[NullPlugin-alert] " + pack);
        }
    }
```

### 스케쥴러
스카우터 서버에서 플러그인 클래스 생성시 별도의 스케쥴러 쓰레드를 생성하도록 작성하였습니다.
해당 스케쥴러 쓰레드는 일정 주기마다 전역 HASHMAP에 담긴 XLOG를 다른 HASHMAP에 복사하고 구글 시트 작성클래스의 메소드를 호출합니다.

```
    class Scheduler implements Runnable {

        @Override
        public void run() {
            while(true) {
                try{
                    schedule = Long.parseLong(conf.getValue("ext_plugin_xlog_collector_schedule"));
                    
                    Thread.sleep(schedule);

                    Logger.println("[XLOG COLLECTOR PLUGIN] Schedule Start");
                    
                    synchronized (this) {
                        cloneHash = hash;
                        Logger.println("[XLOG COLLECTOR PLUGIN] INPUT LOG COUNT : " + logCount);
                        hash = new HashMap();
                        logCount = 0;
                    }
                    
                    if(cloneHash.size() > 0) {
                        maker.makeGoogleSheet(cloneHash);
                    }else{
                        Logger.println("[XLOG COLLECTOR PLUGIN] No Logs Collected ");
                    }

                }catch(Throwable e ) {
                    Logger.println("[XLOG Exception Message] : "+
                            e.getMessage()+ " : " + Arrays.toString(e.getStackTrace()));
                }
            }
        }
    }
```

### 로그 수집
XLOG 수집을 위해 annotation value는 PLUGIN_SERVER_XLOG을 사용하였습니다.
스카우터 서버에서 XLOG가 수집될때마다 플러그인 클래스 내에 XLOG 메소드도 호출됩니다.
XLogPack 클래스에는 수집된 XLOG에 대한 정보가 담겨있는데 필요한 정보를 뽑아 HASHMAP에 담을 데이터로 정제하였습니다.
```
    @ServerPlugin(PluginConstants.PLUGIN_SERVER_XLOG)
    public void xlog(XLogPack pack) {
        xpackEnabled = conf.getBoolean("ext_plugin_xlog_collector_enabled", true);
        elapsed = Integer.parseInt(conf.getValue("ext_plugin_xlog_collector_elapsed"));

        if(xpackEnabled) {

            try {
                
                if(pack.error != 0 || pack.elapsed > elapsed) {

                    if(AgentManager.getAgent(pack.objHash).objName != null) {
                       
                        Object key = getObjectName(AgentManager.getAgent(pack.objHash).objName);
              
                        String logType="NOMAL";
                        if(pack.error != 0) {
                            logType = "ERROR";
                        }

                        Object[] element = new Object[6];
                        element[0] = sdf.format(pack.endTime);
                        element[1] = logType;
                        element[2] = TextRD.getString(DateUtil.yyyymmdd(pack.endTime), TextTypes.SERVICE, pack.service);
                        element[3] = pack.elapsed;
                        element[4] = pack.error;
                        element[5] = pack.txid;

                        //Hash Input
                        pushHash(key, element);
                    }
                }

            } catch (Exception e) {
                Logger.println("[XLOG Exception Message] : " +
                        e.getMessage() + " : " + Arrays.toString(e.getStackTrace()));
            }
        }

    }
```

### HASHMAP 입력
XLOG가 수집되어 HASHMAP에 입력될 때 해당 KEY가 있을 경우와 없을 경우에 따라 분기처리를 하였습니다. 그러다보니 수시로 GET 과 PUT 작업이 수행되어 동기화 처리가 필요했습니다. Thread-Safe한 ConcurrentHashMap를 사용해도 되지만  ConcurrentHashMap를 사용하더라도 [ hash.get(key) == null ] 와 [ hash.put(key,list) ] 사이에서 동기화 문제가 발생할 수 있어 synchronized 메소드로 작성하였습니다. 

```
    protected synchronized void pushHash(Object key, Object[] element) {

        logCount++;
        if(hash.get(key) == null) {

            ArrayList<Object[]> list = new ArrayList<Object[]>();
            list.add(element);
            hash.put(key,list);

        }else{
            hash.get(key).add(element);
        }
    }
```

또한 위에 스케쥴러에서 HashMap을 복사할 때도 synchronized 블록을 사용하여 동기화 하였습니다.

```
synchronized (this) {
    cloneHash = hash;
    hash = new HashMap();
}
```

## 동기화 테스트

멀티 쓰레드 환경에서 HashMap의 동기화가 잘 수행되는지 검증이 필요하여 테스트 코드를 작성했습니다.
데이터를 계속 HashMap에 적재하고 또 다른 쓰레드는 일정 주기마다 해당 HashMap을 복사합니다.
아래 테스트는 30, 60 ,90ms 간격으로 500번의 HashMap 적재가 이뤄지는 동안 다른 쓰레드에서 매 0.5초마다 해당 HashMap을 복사하고, 호출한 수와 복사된 HashMap에 적재된 수가 일치한지 검증하는 테스트입니다.
```
    @Test
    public void isSyncHashWell() throws InterruptedException {

        final int SAMPLE_SIZE = 500;
        int cnt = 0;

        Random random = new Random();
        SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
        String[] objNameArray = new String[]{"PAS","MAS","AUTH","M","EVENT"};
        String[] typeArray = new String[]{"NOMAL","ERROR"};
        String[] serviceArray = new String[]{"/product/category/{GET}", "/event/random/{GET}","/product/maker/{GET}","/product/brand/{GET}"};

        TestConfigure conf = new TestConfigure(0,500);

        for(int i=3; i < 10; i+=3) {

            TestSheetMaker maker = new TestSheetMaker();
            XlogCollector xlog = new XlogCollector(conf, maker);

            while(cnt < SAMPLE_SIZE) {
                xlog.pushHash(objNameArray[random.nextInt(5)],new Object[]{sdf.format(new Date().getTime()),typeArray[random.nextInt(2)],serviceArray[random.nextInt(4)],(int)Math.random(),(int)Math.random()});
                //Thread.sleep(random.nextInt(1));
                Thread.sleep(i);
                cnt++;
            }

            Thread.sleep(conf.getScheduleTime());
            Assert.assertEquals(cnt, xlog.count());

            cnt = 0;
        }
    }
```

### 구글 시트
다음으로 복사된 HashMap 데이터로 구글 시트를 작성합니다. 구글 시트를 생성하고 작성하기 위한 API를 사용하려면 별도의 인증 절차가 필요한데 Service_Account 방식을 이용하였습니다.\
Service Account에 대해서는 아래 링크를 참고해주세요.\
[서비스 어카운트 방식](https://cloud.google.com/iam/docs/understanding-service-accounts?hl=ko)

pom.xml에 관련 디펜던시를 추가해줍니다.
```
<!-- https://mvnrepository.com/artifact/com.google.api-client/google-api-client -->
        <dependency>
            <groupId>com.google.api-client</groupId>
            <artifactId>google-api-client</artifactId>
            <version>1.30.9</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.google.oauth-client/google-oauth-client-jetty -->
        <dependency>
            <groupId>com.google.oauth-client</groupId>
            <artifactId>google-oauth-client-jetty</artifactId>
            <version>1.30.6</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.google.apis/google-api-services-sheets -->
        <dependency>
            <groupId>com.google.apis</groupId>
            <artifactId>google-api-services-sheets</artifactId>
            <version>v4-rev612-1.25.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.10</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpcore -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpcore</artifactId>
            <version>4.4.12</version>
        </dependency>

        <dependency>
            <groupId>com.google.auth</groupId>
            <artifactId>google-auth-library-oauth2-http</artifactId>
            <version>0.20.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.google.apis/google-api-services-drive -->
        <dependency>
            <groupId>com.google.apis</groupId>
            <artifactId>google-api-services-drive</artifactId>
            <version>v3-rev197-1.25.0</version>
        </dependency>
```

### 구글 시트 생성
복제한 HASHMAP 이용하여 구글 시트를 작성합니다.
여기서는 Google Sheet, Driver에 대한 서비스 객체를 생성하고 HashMap의 데이터로 구글 시트를 작성합니다.
```
    public void makeGoogleSheet(HashMap<Object,ArrayList<Object[]>> logMap) {

        try{
          
            //시트 서비스
            Sheets sheetService = getSheetsService(AuthMode.SERVICE_ACCOUNT);
            //드라이브 서비스
            Drive driveService = getDriveService(AuthMode.SERVICE_ACCOUNT);
            
            //구글 시트 생성 
            spreadsheetId = CreateSheet(sheetService, driveService, sheetName);

            ..... 중략 .....

            //서비스별 시트를 생성
            for(Map.Entry entry : logMap.entrySet()) {       
              
                List<List<Object>> values = new ArrayList<>();
                ArrayList<Object[]> list = (ArrayList<Object[]>) entry.getValue();
                for(Object[] val : list) {
                    values.add(Arrays.asList(val));
                }

                //서비스 별 탭 생성
                AddSheet(sheetService,spreadsheetId,values,entry.getKey().toString(), serviceMap.get(entry.getKey().toString()));

            }

            Logger.println("[XLOG COLLECTOR PLUGIN] Log Report Write End :" + (endTime - startTime) / 1000000 +" ms");

        }catch(Throwable e) {

            Logger.println("[XLOG Exception Message] : "+
                    e.getMessage()+ " : " + Arrays.toString(e.getStackTrace()));
        }
    }

```

### 서비스 객체 생성
Google Sheet API 사용을 위한 서비스 객체를 생성합니다.
API를 실제 사용하려면 인증 권한을 가져와야 합니다. 

```
public Sheets getSheetsService(AuthMode authMode) throws IOException{
    ServiceAccountCredentials credential = null;

    if (authMode == AuthMode.SERVICE_ACCOUNT) {
        credential = getServiceAccountAuthorize();
    }

    HttpRequestInitializer requestInitializer = new HttpCredentialsAdapter(credential);

    return new Sheets.Builder(HTTP_TRANSPORT, JSON_FACTORY, requestInitializer)
            .setApplicationName(APPLICATION_NAME)
            .build();
}
```

### 인증 권한 가져오기
GCP에서 서비스 계정 키를 생성하면 인증정보가 담긴 JSON파일을 만들 수 있습니다. 아래 클래스는 JSON 파일에서 엑세스 토큰을 얻어 서비스 이용 범위를 지정한 뒤 반환합니다. 
```
    public ServiceAccountCredentials getServiceAccountAuthorize() throws IOException{

        ServiceAccountCredentials sourceCredentials = ServiceAccountCredentials
                .fromStream(GoogleSheetMaker.class.getResourceAsStream("/service.json"));

        sourceCredentials = (ServiceAccountCredentials) sourceCredentials
                .createScoped(
                        "https://www.googleapis.com/auth/spreadsheets",
                        "https://www.googleapis.com/auth/drive",
                        "https://www.googleapis.com/auth/drive.file",
                        "https://www.googleapis.com/auth/drive.appdata",
                        "https://www.googleapis.com/auth/drive.apps.readonly"
                );

        return sourceCredentials;
    }
```

### 구글 시트 생성
위에서 생성한 서비스 객체로 구글 시트를 생성할 수 있습니다. 다음은 간단한 구글 시트 생성 예제입니다.
```
Spreadsheet request = new Spreadsheet().setProperties(new SpreadsheetProperties().setTitle("SHEET TITLE"));
        
Spreadsheet response = sheetService.spreadsheets().create(request).execute();
```

아래 코드에서 구글 시트를 생성하고 특정 구글 디렉토리에 옮긴 후 해당 디렉토리를 공유된 모든 사용자들에게 읽기 권한으로 지정하였습니다.

```
    public String CreateSheet(Sheets sheetService, Drive driveService, String sheetName) throws IOException {
        //구글 시트 생성
        try{

            Spreadsheet request = new Spreadsheet().setProperties(new SpreadsheetProperties().setTitle(sheetName));
        
            Spreadsheet response = sheetService.spreadsheets().create(request).execute();

            //구글 시트 ID
            String spreadsheetId = response.getSpreadsheetId();
    
            //구글 폴더 ID
            String folderId = "9t7UBivUF3QYpAjRA9IIKGD-A";

            //생성된 구글 Sheet 폴더 이동
            File file = driveService.files().update(spreadsheetId, null)
                    .setFileId(spreadsheetId)
                    .setAddParents(folderId)
                    .setFields("id, parents")
                    .execute();

            //모든 사용자 읽기 권한
            BatchRequest batch = driveService.batch();
            Permission userPermission = new Permission()
                    .setType("anyone")
                    .setRole("reader");
            driveService.permissions().create(spreadsheetId, userPermission).setFields("id").queue(batch, new JsonBatchCallback<Permission>() {
                @Override
                public void onFailure(GoogleJsonError e,
                                      HttpHeaders responseHeaders) throws IOException {
                    Logger.println("[XLOG Exception Message] : "+
                            e.getMessage());
                }

                @Override
                public void onSuccess(Permission permission, HttpHeaders httpHeaders) throws IOException {
                    //Logger.println("Permission ID: " + permission.getId());
                    System.out.println("Permission ID: " + permission.getId());

                }
            });
            batch.execute();
            Logger.println(response.getSpreadsheetUrl());
            //System.out.println(response.getSpreadsheetUrl());

            return spreadsheetId;

        }catch (Exception e) {
            e.printStackTrace();
            Logger.println("[XLOG Exception Message] : "+
                    e.getMessage()+ " : " + Arrays.toString(e.getStackTrace()));
        }
        return "";
    }
```

### 시트 쓰기
시트에 실제 데이터를 입력하기 위해서는 범위를 지정해야합니다.\
기본적으로 A1:A5 처럼 :를 구분자로 범위를 설정하면 되고 특정탭에 작성하고 싶으면 !를 구분자로하여 탭명!A1:A5 형태로 지정하면 됩니다.
입력하려는 데이터는 List 내부에 배열형태로 만들어야 합니다.
```
public void writeSheet(Sheets sheetService, String spreadsheetId, List<List<Object>> values, String name) throws IOException{
        String range = name+"!A1:F1000000";

        values.addAll(0,Arrays.asList(
                Arrays.asList(
                        "Time",
                        "Type",
                        "Service URL",
                        "Elapsed Time(ms)",
                        "Error Code",
                        "Link"
                )
        ));

        ValueRange body = new ValueRange()
                .setValues(values);
        UpdateValuesResponse result =
                sheetService.spreadsheets().values().update(spreadsheetId, range, body)
                        .setValueInputOption("RAW") // RAW -> 사용자가 입력한 값을 그대로 저장 (구문 분석 X)
                        .execute();

        Logger.println("[XLOG COLLECTOR PLUGIN]" + result.getUpdatedRows() + " rows update.");
    }
```


## 참고 자료
1. https://developers.google.com/sheets/api/guides
2. https://github.com/scouter-project/scouter/blob/master/scouter.document/main/Plugin-Guide_kr.md
3. https://m.blog.naver.com/PostView.nhn?blogId=occidere&logNo=221071278283&proxyReferer=https:%2F%2Fwww.google.com%2F
