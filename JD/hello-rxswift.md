# hello rxswift
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

## rx swift란 무엇인가
- 간단하게 말해서, 새로운 데이터에 대해 코드가 순차적이며 독립적으로 반응하도록 비동기적 프로그래밍하는 것
- rx swift는 비동기 코드를 짜는 데에 도움을 줄 것이다

## 비동기 프로그래밍 소개
- iOS는 서로 다른 작업을 다른 스레드에서 동작하도록 하여 동시에 서로 다른 작업을수행할 수 있도록 돕는, 즉 비동기 프로그래밍을 돕는 api를 제공한다

### 비동기 api
- notification center: 특정 이벤트 발생시 코드 실행. 예를 들어 디바이스 방향전환 또는 스크린 상 키보드 숨김
- delegate pattern: 다른 클래스에 의해 수행되는 메소드. 예를들어 원격 노티 발생시 수행되어야 할 메소드. 언제 몇번 수행될 지는 알지 못함
- GCD: 다수의 작업을 처리할 때 큐를 이용해서 처리방식과 우선순위를 설정할 수 있다
- closure: 클래스간 주고받을 수 있는 코드생성. 각각의 클래스는 그것을 수행할지 , 몇번 할지 어느 시점에서 할지 결정

## 1. Observable
- 다른 클래스가 만든 값을 구독할 수 있게 해준다
- Observable<T> 클래스는 실시간으로 반응하는 하나 이상의 observer를 가질수 있다
- Obervable 클래스가 준수하는 프로토콜은 꽤 간단하다. observer는 3가지 이벤트(next, completed, error)에 대해서만 받을 수 있다.
- observable을 사용하면 delegate 프로토콜이나 클로저를 사용하지 않고도 클래스간 의사소통을 가능하게 해준다.

- finite observable sequence: 특정 작업이 완료될때까지만 관찰하면 되는 것. 예를 들어 인터넷에서 파일을 다운로드한다고 가정해보자. 다운로드를 시작하고 데이터를 받는다. 네트워크 연결이 약해지는 상황에서 다운로드가 멈추고 연결은 에러와 함께 끊길 것이다. 또는 데이터 다운이 완료되면 success와 함께 마칠 것이다.
```swift
API.download(file: "http://www...")
    .subscribe(onNext: { data in
        ... append data to temporoary file
    },
    onError: { error in 
        ... display error to user
    },
    onCompleted: {
        ... use downloaded file
    }
    )
```

- infinite observable sequence: 완료 시점 없이 계속 관찰해야 하는 것. 예를 들어 디바이스 화면 회전에 반응하는 것은 언제 끝날지 모르는 작업이므로 계속 지켜봐야 함. 관찰 시작시점부터 초기값이 존재한다.
```swift
UIDevice.rx.orientation
    .subscribe(onNext: { current in
        switch current {
            case .landscape:
                ... re-arrange UI for landscape
            case .portrait:
                ... re-arrange UI for portrait
        }
    })
```
- 위 이벤트의 경우 종료되지 않으므로 onError, onCompleted 생략한다

## 2. operator
```swift
UIDevice.rx.orientation
    .filter { value in
        return value != .landscape
    }
    .map { _ in
        return "Portrait is the best"
    }
    .subscribe(onNext: { string in
        showAlert(test: string)
    })
```
- UIDevice.rx.orientation이 .landscape 또는 .portrait 값을 반환할때 마다 Rx는 filter, map 연산자를 데이터에 적용할 것이다. filter 연산자는 .landscape가 아닌 값만 통과시킨다. 만약 디바이스가 landscape 모드라면 하위 코드는 실행되지 않는다. .portrait 값이라면 map 연산자를 만나 Orientation 타입의 입력값을 받아 String 타입을 출력한다. 마지막으로 subscribe가 위 문자열을 받아서 알림창에 띄운다.
- 연산자들은 서로 퍼즐처럼 결합하여 서로 입력값과 출력값이 된다. 따라서 하나의 연산자가 할 수 있는 기능을 넘어 여러 조합으로 다양한 결과를 도출할 수 있다. 앞으로 비동기 작업과 연관된 복잡한 연산자들을 배울 것이다.

## 3. scheduler
- scheduler는 Rx의 dispatch queue와 같다. rxSwift 덕분에 복수의 작업을 다른 scheduler에 배치하여 최고의 퍼포먼스를 얻을 수 있다. 같은 subscription 이라도 여러 연산자 블록으로 나뉘고 각각의 연산자 블록은 서로 다른 scheduler 상에서 동작할 수 있다.

## 앱 아키텍쳐
- 어떤 방식으로든 rxSwift 가 앱 아키텍쳐를 바꿔서는 안된다. rxSwift는 주로 이벤트, 비동기 데이터 처리, universal communication contract을 처리한다.
- MVC 패턴을 구현하여 반응형 앱을 만들 수 있고 또는 MVVM 패턴을 구현한 반응형 앱을 만들 수도 있다.
- rxSwift는 단방향 데이터 흐름 아키텍쳐를 구현하는 데 매우 유용하다.
- 리액티브 앱을 만들기 위해서 새로 앱을 만들 필요는 없다. 기존의 프로젝트를 리팩토링 할 수 있다.
- 마이크로소프트의 MVVM 패턴은 이벤트 주도 소프트웨어에 특화되어 발전한 아키텍쳐다. rxSwift와 MVVM은 서로 궁합이 잘 맞는다. 왜냐하면 View Model이 Observable<T> 프로퍼티를 노출시켜서 뷰컨트롤러에 연결할 수 있기 때문이다. 이를 통해 모델 데이터와 UI를 간단하게 연결할 수 있다.
- 이 책의 예제는 이해하기 쉽게 하기 위해 MVC패턴을 사용한다

## RxCocoa
- RxSwift는 공통된 Rx api 구현체이므로 Cocoa나 UIkit 클래스에 대해 알지 못한다. RxCocoa 는 UIKit과 Cocoa 를 위한 개발을 돕는 클래스를 포함하는 라이브러리이다. RxCocoa는 UI 에 reactive extension을 추가하여 다양한 UI events를 구독할 수 있다.
예를 들어 RxCocoa를 사용하면 UISwitch의 상태변화를 구독하기 쉽다
```swift
toggleSwitch.rx.isOn
    .subscribe(onNext: { enabled in
        print( enabled ? "it's on" : "it's off" )
    })
```
- RxCocoa는 UISwitch 클래스에 rx.isOn 프로퍼티를 추가하여 유용한 이벤트를 구독할 수 있게 해준다. 게다가 UITextField, URLSession, UIViewController 등에도 rx를 추가할 수 있다.

## 마무리
- 이번 챕터에서는 RxSwift가 무엇인지 소개했다. 비동기 프로그래밍의 복잡성, 가변상태 공유, 부작용 야기 등에 대해서도 배웠다
- 아직 RxSwift를 사용하지 않았지만 RxSwift가 왜 필요한지, 어떤 문제들을 해결할 수 있는지 이제 잘 이해할 수 있다. 이제 아주 간단한 observable을 만들고 MVVM 패턴을 이용한 앱을 만들어보자