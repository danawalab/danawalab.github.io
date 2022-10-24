---
layout: post
title:  "iOS/Android 프로젝트에 Flutter Module 적용하는 방법" 
description: "iOS/Android 프로젝트에 Flutter Module을 적용하고 Method Channel을 통해 argument를 전달하는 방법을 알아보겠습니다. "
date:   2022.10.24.
writer: "선요한"
categories: Common
---

## Flutter Module이란

Flutter Module은 기존에 존재하던 안드로이드나 iOS앱의 일부로 Flutter로 만든 앱을 통합할 수 있도록 해주는 기술입니다. 기존에 존재하던 네이티브 앱 위에 어떤 페이지를 빠르게 추가/변경하고자 할 때 안드로이드와 iOS별로 따로 개발하는 방식이 아닌 Flutter로 모듈을 만들어 안드로이드와 iOS 앱에 동시에 적용하는 방식으로 개발의 효율성을 높일 수 있습니다. 이 때 Flutter 모듈은 dart 코드로 실행되기 때문에 일반적인 Flutter 프로젝트와  동일한 성능을 가지게 됩니다. 

 

## Flutter Module 생성

네이티브 앱에 플러터 모듈을 적용하기 위해 먼저 플러터 모듈을 생성하는 방법에 대해 알아보겠습니다. 



#### 1. Flutter 프로젝트 생성시 디폴트인 Application이 아닌 Module을 선택

![image](https://user-images.githubusercontent.com/54565079/197431063-e8e93422-25a7-441e-89f9-397f47549293.png)



커맨드로는 모듈을 생성하는 방법은 다음과 같습니다

```bash
flutter create -t module --org com.example my_flutter
```



#### 2. 모듈에서 실행될 코드 작성

저의 경우 예시로 홈화면이 없는 세개의 페이지를 만들었고 각 페이지에서 네이티브 앱으로부터 [Method Channel](https://docs.flutter.dev/development/platform-integration/platform-channels)을 통해 인자를 받고 화면에 출력하는 코드를 작성하였습니다. Method Channel은 플러터에서 네이티브 플랫폼과 소통하는 채널입니다.

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

const channelName = 'com.example.module-test/custom';
const methodChannel = MethodChannel(channelName);

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Module Test',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      routes: {
        '/custom1': (context) => const Custom1(title: 'Flutter Custom Module1'),
        '/custom2': (context) => const Custom2(title: 'Flutter Custom Module2'),
        '/custom3': (context) => const Custom3(title: 'Flutter Custom Module3'),
      },
    );
  }
}
class Custom1 extends StatefulWidget {
  const Custom1({super.key, required this.title});
  final String title;

  @override
  State<Custom1> createState() => _Custom1State();
}

class _Custom1State extends State<Custom1> {
  int _counter = 0;
  var _methodCallArguments = "null";

  @override
  void initState() {
    super.initState();
    methodChannel.setMethodCallHandler(methodHandler); 
    // 화면이 생성되는 순간 메소드 채널에서 호출되는 메소드의 핸들러를 설정
  }

  Future<dynamic> methodHandler(MethodCall methodCall) async {
      // 메소드 호출이 입력으로 들어오는 함수
    print('methodHandler: ${methodCall.method}');
    if (methodCall.method == "getUserToken"){ 
        // 메소드 채널에서 호출된 메소드가 "getUserToken"이라는 메소드인 경우
      print('methodHandler: ${methodCall.arguments}');
      setState(() {
        _methodCallArguments = methodCall.arguments;
      }); // "getUserToken"이라는 메소드를 통해 들어온 인자를 private 변수에 저장
      return "received flutter";
    }
  } // 메소드 채널로 전달된 인자를 private 변수로 저장합니다 

