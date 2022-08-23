---
layout: post
title:  "Flutter에서 GraphQL 사용하기"
description: "Flutter에서 GraphQL 사용하는 방법에 대해 알아보겠습니다."
date:   2022.08.22.
writer: "선요한"
categories: Flutter
---

## GraphQL이란	

[GraphQL](https://graphql-kr.github.io/)은 페이스북이 2015년에 발표된 API를 위한 쿼리언어입니다. 기존의 Rest API의 단점을 보완하고자 나온 기술로 클라이언트에서의 데이터 요청과 서버에서의 기능 확장이 더 유연해질 수 있도록 설계되었습니다. 주요 특징으로는 Rest API와 달리 하나의 endpoint를 가지고 있고 POST 메서드만을 사용하지만 내부적으로 query, mutation, subscription으로 불리는 세가지 요청 방식을 가지고 있습니다. [더보기](https://graphql-kr.github.io/learn/)



## Flutter환경 설정

그렇다면 Flutter에서 GraphQL을 사용하기 위한 설정에 대해 알아보겠습니다. Flutter에서 GraphQL을 사용하기 위해서는 [GraphQL Flutter](https://pub.dev/packages/graphql_flutter)라이브러리를 사용해야 합니다. 

```yaml
dependencies:
  flutter:
    sdk: flutter
  graphql_flutter: ^5.1.0
```

다른 라이브러리들과 동일하게 pubspec.yaml 파일에 추가해줍니다. 

```dart
...

import 'package:graphql_flutter/graphql_flutter.dart'; // 라이브러리 import

void main() async {

  final HttpLink httpLink = HttpLink(
    'https://api.github.com/graphql', // endpoint 등록
  );

  var authLink = AuthLink(
    getToken: () async => 'Bearer <YOUR_PERSONAL_ACCESS_TOKEN>',
    // OR
    // getToken: () => 'Bearer <YOUR_PERSONAL_ACCESS_TOKEN>',
  ); // 인증 토큰이 있다면 등록

  final Link link = authLink.concat(httpLink);
    
  var graphQLClient = GraphQLClient(
      link: link,
      cache: GraphQLCache(
      	store: InMemoryStore(), partialDataPolicy: PartialDataCachePolicy.accept),
    );

  ValueNotifier<GraphQLClient> client = ValueNotifier(
    graphQLClient
  ); // endpoint + 토큰정보가 담긴 링크와 캐시 정책을 정의한 클라이언트 정의

  ...
}

...
```

라이브러리 import 후 GraphQL의 endpoint와 인증토큰 정보, 메모리 정책등을 정의한 클라이언트를 정의해줍니다.

```dart
return GraphQLProvider(
    client: client,
    child: MaterialApp(
      title: 'Flutter Demo',
      ...
    ),
  );
```

마지막으로 앱의 뿌리가 되는 MaterialApp을 전에 정의해두었던 client를 인자로 가지고 있는 GraphQLProvider로 감싸주게 되면 Flutter에서 GraphQL을 사용하기 위한 모든 준비가 끝나게 됩니다. 



### Query, Mutation, Subscription

GraphQL에서는 POST, GET, PUT등의 HTTP 메서드들 중에 POST 메서드만 사용하는 대신에 자체적으로 query, mutation, subscription이라는 요청방식을 가지고 있습니다. 

- Query는 서버 데이터 조회(CRUD에서 R)를 위한 단순 fetch용 쿼리를 날릴 때 사용하는 방식으로 HTTP의 GET과 같은 개념이라고 할 수 있습니다.
- Mutation은 서버의 데이터 변경(CRUD에서 CUD)을 요청할때 사용하는 방식으로 HTTP의 POST와 같은 개념이라고 할 수 있습니다.
- Subscription은 Query와 비슷하지만 web socket과 stream을 사용하여 실시간으로 변하는 데이터를 조회하는 방식입니다. 



## GraphQL 사용

Flutter에서 GraphQL을 사용하는 방식에는 Widget방식과 Method방식이 있습니다. 공통점은 동일한 쿼리문을 사용하지만 차이점은 Widget에 직접적으로 구현을 하는지 여부라고 할 수 있습니다. 

우선 사용할 GraphQL 쿼리를 String 형식으로 정의해줍니다.

```dart
static String getPostByID = r""" 
  query($id: ID!) {
    post(id: $id) {
      title
      content
    }
  }
""";
// query 예시

static String createPost = r"""
  mutation($post: createPostInput) {
    createPost(input: $post) {
      post {
        title
        description
      }
    }
  }
""";
// mutation 예시

static String fetchUsers = """
   subscription fetchOnlineUsers {
   online_users {
     user {
       name
     }
   }
 }
""";
// subscription 예시

```



#### Widget 방식

Widget 방식은 `Query`, `Mutation`, `Subscription` 위젯을 사용해 구현하는 방식입니다.

- Query

```dart
Widget build(BuildContext context) {
  return Query(
      options: QueryOptions(document: gql(getPostByID), variables: {"id": "${검색할 ID}"}),
      builder: (result, {refetch, fetchMore}) {
        if (result.isLoading) {
          return CircularProgressIndicator();
        }
        final post = result.data!['post'];
        return Scaffold(
          appBar: AppBar(title: Text(post['title'])),
          body: Container(
            child: Text(post['content']),
          ),
        );
      });
}
```

Query 위젯은 option과 builder라는 두개의 필수 인자를 갖고 있습니다. builder의 첫번째인자인 result의 .isLoading을 통해 로딩중인지 여부를 확인하여 로딩UI를 표시할 수 있습니다. 

option 인자에 들어오는 QueryOption은 document와 variables이라는 두개의 필수인자를 가지고 있는데 document에는 gql(쿼리문), variables에는 쿼리에 필요한 변수들을 json 형식으로 넣어주면 됩니다.



- Mutation

```dart
Widget build(BuildContext context) {
  return Mutation(
      options: MutationOptions(
        document: gql(createPost),
        update: (cache, result) => result,
        onCompleted: (data) {
          //mutation이 완료되었을 때 실행되는 부분
        },
        onError: (e) {
          //error시 실행되는 부분
        }),
      builder: (runMutation, result) => IconButton(
            onPressed: () async {
              runMutation({
                "post": {
                  "data": {
                    "title": PostInputController.to.title,,
                    "description": PostInputController.to.description,
                    }
                  } 
                });
            },
            icon: result!.isLoading
                ? CircularProgressIndicator()
                : Icon(Icons.upload_rounded),
          ));
}
```

Mutation 위젯도 option과 builder라는 두개의 필수 인자를 갖고 있습니다. builder의 result는 Query의 result와 같은 인자이고 runMutation은 mutation을 실행하는 함수로 안에 인자를 넣게 됩니다. 



- Subscription

```dart
 final WebSocketLink websocketLink = WebSocketLink(
      url: 'wss://api.github.com/graphql',
      config: SocketClientConfig(
        autoReconnect: true,
        inactivityTimeout: Duration(seconds: 30),
      ),
    );
```

Subscription을 사용하기 위해서는 HTTP 링크가 아닌 Web socket 링크를 사용해야 합니다.

```dart
Widget build(BuildContext context) {
  return Subscription(
    options: SubscriptionOptions(
      document: gql(fetchUsers),
    ),
    builder: (result) {
      if (result.hasException) {
        return Text(result.exception.toString());
      }
      if (result.isLoading) {
        return Center(
          child: const CircularProgressIndicator(),
        );
      }
      return ResultAccumulator.appendUniqueEntries(
        latest: result.data,
        builder: (context, {results}) => DisplayUsers(
          reviews: results.reversed.toList(),
        ),
      );
    },
  );
}
```

Subscription 위젯도 option과 builder라는 두개의 필수 인자를 갖고 있습니다. builder의 result는 Query의 result와 같은 인자로 쿼리로 받은 결과물을 받을 수 있습니다. 



#### Method 방식

Method 방식은 `query()`, `muate()`, `subscribe()` 함수를 사용해 구현하는 방식입니다.

- Query

```dart
var queryResult = graphQLClient.query(QueryOptions(
      document: gql(getPostByID),
      variables: {
        "input": {
          "id":  UserInfoController.to.userID,
        }
      },
  ));
  print(queryResult);
```

GraphQL 클라이언트의 query() 함수를 통해 결과를 반환합니다. 



- Mutation

```dart
graphQLClient.mutate(MutationOptions(
    document: gql(createPost),
    variables: {
      "post": {
        "data": {
          "title": PostInputController.to.title,
          "description": PostInputController.to.description, 
        }
      }
    },
    onCompleted: (dynamic result) {
      if(res!=null) {
        print(result);
      }
    },
    onError: (e) {
      print("error = $e");
    },
  ));
```

GraphQL 클라이언트의 mutate() 함수를 통해 결과를 반환합니다. query()와 다르게 onCompleted와 onError 인자를 제공합니다. 



- Subscription

```dart
Stream<dynamic> _logStream = graphQLClient.subscribe(SubscriptionOptions(
    document: gql(fetchUsers),
  ));
```

GraphQL 클라이언트의 subscribe() 함수를 통해 결과를 반환합니다. query와 mutation과 다르게 필수 인자가 document밖에 없습니다.




## 정리

Flutter에서 GraphQL을 사용하는 방법에 대해 알아보았습니다. 위젯 방식과 메서드 방식 모두 장단점이 있지만 공식 문서에는 위젯방식을 추천하고 있습니다. 쏙 프로젝트의 경우 쿼리 관련된 모든 로직을 분리하고 위치에 구애받지 않는 통신을 하기 위해 메서드 방식을 적용했습니다. 




## 참고 자료
- <https://graphql-kr.github.io/>
- <https://graphql-kr.github.io/learn/>
- <https://pub.dev/packages/graphql_flutter>
- <https://sulfurbottom.netlify.app/Development/flutter%EC%97%90-graphql-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0/>
- <https://blog.promedius.ai/flutter-graphqleul-jal-sayonghaeboja/>