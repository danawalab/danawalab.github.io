---

layout: post
title:  "Flutter - GetX를 이용한 상태관리"
description: "GetX를 이용한 Flutter 상태관리에 대해 알아보겠습니다."
date:   2022.08.05.
writer: "선요한"
categories: Flutter
---

<iframe src="https://mblogthumb-phinf.pstatic.net/20130628_109/boonsuck_1372409471077z3rza_GIF/%AC%D4%AC%DA%AC%E6%AC%DC%AC%DA-%AC%DC%AC%E0%AC%E4%AC%EF-%AC%A5%AC%D1%AC%DB-%AC%E1%AC%F1%AC%E4%AC%EE-539490.gif?type=w2" height="300" width="500" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

## Flutter란

Flutter의 상태관리에 대한 설명에 앞서 Flutter가 무엇인지 알아보겠습니다. Flutter는 2017년 구글이 발표한 하나의 코드로 안드로이드, iOS, 리눅스, 윈도우, 맥, 웹 브라우저에서 모두 동작되는 앱을 만들 수 있게 해주는 크로스 플랫폼 프레임워크입니다. Flutter에서 사용되는 언어 역시 구글에서 만든 dart라는 언어를 사용하고 있습니다. 



## Flutter의 구조

그렇다면 Flutter에서 상태관리가 필요한 배경이 무엇인지 알아보겠습니다. Flutter는 모든 것이 'Widget'으로 이루어져 있습니다. Java에서 모든 객체가 Object라는 클래스를 상속받듯이 Flutter에서 UI와 관련된 모든 것은 Widget입니다. 화면 역시 두가지의 Widget으로 분류할 수 있습니다. 

![1](/images/2022-08-05-Flutter-Getx/1.JPG)

```html
<img src="/images/2022-08-05-Flutter-Getx/1.JPG" alt="drawing" width="800"/>
```

1. StatelessWidget
2. StatefulWidget

StatelessWidget은 변하지 않는 정적인 화면을 구성할때 사용합니다. 단순 텍스트나 앱에 대한 정보 화면 등이 이 경우에 해당합니다. StatefulWidget보다 성능이 좋습니다. 

StatefulWidget은 화면에 변화가 있는 동적인 화면을 구성할때 사용합니다.  체크박스나 라디오박스, 사용자가 텍스트를 입력하는 필드, 게임의 점수 등 실시간으로 업데이트되는 UI가 포함된 화면의 경우 StatefulWidget을 사용하게 됩니다. 

코드로 보면 다음과 같습니다.

##### StatelessWidget

```dart
class StatelessExample extends StatelessWidget {
  const StatelessExample({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return  Container(
      color: Colors.blue,
      child: const Text('StatelessWidget Example'),
    );
  }
}
```

##### StatefulWidget

 ```dart
 import 'package:flutter/material.dart';
 
 class StatefulExample extends StatefulWidget {
   const StatefulExample({Key? key}) : super(key: key);
 
   @override
   State<StatefulExample> createState() => _StatefulExampleState();
 }
 
 class _StatefulExampleState extends State<StatefulExample> { // setState() 의 영향을 받는 영역
     int _counter = 0;
 
   @override
   Widget build(BuildContext context) { // UI가 렌더링 되는 영역
     return Scaffold(
       appBar: AppBar(
         title: const Text('StatefulWidget Example'),
       ),
       body: Center(
         child: ElevatedButton(
           child: Text(
             '현재 숫자: $_counter',
           ),
           onPressed: () {
             setState(() { // 누르는 순간 재렌더링
               _counter++;
             });
           },
         ),
       ),
     );
   }
 }
 ```

StatelessWidget은 클래스 내에 생성자와 build() 함수 하나만 있습니다. 하지만 StatefulWidget은 StatelessWidget과 달리 State 형식의 또다른 서브클래스가 존재하는데 이 서브클래스가 동적인 화면을 렌더링하게 됩니다. 위의 예시는 버튼을 누를때마다 카운터가 증가되는 동적인 화면의 예시입니다. setState() 함수를 통해 화면 변경된 데이터로 화면을 재렌더링(build() 함수)하게 됩니다. 