  void _incrementCounter() {
    setState(() {
      _counter++;
      print("_methodCallArguments: $_methodCallArguments");
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.cyanAccent,
        leading: IconButton(
          icon: const Icon(Icons.arrow_back_ios),
          onPressed: () {
            SystemNavigator.pop();
          },
        ),
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'Module Test1',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headline4,
            ),
            Padding(
              padding: const EdgeInsets.only(top: 20.0),
              child: Column(
                children: [
                  const Text(
                      "Method Call Arguments"
                  ),
                  Padding(
                    padding: const EdgeInsets.only(top: 8.0),
                    child: Text(
                      _methodCallArguments,
                      style: Theme.of(context).textTheme.headline5,
                    ),
                  ), // 메소드 채널로 받은 인자를 화면에 출력합니다
                ],
              ),
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Colors.cyanAccent,
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}


class Custom2 extends StatefulWidget {...}
class _Custom2State extends State<Custom2> {...}
// Custom1과 같은 코드

class Custom3 extends StatefulWidget {...}
class _Custom3State extends State<Custom3> {...}
// Custom1과 같은 코드
```



#### 3. 안드로이드앱에서 참조하는 AAR(Android Archive)을 생성

플러터 모듈 경로에서 다음 명령어를 입력합니다.

```bash
flutter build aar
```

정상적으로 빌드가 되면 build 폴더 아래에 aar이 생성되고 다음과 같은 안내가 나오게 됩니다. 

![image](https://user-images.githubusercontent.com/54565079/197436055-b994271a-f1c6-4ae9-b02c-7d21c69fb255.png)

![image](https://user-images.githubusercontent.com/54565079/197432554-ed1630d5-b033-4f21-b90d-c9884b4df739.png)

플러터 모듈 생성이 완료되었습니다. 위의 안내대로 안드로이드 앱에서 모듈을 적용해보겠습니다. 



## Android 앱 설정

#### 1. app/build.gradle 파일에서 자바 버전을 확인

안드로이드 앱에서 플러터 모듈을 적용하기 전에 플러터 안드로이드 엔진이 Java8 버전을 사용하기 때문에 안드로이드 프로젝트가 Java 8버전을 사용하는지 확인해야 합니다. 

```gradle
android {
  //...
  compileOptions {
    sourceCompatibility 1.8 // or JavaVersion.VERSION_1_8
    targetCompatibility 1.8 // or JavaVersion.VERSION_1_8
  }
}
```



#### 2. 모듈의 경로 설정

AAR을 빌드하고 나서 안내의 2~4번 대로 안드로이드 프로젝트에서 모듈을 인식할 수 있도록 경로를 설정합니다.

```gradle
android {
  // ...
  buildTypes {
        ...
        profile {
            initWith debug
        } // 추가 1
    }
}

repositories {
  maven {
  	String storageUrl = System.env.FLUTTER_STORAGE_BASE_URL ?: "https://storage.googleapis.com"
    url '../flutter_module/build/host/outputs/repo' // aar 경로
  }
  maven {
    url "$storageUrl/download.flutter.io"
  }
} // 추가 2

dependencies {
  // ...
  
  debugImplementation 'com.example.flutter_module:flutter_debug:1.0'
  profileImplementation 'com.example.flutter_module:flutter_profile:1.0'
  releaseImplementation 'com.example.flutter_module:flutter_release:1.0'
  // 추가 3
  
}
```



- `Build was configured to prefer settings repositories over project repositories but repository ...` 와 같은 에러가 날 경우 안드로이드 스튜디오 버전이 다른 경우이기 때문에 경로 설정을 app/build.gradle이 아닌 settings.gradle에 해주어야 합니다. 

```gradle
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        String storageUrl = System.env.FLUTTER_STORAGE_BASE_URL ?: "https://storage.googleapis.com"
        maven {
            url '../flutter_module/build/host/outputs/repo'
        }
        maven {
            url "$storageUrl/download.flutter.io"
        }
    }
}
rootProject.name = "AndroidApp"
include ':app'
```

- `FAILURE: Build completed with 3 failures.` 와 같은 에러가 날 경우 dependencies에 추가한 코드를 다음과 같이 변경해야 합니다.

```gradle
dependencies {
		
		...
		// debugImplementation 'com.example.flutter_module:flutter:1.0:debug'
		// profileImplementation 'com.example.flutter_module:flutter:1.0:profile'
		// releaseImplementation 'com.example.flutter_module:flutter:1.0:release'
		// 안내 방식
		
        debugImplementation 'com.example.flutter_module:flutter_debug:1.0'
   		profileImplementation 'com.example.flutter_module:flutter_profile:1.0'
    	releaseImplementation 'com.example.flutter_module:flutter_debug:1.0'
    	// 변경 후 
    }
```



#### 3. FlutterActivity 추가

AndroidManifest.xml 파일의   `<application> </application>` 태그 내에 FlutterActivity를 추가합니다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.androidapp">
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.AndroidApp">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <activity
            android:name="io.flutter.embedding.android.FlutterActivity"
            android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale|layoutDirection|fontScale|screenLayout|density|uiMode"
            android:hardwareAccelerated="true"
            android:windowSoftInputMode="adjustResize"
            /> <!-- FlutterActivity 추가 --> 
        
    </application>
</manifest>
```

플러터 모듈을 실행하는 페이지에서 FlutterActivity를 import하고 모듈을 실행하는데에 사용합니다. 

```kotlin
import io.flutter.embedding.android.FlutterActivity

...

startActivity(
    FlutterActivity.createDefaultIntent(this)
  )
```



#### 4. 모듈 실행하는 코드 작성

안드로이드앱에서 플러터 모듈을 실행하는 코드를 작성합니다. 저의 경우 플러터 모듈내에 생성했던 3개의 페이지로 각각 메소드 채널을 통해 인자를 넘겨주는 방식의 코드를 작성하였습니다. 이때 FlutterActivity는 FlutterEngine을 사용하게 되는데 FlutterEngine은 생성되는데에 어느정도 시간이 걸리기 때문에 미리 캐시로 등록하는 방법을 사용했습니다. 각각의 FlutterEngine은 모듈 내의 페이지의 경로를 가지게 되고 메소드 채널은 각각의 FlutterEngine 안에서 모듈과 소통하게 됩니다. 

```kotlin
package com.example.androidapp

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.util.Log
import android.widget.Button
import android.widget.EditText
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.embedding.engine.FlutterEngineCache
import io.flutter.embedding.engine.dart.DartExecutor
import io.flutter.plugin.common.MethodChannel

private const val FLUTTER_ENGINE_NAME1 = "custom1" 
private const val FLUTTER_ENGINE_NAME2 = "custom2"
private const val FLUTTER_ENGINE_NAME3 = "custom3"
// FlutterEngineCache 에 등록하게 될 FlutterEngine 고유 id

class MainActivity : AppCompatActivity() {

    private val channelName = "com.example.module-test/custom" // 메소드 채널 이름(모듈과 같아야 함)
    lateinit var channel1 : MethodChannel
    lateinit var channel2 : MethodChannel
    lateinit var channel3 : MethodChannel
    // 메소드채널 변수 생성

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        warmupFlutterEngine()
        setContentView(R.layout.activity_main)

        val btn1 = findViewById<Button>(R.id.btn_1)
        val btn2 = findViewById<Button>(R.id.btn_2)
        val btn3 = findViewById<Button>(R.id.btn_3)
        // 각 모듈 페이지를 실행하게 될 버튼

        val edit1 = findViewById<EditText>(R.id.edit1)
        val edit2 = findViewById<EditText>(R.id.edit2)
        val edit3 = findViewById<EditText>(R.id.edit3)
        // 각 모듈 페이지로 전달할 인자를 입력하는 텍스트필드

        btn1.setOnClickListener {
            val parameter = edit1.text.toString()
            Log.d("parameter", parameter)
            channel1.invokeMethod("getUserToken", parameter)

            startActivity(
                FlutterActivity
                    .withCachedEngine(FLUTTER_ENGINE_NAME1)
                    .build(this)
            )
            overridePendingTransition(0, 0)
        } // 1번 버튼이 눌렸을 때 텍스트 필드의 값을 메소드채널을 통해 전달, FLUTTER_ENGINE_NAME1에 등록된 모듈의 1번페이지 실행
        btn2.setOnClickListener {
            val parameter = edit2.text.toString()
            Log.d("parameter", parameter)
            channel2.invokeMethod("getUserToken", parameter)

            startActivity(
                FlutterActivity
                    .withCachedEngine(FLUTTER_ENGINE_NAME2)
                    .build(this)
            )
            overridePendingTransition(0, 0)
        } // 2번 버튼이 눌렸을 때 텍스트 필드의 값을 메소드채널을 통해 전달, FLUTTER_ENGINE_NAME2에 등록된 모듈의 2번페이지 실행

        btn3.setOnClickListener {
            val parameter = edit3.text.toString()
            Log.d("parameter", parameter)
            channel3.invokeMethod("getUserToken", parameter)

            startActivity(
                FlutterActivity
                    .withCachedEngine(FLUTTER_ENGINE_NAME3)
                    .build(this)
            )
            overridePendingTransition(0, 0)

        } // 3번 버튼이 눌렸을 때 텍스트 필드의 값을 메소드채널을 통해 전달, FLUTTER_ENGINE_NAME2에 등록된 모듈의 3번페이지 실행
    }

    private fun warmupFlutterEngine() {
        val flutterEngine1 = FlutterEngine(this)
        val flutterEngine2 = FlutterEngine(this)
        val flutterEngine3 = FlutterEngine(this)
        // FlutterEngine 변수들 초기화

        flutterEngine1.navigationChannel.setInitialRoute("/custom1");
        flutterEngine2.navigationChannel.setInitialRoute("/custom2");
        flutterEngine3.navigationChannel.setInitialRoute("/custom3");
        // FlutterEngine 변수들 경로 설정

        flutterEngine1.dartExecutor.executeDartEntrypoint(
            DartExecutor.DartEntrypoint.createDefault()
        )
        flutterEngine2.dartExecutor.executeDartEntrypoint(
            DartExecutor.DartEntrypoint.createDefault()
        )
        flutterEngine3.dartExecutor.executeDartEntrypoint(
            DartExecutor.DartEntrypoint.createDefault()
        )
        // FlutterEngine을 등록하기 위해 Dart 코드 실행

        channel1 = MethodChannel(flutterEngine1.dartExecutor.binaryMessenger, channelName)
        channel2 = MethodChannel(flutterEngine2.dartExecutor.binaryMessenger, channelName)
        channel3 = MethodChannel(flutterEngine3.dartExecutor.binaryMessenger, channelName)
        // 메소드채널 위치 설정


        FlutterEngineCache
            .getInstance()
            .put(FLUTTER_ENGINE_NAME1, flutterEngine1)
        FlutterEngineCache
            .getInstance()
            .put(FLUTTER_ENGINE_NAME2, flutterEngine2)
        FlutterEngineCache
            .getInstance()
            .put(FLUTTER_ENGINE_NAME3, flutterEngine3)
        // FlutterEngine 캐시에 등록
    }
}
```



#### 5. 결과

<iframe src="https://www.youtube.com/embed/9jtTzK8u9xc" title="Flutter Add-To-App (Android)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen style="display:block; width:100%; height: 100%"></iframe>



## iOS 프로젝트 설정

#### 1. 모듈의 경로 설정

프로젝트의 Podfile(없다면 생성)에 다음 코드를 입력합니다. 

~~~
flutter_application_path = '../../flutter_module'
load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')

target 'ios_module_test' do
  install_all_flutter_pods(flutter_application_path)
end

post_install do |installer|
  flutter_post_install(installer) if defined?(flutter_post_install)
end

~~~

파일 저장 후 다음 커맨드를 입력합니다.

~~~bash
pod install
~~~



#### 2. 모듈 실행하는 코드 작성

AppDelegate.swift 파일에 FlutterEngine을 생성하고 GeneratedPluginRegistrant에 등록합니다.

~~~swift
import UIKit
import Flutter
import FlutterPluginRegistrant

@UIApplicationMain
class AppDelegate: FlutterAppDelegate {
    let engineGroup = FlutterEngineGroup(name: "my flutter engine group", project: nil)
    var custom1Engine: FlutterEngine?
    var custom2Engine: FlutterEngine?
    var custom3Engine: FlutterEngine?
  	// FlutterEngine 변수 생성


