---
layout: post
title:  "엘라스틱서치 플러그인 이슈"
description: "다나와에선 엘라스틱서치 버전을 기존 7.6.2를 사용하고 있었습니다. 그리고 당시 최신이였던 7.9.1 버전으로 업그레이드를 진행하였습니다. 그리고 잘사용하던 중 알수없는 현상이 발생하기 시작하였습니다. 테스크가 쌓이기 시작하였고, 특정 노드에는 API 호출이 안되는 현상이 발생하였습니다.  어떤 작업을해도 테스크가 해소되지 않았고, 엘라스틱서치 클러스터를 결국 재시작해야하는 상황까지 발생하였습니다. 원인파악을 위해 알아보던 중 엘라스틱서치 플러그인에 제공되는 NodeClient에 문제가 확인되어 공유하기 위해 포스팅하였습니다." 
date:   2020.12.30.
writer: "김준우"  
categories: Elastic
---
## 소개
다나와에선 엘라스틱서치 버전을 기존 7.6.2를 사용하고 있었습니다. 그리고 당시 최신이였던 7.9.1 버전으로 업그레이드를 진행하였습니다. 그리고 문제없이 사용하던 중 알수없는 현상이 발생하기 시작하였습니다. 테스크가 쌓이기 시작하였고, 특정 노드에는 API 호출이 안되는 현상이 발생하였습니다.  어떤 작업을해도 테스크가 해소되지 않았고, 엘라스틱서치 클러스터를 결국 재시작해야하는 상황까지 발생하였습니다. 원인파악을 위해 알아보던 중 엘라스틱서치 플러그인에서 문제가 발생을 확인되어 정리합니다.


### 현상
- 플러그인 API 호출이 간헐적으로 테스크가 쌓이면서 해소가 안되는 현상 발생
- 플러그인 API 호출한 노드만 문제되며, 해당 노드 API 호출이 안되는 현상 발생
- 다른노드의 API 문제가 없으나 요청을 재현하면 1, 2번과 같이 동일한 현상 발생


테스크 확인하는 API
```jsx
GET /_cat/tasks?v&detailed
```

테스크가 쌓이게 되면 아래 이미지처럼 running_time이 증가하며 테스크가 해소되지 않아 갯수가 증가하는 현상이 발생합니다.

![/images/2020-12-30-elasticsearch-plugins-issue/Untitled.png](/images/2020-12-30-elasticsearch-plugins-issue/Untitled.png)

## 테스트 방식
원인파악시 플러그인 로직에서 최초 의심되는 부분이 NodeClient를 통해 검색요청이였습니다. 그래서 가정을 간헐적 통신에 장애가 있어 검색요청에 hang이 발생한다고 생각하였습니다. 그래서 검색요청에 타임아웃 60초로 설정합니다. tasks가 쌓여도 timeout으로 해소가되는지 확인하기 위함입니다. 그리고 NodeClient에서 제공하는 검색방식이 ActionListener 사용 유무(이하 동기/비동기 방식)의 차이가 있습니다. 엘라스틱서치 내부 로직을 확인해보았을땐 결과값의 리턴차이만 가지고, 결국엔 동일한 비동기 방식을 사용한다는걸 알 수 있었습니다. 그렇지만 차이가있는지 확인을 위해 각각 액션을 다르게 구성하여 테스트 해보도록 하겠습니다. 그리고 NodeClient의 동기 비동기 방식의 차이가 아닌 다른 문제가 있는지 확인을 하기 위해 비동기 방식로직에 결과를 멤버변수의 큐에 적재하고, 메인로직에선 큐의 적재된 결과를  전달받아 응답하도록하여 구성하였습니다.  개발된 테스트 플러그인을 ES에 적용하여 아래 엔드포인트로 요청보내며 현상이 발생하지는 확인해 보겠습니다.

- /_async : ActionListener 람다를 통해 결과를 응답합니다.
- /_sync: get 메소드를 호출하여 대기하였다가 결과를 응답합니다.
- /_async_more: ActionListener를 통해 결과를 큐에 넣고, 큐의 결과를 받아 응답합니다.

아래는 플러그인 소스코드입니다.