##### Stateful Widget 코드예시 결과

![1](/images/2022-08-05-Flutter-Getx/3.gif)

<img src="/images/2022-08-05-Flutter-Getx/3.gif" alt="drawing" width="500"/>



## 상태 관리란

상태관리는 UI에서 실시간으로 변하는 여러 데이터들의 상태를 효율적으로 관리하기 위한 개념입니다. 예시를 들어 설명해보겠습니다. 

![2](/images/2022-08-05-Flutter-Getx/2.jpg)



위의 화면을 보면 여러 데이터를 확인할 수 있습니다. 

- 글쓴이 프로필 이미지
- 글쓴이 닉네임
- 글의 작성 시간
- 글의 카테고리
- 글의 좋아요
- 댓글 갯수
- 댓글 쓴 사람들의 목록
- 나의 글/댓글 좋아요 눌렀는지 여부 등

한 화면에도 여러 데이터들이 있는 것을 확인할 수 있습니다. 이 화면에서 상태관리가 필요한 이유는 크게 두가지가 있습니다.

1. 특정 데이터가 바뀔 때 마다 화면 전체를 재렌더링 하기에는 애플리케이션에서의 자원의 낭비가 너무 크다.
2. 특정 데이터가 바뀔 때 다른 화면에서도 해당 데이터의 변화가 동일하게 이루어져야 하는 경우가 있다. 

1번의 예시는 다음과 같습니다. 사용자가 댓글을 입력하고 업로드 하는 순간 댓글 목록에 새로운 댓글이 보여져야 합니다. 화면의 관점에서 새로운 데이터가 생기고 그에 따라 새로운 UI를 그려주어야 하기 때문에 Flutter는 StatefulWidget으로 해당 화면을 재렌더링 하게 됩니다. 그 외에도 글이나 댓글의 하트를 누를 경우에도, 댓글을 삭제하고 수정할때도 Flutter는 화면의 일부를 변경하기 위해 화면 전체를 재렌더링 하게 됩니다. 하지만 화면의 일부분의 변경을 적용하기 위해 화면 전체를 재렌더링 하는 방식은 너무 비효율적입니다.  

2번의 예시는 다음과 같습니다. 사용자가 해당 글에 하트를 눌러서 하트 UI가 노란색 하트로 변경이 되었습니다. 만약에 해당 글에 하트를 눌렀는지의 여부 데이터를 다른 페이지에서도 참고하고 있다면 해당 페이지에서도 하트가 노란색 하트로 변경이 되어야 합니다. 

Flutter가 디폴트로 제공하는 StatefulWidget을 통해서도 기능은 구현이 되지만 애플리케이션이 복잡해질수록 setState() 로 전체 화면을 재렌더링 하는 방식은 비효율성이 애플리케이션에 규모에 비례하여 더 커지게 됩니다. 

상태관리 기술을 사용하게 되면 1번 처럼 실시간으로 변화하는 데이터에 대한 처리와 2번처럼 여러 컴포넌트에서 공통적으로 사용하는 데이터의 동기화를 아주 쉽고 효율적으로 해결할 수 있습니다.



## GetX 상태 관리 라이브러리