  override func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
   
      custom1Engine = engineGroup.makeEngine(withEntrypoint: nil, libraryURI: nil, initialRoute: "/custom1")
      custom2Engine = engineGroup.makeEngine(withEntrypoint: nil, libraryURI: nil, initialRoute: "/custom2")
      custom3Engine = engineGroup.makeEngine(withEntrypoint: nil, libraryURI: nil, initialRoute: "/custom3")
      // 각 FlutterEngine에 경로 설정
              
      GeneratedPluginRegistrant.register(with: custom1Engine!)
      GeneratedPluginRegistrant.register(with: custom2Engine!)
      GeneratedPluginRegistrant.register(with: custom3Engine!)
      // 생성한 FlutterEngine들을 GeneratedPluginRegistrant에 등록
          
      return super.application(application, didFinishLaunchingWithOptions: launchOptions);
  }
}
~~~



ViewController.swift 파일에 모듈을 실행하는 코드를 추가합니다.

~~~swift
import UIKit
import Flutter

let channelName = "com.example.module-test/custom"

class ViewController: UIViewController {
    
    @IBOutlet var textButton1: UIButton!
    @IBOutlet var textButton2: UIButton!
    @IBOutlet var textButton3: UIButton!
  	// 각 모듈 페이지를 실행하게 될 버튼
  