```jsx
public class TestAction extends BaseRestHandler {

	private static Logger logger = Loggers.getLogger(TestAction.class, "");
	private static long asyncCount = 0;
	private static long asyncMoreCount = 0;
	private static long syncCount = 0;
	private static long sleepCount = 0;

	private static final String CONTENT_TYPE_JSON = "application/json;charset=UTF-8";

	@Inject
	TestAction(Settings settings, RestController controller) {
//		controller.registerHandler(POST, "/_async", this);
//		controller.registerHandler(POST, "/_sync", this);
//		controller.registerHandler(POST, "/_async_more", this);
//
//		controller.registerHandler(GET, "/_async", this);
//		controller.registerHandler(GET, "/_sync", this);
//		controller.registerHandler(GET, "/_async_more", this);
	}

	@Override
	public String getName() {
		return "rest_handler_test";
	}

	@Override
	public List<Route> routes() {
		List<Route> list = new ArrayList<>();
		list.add(new Route(POST, "/_async"));
		list.add(new Route(POST, "/_sync"));
		list.add(new Route(POST, "/_async_more"));

		list.add(new Route(GET, "/_async"));
		list.add(new Route(GET, "/_sync"));
		list.add(new Route(GET, "/_async_more"));

		return list;
	}

	private LinkedBlockingQueue<Object> future = new LinkedBlockingQueue<>(1);

	/**
	 * 액션 핸들링 처리
	 */
	@Override
	protected RestChannelConsumer prepareRequest(RestRequest request, NodeClient client) throws IOException {
		return channel -> {
			final String action = request.path().replace("/_","");
			logger.info("action: {}", action);
			final StringWriter buffer = new StringWriter();
			JSONWriter builder = new JSONWriter(buffer);

			SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
			searchSourceBuilder.query(QueryBuilders.matchAllQuery());
			searchSourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
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
			} else if ("async_more".equalsIgnoreCase(action)) {

				client.search(searchRequest, new ActionListener<SearchResponse>() {
					@Override
					public void onResponse(SearchResponse searchResponse) {
						logger.info("비동기 큐 요청. 요청 횟수: {}", ++asyncMoreCount);
						try {
							future.put(searchResponse);
						} catch (InterruptedException e) {
							logger.error("", e);
						}
					}

					@Override
					public void onFailure(Exception e) {
						logger.trace("비동기 큐 요청. 요청 횟수: {}", ++asyncMoreCount);
						try {
							future.put(e);
						} catch (InterruptedException interruptedException) {
							logger.error("", e);
						}
					}
				});

				Object f = future.take();
				long size = 0;
				if (f instanceof SearchResponse) {
					size = ((SearchResponse) f).getHits().getTotalHits().value;
				}

				builder.object()
						.key("action").value(action)
						.key("count").value(asyncMoreCount)
						.key("totalHits").value(size)
						.endObject();
				channel.sendResponse(new BytesRestResponse(RestStatus.OK, CONTENT_TYPE_JSON, buffer.toString()));

			}
		};
	}

}
```

## 테스트 결과

아래 표를 보면 7.9.0 부터 현재 최신버전까지 동기방식과 비동기 큐 방식을 사용하였을때 테스크가 쌓이고, 노드의 통신이 안되는 현상이 발생하였습니다. NodeClient의 검색 timeout을 설정하였지만 적용되지 않아 테스크의 러닝타임이 60s를 초과해도 해소되지 않은 모습을 확인하였습니다. NodeClient의 검색시 단순히 동기 방식과 비동기 방식 차이만 보면 동기방식을 사용할때 결과를 리턴하지 못해 hang이 발생하여 문제가 발생한다고 판단할 수 있습니다. 하지만 비동기 큐 방식의 결과를 보았을땐 비동기지만 동일하게 테스크가 쌓이며 통신에 장애가 발생하였습니다. NodeClient를 사용하여 검색요청 후 추가 로직을 수행하게 되면 hang이 발생하며 해당 노드의 테스크가 쌓이며 통신에 장애가 발생한다는걸 확인 할 수 있었습니다. 

|버전|동기|비동기|비동기 큐|
|---|---|---|
|7.6.2|발견 안됨|발견 안됨|발견 안됨|
|7.7.0|발견 안됨|발견 안됨|발견 안됨|
|7.8.0|발견 안됨|발견 안됨|발견 안됨|
|7.8.1|발견 안됨|발견 안됨|발견 안됨|
|7.9.0|발견됨|발견 안됨|발견됨|
|7.9.1|발견됨|발견 안됨|발견됨|
|7.10.1|발견됨|발견 안됨|발견됨|

## 정리

이번 엘라스틱서치 hang 발생을 확인해보았습니다. 현재 7.9.0 버전 부터 엘라스틱서치 플러그인의 NodeClient 검색 요청 후 추가 코드를 수행하면 테스크가 쌓이고, 통신이 안되는 문제가 발생하는 현상을 확인 할 수 있었습니다. 테스트에선 동기와 비동기를 구분하였지만 실제 엘라스틱서치 내부 로직에선 리턴 차이밖에 없기 때문에 동기와 비동기에서 문제가 발생되어 보이진 않습니다.  현재 엘라스틱서치 깃협이력을 보아 최근 플러그인연관 로직이 자주 변경 되어 버그로 생각됩니다.