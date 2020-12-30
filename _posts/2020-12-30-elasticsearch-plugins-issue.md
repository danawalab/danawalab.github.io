---
layout: post
title:  "엘라스틱서치 플러그인 이슈"
description: "다나와에선 엘라스틱서치 버전을 기존 7.6.2를 사용하고 있었습니다. 그리고 당시 최신이였던 7.9.1 버전으로 업그레이드를 진행하였습니다. 그리고 잘사용하던 중 알수없는 현상이 발생하기 시작하였습니다. 테스크가 쌓이기 시작하였고, 특정 노드에는 API 호출이 안되는 현상이 발생하였습니다.  어떤 작업을해도 테스크가 해소되지 않았고, 엘라스틱서치 클러스터를 결국 재시작해야하는 상황까지 발생하였습니다. 원인파악을 위해 알아보던 중 엘라스틱서치 플러그인에 제공되는 NodeClient에 문제가 확인되어 공유하기 위해 포스팅하였습니다." 
date:   2020.12.30. 13:00:00
writer: "김준우"  
categories: Elastic
---
## 소개
다나와에선 엘라스틱서치 버전을 기존 7.6.2를 사용하고 있었습니다. 그리고 당시 최신이였던 7.9.1 버전으로 업그레이드를 진행하였습니다. 그리고 잘사용하던 중 알수없는 현상이 발생하기 시작하였습니다. 테스크가 쌓이기 시작하였고, 특정 노드에는 API 호출이 안되는 현상이 발생하였습니다.  어떤 작업을해도 테스크가 해소되지 않았고, 엘라스틱서치 클러스터를 결국 재시작해야하는 상황까지 발생하였습니다. 원인파악을 위해 알아보던 중 엘라스틱서치 플러그인에 제공되는 NodeClient에 문제가 확인되어 공유하기 위해 포스팅하였습니다.

### 현상
- 플러그인 API 호출이 간헐적 테스크가 쌓이면서 해소가 안되는 현상 발생
- 플러그인 API 요청하여 문제되면 대부분 해당 노드의 API 호출이 안되는 현상 발생
- 다른노드의 API 문제가 없으나 1번과 같이 현상이 발생되면 동일한 현상 발생
   

테스크 확인하는 API
```jsx
GET /_cat/tasks?v&detailed
```

테스크가 쌓이게 되면 아래 이미지처럼 running_time과 테스크 갯수가 계속 증가합니다.
![/images/2020-12-30-elasticsearch-plugins-issue/Untitled.png](/images/2020-12-30-elasticsearch-plugins-issue/Untitled.png)

## 테스트 방식

도커 컴포즈를 사용하여 엘라스틱서치 클러스터를 구성하였습니다. 테스트를 위한 플러그인을 적용합니다. 구현한 /_async, /_sync API로 여러번 요청을 보내 task가 쌓이는 현상이 재현되는지 확인합니다. 그리고 엘라스틱서치 버전을 변경하며 확인합니다.

### 테스트코드

플러그인 내부 로직은 NodeClient 를 사용하여 검색API를 호출합니다. 액션으로 분기하여 비동기, 동기 기능을 수행하도록 구현해두었습니다. 비동기 방식은 ActionListener를 사용하여 결과를 받으면 onResponse  호출하도록 구성하였고, 동기 방식은 get 메소드를 호출하여 결과를 받을때 까지 대기하도록 구성하였습니다. 

아래는 플러그인 소스코드입니다.

```jsx
public class TestAction extends BaseRestHandler {

	private static Logger logger = Loggers.getLogger(TestAction.class, "");
	private static long asyncCount = 0;
	private static long syncCount = 0;

	private static final String CONTENT_TYPE_JSON = "application/json;charset=UTF-8";

	@Inject
	TestAction(Settings settings, RestController controller) {}

	@Override
	public String getName() {
		return "rest_handler_test";
	}

	@Override
	public List<Route> routes() {
		List<Route> list = new ArrayList<>();
		list.add(new Route(POST, "/_async"));
		list.add(new Route(POST, "/_sync"));

		list.add(new Route(GET, "/_async"));
		list.add(new Route(GET, "/_sync"));
		return list;
	}

	/**
	 * 액션 핸들링 처리
	 */
	@Override
	protected RestChannelConsumer prepareRequest(RestRequest request, NodeClient client) throws IOException {
		return channel -> {
			final String action = request.path().replace("/","").replace("_", "");
			logger.info("action: {}", action);
			final StringWriter buffer = new StringWriter();
			JSONWriter builder = new JSONWriter(buffer);

			SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
			searchSourceBuilder.query(QueryBuilders.matchAllQuery());
			SearchRequest searchRequest = new SearchRequest();
			searchRequest.source(searchSourceBuilder);

			if ("async".equalsIgnoreCase(action)) {
				// 비동기 호춣
				client.search(searchRequest, new ActionListener<SearchResponse>() {
					@Override
					public void onResponse(SearchResponse searchResponse) {
						logger.info("비동기 요청. 요청 횟수: {}", ++asyncCount);
						builder.object()
								.key("action").value(action)
								.key("count").value(asyncCount)
								.endObject();
						channel.sendResponse(new BytesRestResponse(RestStatus.OK, CONTENT_TYPE_JSON, buffer.toString()));
					}

					@Override
					public void onFailure(Exception e) {
						logger.error("비동기 방식의 요청.  에러:", e);
						builder.object()
								.key("action").value(action)
								.key("count").value(asyncCount)
								.endObject();
						channel.sendResponse(new BytesRestResponse(RestStatus.INTERNAL_SERVER_ERROR, CONTENT_TYPE_JSON, buffer.toString()));
					}
				});

			} else if ("sync".equalsIgnoreCase(action)) {
				// 동기 호춣
				client.search(searchRequest).get();
				logger.info("동기 요청. 요청 횟수: {}", ++syncCount);
				builder.object()
						.key("action").value(action)
						.key("count").value(syncCount)
						.endObject();
				channel.sendResponse(new BytesRestResponse(RestStatus.OK, CONTENT_TYPE_JSON, buffer.toString()));
			}
		};
	}

}
```

## 테스트 결과

테스크가 쌓이는 현상이 발견되었는지 확인해보았습니다. 아래 표를 보면 7.9.0 부터 현재 최신버전까지 동기방식을 사용하였을때 테스크가 쌓이고, 노드의 통신이 안되는 현상이 발생하였습니다. 

|버전|동기|비동기|
|---|---|---|
|7.6.2|발견 안됨|발견 안됨|
|7.7.0|발견 안됨|발견 안됨|
|7.8.0|발견 안됨|발견 안됨|
|7.8.1|발견 안됨|발견 안됨|
|7.9.0|발견됨|발견 안됨|
|7.9.1|발견됨|발견 안됨|
|7.10.1|발견됨|발견 안됨|

## 정리

현재 엘라스틱서치 깃헙 이력을 보았을때 NodeClient 관련 로직이  최근에 지속적으로 변경이되고 있기 때문에 당분간 커스텀 플러그인을 개발할때는 내장 액션과 유사하게 비동기 방식을 사용하여 개발해야 이런 이슈에 회피할 수 있습니다.