    @IBOutlet var textField1: UITextField!
    @IBOutlet var textField2: UITextField!
    @IBOutlet var textField3: UITextField!
  	// 각 모듈 페이지로 전달할 인자를 입력하는 텍스트필드

  override func viewDidLoad() {
    super.viewDidLoad()
  }
    
    @IBAction func buttonTapped1(_ sender: UIButton) {
      
        let mText = textField1.text
        
        if let flutterEngine = (UIApplication.shared.delegate as! AppDelegate).custom1Engine{
            let flutterViewController = FlutterViewController(engine: flutterEngine, nibName: nil, bundle: nil)
            // 기존에 등록한 FlutterEngine으로 FlutterViewController를 생성
            let newsChannel = FlutterMethodChannel(name:channelName, binaryMessenger: flutterViewController.binaryMessenger)
            // 메소드채널 설정
              
          newsChannel.invokeMethod("getUserToken", arguments: mText, result: {
                       (result) -> Void in
              print("swift-to-flutter result: \(String(describing: result))")
                   })

              flutterViewController.modalPresentationStyle = .overCurrentContext
              flutterViewController.isViewOpaque = false
              present(flutterViewController, animated: false, completion: nil)
              // FlutterViewController 실행
          }
    } // 1번 버튼이 눌렸을 때 텍스트 필드의 값을 메소드 채널을 통해 전달, custom1Engine에 등록된 모듈의 1번페이지 실행
    
    
    @IBAction func buttonTapped2(_ sender: UIButton) {
        let mText = textField2.text
        
        if let flutterEngine = (UIApplication.shared.delegate as! AppDelegate).custom2Engine{
            let flutterViewController = FlutterViewController(engine: flutterEngine, nibName: nil, bundle: nil)
            // 기존에 등록한 FlutterEngine으로 FlutterViewController를 생성
            let newsChannel = FlutterMethodChannel(name:channelName, binaryMessenger: flutterViewController.binaryMessenger)
            // 메소드채널 설정
            
            newsChannel.invokeMethod("getUserToken", arguments: mText, result: {
                         (result) -> Void in
                print("swift-to-flutter result: \(String(describing: result))")
                     })
            
            flutterViewController.modalPresentationStyle = .overCurrentContext
            flutterViewController.isViewOpaque = false
            present(flutterViewController, animated: false, completion: nil)
            // FlutterViewController 실행
        }
    } // 2번 버튼이 눌렸을 때 텍스트 필드의 값을 메소드 채널을 통해 전달, custom2Engine에 등록된 모듈의 2번페이지 실행
    
    
    @IBAction func buttonTapped3(_ sender: UIButton) {
        let mText = textField3.text
        
        if let flutterEngine = (UIApplication.shared.delegate as! AppDelegate).custom3Engine{
            let flutterViewController = FlutterViewController(engine: flutterEngine, nibName: nil, bundle: nil)
            // 기존에 등록한 FlutterEngine으로 FlutterViewController를 생성
            let newsChannel = FlutterMethodChannel(name:channelName, binaryMessenger: flutterViewController.binaryMessenger)
            // 메소드채널 설정
            
            newsChannel.invokeMethod("getUserToken", arguments: mText, result: {
                         (result) -> Void in
                print("swift-to-flutter result: \(String(describing: result))")
                     })
            
            flutterViewController.modalPresentationStyle = .overCurrentContext
            flutterViewController.isViewOpaque = false
            present(flutterViewController, animated: true, completion: nil)
            // FlutterViewController 실행
        }
    } // 3번 버튼이 눌렸을 때 텍스트 필드의 값을 메소드 채널을 통해 전달, custom3Engine에 등록된 모듈의 3번페이지 실행
    
    
}