Flutter에서 상태관리의 필요성에 대해 알아보았으니 이제 상태관리를 적용하는 방법을 알아보겠습니다. React에서 대표적인 상태관리 라이브러리가 Redux가 있다면 Flutter에서는 [GetX](https://pub.dev/packages/get/install)를 주로 사용합니다. 

![1](/images/2022-08-05-Flutter-Getx/4.jpg)

<img src="/images/2022-08-05-Flutter-Getx/4.JPG" alt="drawing" width="800"/>

#### 라이브러리 사용 설정

- 라이브러리 import

  ```bash
   $ flutter pub add get
  ```

  - 프로젝트 경로에서 위의 커맨드로 라이브러리를 import 합니다.
  - 명령어 대신에 pubspec.yaml 파일에 직접 입력하여 추가하는 방법도 있습니다. 이 경우 버전 명시도 가능합니다. 

- 라이브러리 동기화

  ```bash
   $ flutter pub get
  ```

  - 프로젝트 경로에서 위의 커맨드로 라이브러리를 현재 프로젝트와 동기화 합니다.
  - 명령어 대신에 Android Studio 에디터에서 pubspec.yaml 파일을 열고 오른쪽 상단에 'Packeges get' 버튼을 클릭하는 방법도 있습니다. 

- 라이브러리 사용

  ```dart
  import 'package:get/get.dart';
  ```

  - 라이브러리를 사용하는 dart 파일에서 import합니다.

  ```dart
  void main() {
    runApp(const GetMaterialApp(home: MyApp()));
  }
  ```

  - 프로젝트 main() 함수안에 첫페이지를 시작하는 부분을 GetMaterialApp() 으로 감싸줍니다. 



#### 사용법

GetX를 통한 상태관리 방식은 크게 두가지가 있습니다.

1. 단순 상태 관리
2. 반응형 상태 관리

단순 상태 관리와 반응형 상태 관리의 차이는 반응형 상태 관리의 경우 데이터가 변화가 있을 때만 재랜더링을 하게 되는 반면에 단순 상태 관리는 기존의 데이터와 변경되는 데이터가 같은지 확인하지 않습니다. 더 나아가 반응형 상태관리는 workers라는 추가 기능도 있습니다. 두가지 방식의 상태 관리를 코드로 확인해보겠습니다.



#### 1. 단순 상태 관리

```dart
import 'package:get/get.dart';

class SimpleController extends GetxController {
  int counter = 0;

  void increase() {
    counter++;
    update();
  }
}
```

- 단순 상태 관리를 위한 controller를 만들어줍니다. 이 controller는 counter라는 변수와 counter의 값을 1씩 증가해주는 increase() 함수를 가지고 있습니다. increase() .함수 안에 update()는 이 controller을 바라보고있는 모든 코드에 업데이트를 알리는 역할을 합니다. 

```dart
class MyHomePage extends StatelessWidget {
  MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    Get.put(SimpleController()); // controller 등록
    return Scaffold(
      appBar: AppBar(
        title: const Text("단순 상태관리"),
      ),
      body: Center(
        child: GetBuilder<SimpleController>( // 실시간 렌더링
          builder: (controller) {
            return ElevatedButton(
              child: Text(
                '현재 숫자: ${controller.counter}',
              ),
              onPressed: () {
                controller.increase();
                // Get.find<SimpleController>().increase();
              },
            );
          },
        ),
      ),
    );
  }
}
```

- 위에서 만들어준 controller를 사용하는 화면 클래스입니다. 먼저 controller를 사용하기 위해 Get.put()으로 controller를 등록해줍니다. GetBuilder()아래의 모든 위젯은 controller에서 변경되는 데이터를 실시간으로 반영할 수 있는 상태가 됩니다. controller.counter는 controller의 변수를 실시간으로 반영하게 되고 controller.increase()는 controller의 counter 데이터를 실시간으로 증가시키게 됩니다. 만약 GetBuilder를 사용하지 않을 경우 Get.find<[Controller종류]>().[변수 혹은 함수] 로 컨트롤러의 데이터를 실시간 변경 혹은 반영할 수 있습니다. 



#### 결과

![1](/images/2022-08-05-Flutter-Getx/5.gif)

<img src="/images/2022-08-05-Flutter-Getx/5.gif" alt="drawing" width="500"/>





#### 2. 반응형 상태 관리

```dart
import 'package:get/get.dart';

class ReactiveController extends GetxController {
  RxInt counter = 0.obs;

  void increase() {
    counter++;
  }
}
```

- 반응형 상태 관리를 위한 Controller를 만들어줍니다. 이 Controller는 counter라는 변수와 counter의 값을 1씩 증가해주는 increase() 함수를 가지고 있습니다. 
- 단순 상태관리와 비교하면 변수를 선언하는 방식과 업데이트 함수 부분이 다릅니다. 변수를 선언하는 방식은 변수의 타입을 RxInt, RxString등 Rx{타입}의 방식으로 선언하고 변수의 값은 '.obs'를 붙이게 됩니다. 업데이트의 경우 update() 함수를 부르지 않아도 됩니다. 

```dart
class MyHomePage extends StatelessWidget {
  MyHomePage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    Get.put(SimpleController()); // 단순 상태 관리 controller 등록
    Get.put(ReactiveController()); // 반응형 상태 관리 controller 등록
    return Scaffold(
      appBar: AppBar(
        title: const Text("단순 / 반응형 상태관리"),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            GetBuilder<SimpleController>( // 단순 상태 관리
              builder: (controller) {
                return ElevatedButton(
                  child: Text(
                    '[단순]현재 숫자: ${controller.counter}',
                  ),
                  onPressed: () {
                    controller.increase();
                    // Get.find<SimpleController>().increase();
                  },
                );
              },
            ),
            GetX<ReactiveController>( // 반응형 상태관리 - 1
              builder: (controller) {
                return ElevatedButton(
                  child: Text(
                    '반응형 1 / 현재 숫자: ${controller.counter.value}', // .value 로 접근
                  ),
                  onPressed: () {
                    controller.increase();
                    // Get.find<ReactiveController>().increase();
                  },
                );
              },
            ),
            Obx( // 반응형 상태관리 - 2
                  () {
                    return ElevatedButton(
                      child: Text(
                        '반응형 2 / 현재 숫자: ${Get.find<ReactiveController>().counter.value}', // .value 로 접근
                      ),
                      onPressed: () {
                        Get.find<ReactiveController>().increase();
                      },
                    );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

- 단순 상태 관리를 테스트했던 화면에 그대로 반응형 상태관리 테스트를 위한 위젯을 추가한 코드입니다. 먼저 단순 상태 관리와 동일하게 controller를 사용하기 위해 Get.put()으로 controller를 등록해줍니다. 반응형 상태 관리에서 데이터를 실시간으로 반영하는 방식에는 두가지가 있습니다. 
  1. GetX() - GetX() 아래의 모든 위젯은 controller에서 변경되는 데이터를 실시간으로 반영할 수 있는 상태가 됩니다. controller.counter.value (단순 상태 관리와 다르게 .value 를 추가해 주어야 합니다) 는 controller의 변수를 실시간으로 반영하게 되고 controller.increase()는 controller의 counter 데이터를 실시간으로 증가시키게 됩니다. 만약 GetX를 사용하지 않을 경우 Get.find<[Controller종류]>().[변수 혹은 함수] 로 컨트롤러의 데이터를 실시간 변경 혹은 반영할 수 있습니다. 
  2. Obx() - Obx() 아래의 모든 위젯은 GetX()와 마찬가지로 controller에서 변경되는 데이터를 실시간으로 반영할 수 있는 상태가 됩니다. 사용 방식은 거의 동일하지만 차이가 있다면 GetX()와 달리 controller의 이름을 지정할 수가 없어서 Get.find() 방식으로 접근해야 합니다. 



#### 결과

![1](/images/2022-08-05-Flutter-Getx/6.gif)

<img src="/images/2022-08-05-Flutter-Getx/6.gif" alt="drawing" width="500"/>



#### 반응형 상태 관리의 추가 기능 - worker

반응형 상태관리에서는 worker라는 추가 기능이 있습니다. Worker는 controller 안에서 onInit() 함수를 override하고 그 안에 추가해서 사용하게 되는데 아래의 4가지 종류가 있습니다. 

- Ever : 매번 변경 될 때 실행
- Once : 처음 변경 되었을 때만 실행
- Interval : 계속 변경이 있는 동안 특정 지정 시간 인터벌이 지나면 실행
- Debounce : 인터벌이 끝나고 나서 특정 지정 시간 이후에 한번만 실행



```dart
import 'package:get/get.dart';

class ReactiveController extends GetxController {
  static ReactiveController get to => Get.find();
  RxInt counter = 0.obs;

  @override
  void onInit() {
    once(counter, (_) {
      print('once : $_이 처음으로 변경되었습니다.');
    });
    ever(counter, (_) {
      print('ever : $_이 변경되었습니다.');
    });
    debounce(
      counter,
          (_) {
        print('debounce : $_가 마지막으로 변경된 이후, 1초간 변경이 없습니다.');
      },
      time: Duration(seconds: 1),
    );
    interval(
      counter,
          (_) {
        print('interval $_가 변경되는 중입니다.(1초마다 호출)');
      },
      time: Duration(seconds: 1),
    );
    super.onInit();
  }


  void increase() {
    counter++;
  }
}
```

- 반응형 controller 내부에 worker들을 추가한 모습입니다. 

#### 결과

![1](/images/2022-08-05-Flutter-Getx/7.gif)

<img src="/images/2022-08-05-Flutter-Getx/7.gif" alt="drawing" width="500"/>



#### Get.find() 를 보다 간단하게 사용하는 방법

##### 1. Getter 사용

- Get.find<[Controller종류]>().[변수 혹은 함수]를 보다 간단하게 사용하기 위해서는 아래와 같이 controller 내부에 getter를 생성해주면 됩니다.

```dart
class SimpleController extends GetxController {
  static SimpleController get to => Get.find();
  ...
 }
```

- Get.find()를 기존보다 더 짧은 코드로 사용할 수 있게 됩니다.

```dart
// 전
Get.find<SimpleController>().increase();
// 후
SimpleController.to.increase();
```



##### 2. GetView 사용

- Get.find()를 사용하는 클래스에 StatelessWidget 대신 GetView를 상속받는 방식입니다.

```dart
// 전
class SimpleState extends StatelessWidget{}
// 후
class SimpleState extends GetView<SimpleController>{}
```

- Get.find()를 기존보다 더 짧은 코드로 사용할 수 있게 됩니다.

```dart
// 전
Get.find<SimpleController>().increase();
// 후
controller.increase();
```



## GetX 적용 코드

SSOK앱에 GetX를 적용한 코드의 일부분입니다.

![1](/images/2022-08-05-Flutter-Getx/8.jpg)

<img src="/images/2022-08-05-Flutter-Getx/8.JPG" alt="drawing" width="800"/>

- 달력 페이지의 데이터를 관리하는 controller입니다.

![1](/images/2022-08-05-Flutter-Getx/9.jpg)

<img src="/images/2022-08-05-Flutter-Getx/8.JPG" alt="drawing" width="800"/>

![1](/images/2022-08-05-Flutter-Getx/10.jpg)

<img src="/images/2022-08-05-Flutter-Getx/8.JPG" alt="drawing" width="800"/>

- 달력 페이지에서 controller를 구독하고 있는 모습입니다. 



#### 결과

![1](/images/2022-08-05-Flutter-Getx/11.gif)

<img src="/images/2022-08-05-Flutter-Getx/11.gif" alt="drawing" width="500"/>




## 정리

Flutter에서 GetX 라이브러리를 통한 상태관리에 대해서 알아보았습니다. GetX말고도 Provider나 Bloc 패턴이 존재하지만 현재 시점에서 GetX가 간단한 문법과 여러가지 기능들을 가지고 있어서 가장 인기가 많은 것 같습니다. 




## 참고 자료
- https://ko.wikipedia.org/wiki/%ED%94%8C%EB%9F%AC%ED%84%B0
- https://joooosan.tistory.com/entry/Flutter-3%ED%83%84-Flutter%EC%9D%98-UI%EC%97%90-%EA%B4%80%ED%95%98%EC%97%AC?category=391758
- https://fl0wering.tistory.com/84