~~~



#### 3. 결과

 <iframe src="https://www.youtube.com/embed/abJ3flg2nUc" title="Flutter Add-To-App (iOS)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen style="display:block; width:100vw; height: 100vh"></iframe>



## 정리

Android/iOS 앱에 Flutter 모듈을 화면 단위로 추가하는 방법을 알아보았습니다. 이미 존재하는 앱을 부분적으로 수정하거나 새로운 기능을 빠르게 구현해야 할 경우 매우 유용한 기능이라고 생각합니다. 




## 참고 자료
- <https://docs.flutter.dev/development/add-to-app>
- <https://g-y-e-o-m.tistory.com/187?category=411699>
- <https://minchanyoun.tistory.com/84?category=1013568#Arctic%--Fox%--%EB%B-%--%EC%A-%--%--%EC%-D%B-%ED%-B%--%EC%--%--%--%EC%--%-D%EC%--%B-%EB%--%-C%--%ED%--%--%EB%A-%-C%EC%A-%-D%ED%-A%B-%EB%-A%--%--build-gradle%EC%-D%B-%--%EC%--%--%EB%-B%-C%--settings-gradle%EC%--%--%--%EC%--%--%EB%-E%--%EC%--%--%EA%B-%--%EC%-D%B-%--allprojects%--%EB%A-%--%ED%-F%AC%EC%A-%--%ED%--%A-%EB%A-%AC%EB%A-%BC%--%EC%B-%--%EA%B-%-->
