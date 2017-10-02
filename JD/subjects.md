# Subjects
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

- 지금까지 observable을 다루는 방법을 배웠다. 생성하고 구독하고 완료되었을 때 처리하는 방법을 배웠다. observable은 RxSwift의 기본이다. 하지만 우리는 앱을 만들 때 observable과 observer 로서 기능할 수 있는 무언가가 필요하다. 이는 Subject라고 불린다
- 이번 챕터에서 다양한 타입의 subject에 대해 배워보자

## 시작하기

```swift
example(of: "PublishSubject") {

    let subject = PublishSubject<String>()
}
```

- PublishSubject는 신문 발행과 같다. 정보를 받아서 구독자들에게 발행한다. 위에서는 String 타입만 받고 전달할 수 있다. 생성된 후에 정보를 받을 준비가 된다.
- example 안에 다음을 추가하자

```swift
subject.onNext("Is anyone listening?")
```

- subject에 새로운 단어를 넣었다. 하지만 아직 아무것도 출력되지 않는다. 옵저버가 없기 때문이다. example에 다음을 추가하여 옵저버를 만들자

```swift
let subscriptionOne = subject
    .subscribe(onNext: { string in
        print(string)
    })
```

- 이전 챕터에서 .next 이벤트를 출력하는 것과 같은 subscription을 만들었다. 하지만 여전히 아무것도 출력되지 않는다. 재미 없으니 간략하게 다른 subject 를 알아보자
- 여기서 PublishSubject는 오로지 현재 구독자에게만 방출한다. 이전에 무언가가 추가되었을 때 구독자가 아닌 상태였다면, 현재 구독을 하더러도 이전의 정보를 받지는 못한다. 이를 수정하기 위해 다음을 example 끝에 추가하자

```swift
subject.on(.next("1"))
```

- String 타입을 구독하기로 선언했으므로 오로지 String만 받을 수 있다. 이제 subject는 구독자가 있기 때문에 다음과 같이 텍스트를 방출할 수 있다

```swift
1
```

- subscribe 연산자와 비슷하게 on(.next(_:)) 는 새로운 .next 이벤트를 추가하고 파라미터로 원소를 전달하는 방식이다. 또한 subscribe 처럼 약식 문법이 존재한다. 다음을 예제에 추가하자

```swift
subject.onNext("2")
```

- onNext(_:) 는 on(.next(_)) 와 같다. 조금 더 쉬워 보인다. 이제 2도 출력된다

```swift
1
2
```

## subject 란 무엇인가
- subject는 observable 과 observer 기능을 한다. 앞에서 어떻게 subject가 이벤트를 받고 구독되는지 살펴보았다. subject는 .next 이벤트를 받아서 매번 구독자에게 방출한다.

- RxSwift에서는 4가지 subject 타입이 존재한다
- 1. PublishSubject: 비어있는 상태로 시작하고 구독자에게 오로지 새로운 원소만 방출 
- 2. BehaviorSubject: 초기값과 함께 시작하고 새로운 구독자에게 최신 원소를 방출
- 3. ReplaySubject: 버퍼 사이즈로 초기화되며 그 사이즈까지 유지하고 새 구독자에게 방출
- 4. Variable: BehaviorSubject를 담아서 현재 값을 유지하고 새로운 구독자에게 최신 값을 방출

### 1. PublishSubject
- 구독 시작시점부터 구독을 종료할 때 까지 새로운 이벤트를 알려주는 것이다. 마블 다이어그램에서 가장윗줄은 publish subject 이고 두번째줄과 세번째줄은 구독자이다. 위로 향하는 화살표는 구독을 나타내며 아래로 향하는 화살표는 이벤트 방출을 나타낸다.
- 첫번째 구독자는 1 이후에 구독하므로 1은 받지 못하고 2, 3 을 받는다. 두번째 구독자는 2 이후에 구독을 시작하므로 3만 받는다
- 예제 끝에 다음을 추가하자

```swift
let subscriptionTwo = subject
    .subscribe { event in
        print("2)", event.element ?? event)
    }
```

- 이벤트는 옵셔널 원소 프로퍼티를 갖으며 이는 .next 이벤트에 방출되는 원소이다. 여기서는 nil 병합 연산자를 사용하여 원소가 존재하면 이를 출력하고 없으면 그냥 이벤트를 출력하도록 했다
- 예상대로 subscriptionTwo 는 출력하지 않는다 왜냐하면 1, 2 가 방출된 이후에 구독을 시작했기 때문이다. 이제 다음을 추가하자

```swift
subject.onNext("3")
```

- subscriptionOne과 subscriptionTwo에서 각각 3이 출력된다

```swift
3
2) 3
```

- 이제 구독1을 취소하기 위해 다음을 추가하고 새로운 이벤트를 방출해보자

```swift
subscriptionOne.dispose()
subject.onNext("4")
```

- 구독1이 취소되었으므로 구독2에서만 4가 출력된다

```swift
2) 4
```

- publish subject가 .completed 또는 .error 이벤트를 받으면 stop 이벤트를 방출하고 더이상 .next 이벤트를 방출하지 않는다. 그러나 미래의 구독자에게 stop 이벤트를 방출할 것이다. 예제에 다음을 추가하자

```swift
subject.onCompleted()

subject.onNext("5")

subscriptionTwo.dispose()

let disposeBag = DisposeBag()

subject
    .subscribe {
        print("3)", $0.element ?? $0)
    }
    .addDisposableTo(disposeBag)

 subject.onNext("?")   
```

- 1. 편의 연산자를 사용해서 subject에 .completed 이벤트 넣는다. 성공적으로 observable을 종료시킨다
- 2. subject에 새로운 next 이벤트 넣는다. 그러나 이미 subject가 종료되었기 때문에 이벤트가 방출되거나 출력되지는 않는다
- 3. 마친 구독은 disposeBag에 넣는다
- 4. 새로운 구독을 생성하고 disposeBag에 넣는다
- 새로 생성한 구독이 subject를 다시 활성화시킬까? 아니다. 하지만 .completed 이벤트를 얻을 것이다.

```swift
2) completed
3) completed
```

- 모든 subject 타입은 종료가 되면 미래 구독자에게 stop 이벤트를 방출할 것이다. 따라서 단순히 종료를 알리는 것 뿐만 아니라 stop 이벤트에 핸들러를 포함시키는 것은 좋은 생각이다.
- 온라인 경매 앱처럼 시간에 민감한 데이터를 모델링할 때 publish subject를 사용한다. 10시 1분에 참여한 사용자에게 경매가 1분 남았다고 알리는 것은 이상할 것이다.

- 원소가 구독 이전에 방출되었을지라도, 때때로 새로운 구독자들에게 최신 원소값을 알리고 싶을 때가 있을 것이다. 이를 위한 옵션이 바로 behavior subject이다.

## BehaviorSubject
- Bahavior subject는 publish subject와 유사하게 동작하지만 새로운 구독자에게 최신의 .next 이벤트를 재생해준다. 마블 다이어그램을 살펴보자
- 첫째줄은 subject 이다. 두번째 줄의 첫번째 구독자는 1 이후, 2 이전에 구독을 시작한다. 따라서 즉시 1을 받고 그리고나서 2, 3을 받는다. 유사하게 두번째 구독자는 2 이후, 3 이전에 구독을 시작하는데 이전 이벤트 중 가장 최근 이벤트인 2를 즉시 받고 그리고 나서 3을 받는다
- 다음의 새로운 예제를 추가하자

```swift
enum MyError: Error {
    case anError
}

func print<T: CustomStringConvertible>(label: String, event: Event<T>) {
    print(label, event.element ?? event.error ?? event)
}


example(of: "BehaviorSubject") {
    
    let subject = BehaviorSubject(value: "Initial value")
    let disposeBag = DisposeBag()
}
```

- 1. 다음 예제에서 사용될 error 타입을 정의
- 2. 이전 예제에서 사용한 3항 연산자를 확장시켜서 여기서는 존재하는 경우, 에러인 경우, nil 인경우로 원소를 출력하는 helper 메소드를 만들었다
- 3. 새로운 예제 시작
- 4. 새로운 behavior subject 를 생성하고 초기값에 initial value를 넣는다
- Note: BehaviorSubject 는 항상 이전의 최신 원소를 방출하므로 초기값 없이는 인스턴스를 생성할 수가 없다. 만약 초기값이 없다면 PublishSubject를 대신 사용해야 할 것이다.

- 예제에 다음을 추가하자

```swift
subject
    .subscribe {
        print(label: "1)", event: $0)
    }
    .addDisposableTo(disposeBag)
```

- 첫번째 구독자를 생성했지만 subject에 아직 어떤 원소도 추가되지 않았다. 따라서 subject는 구독자에게 초기값을 보낸다. 다음과 같이 출력된다

```swift
1) Initial value
```

- 첫번째 구독 이전에 다음 이벤트를 추가하자 

```swift
subject.onNext("X")
```

- X 가 출력된다 왜냐하면 이제 최신 원소가 초기값이 아닌 X 이기 때문이다

```swift
X
```

- 예제 끝에 다음을 추가하고 무엇이 출력될지 생각해보자

```swift
subject.onError(MyError.anError)

subject
    .subscribe {
        print(label: "2)", event: $0)
    }
    .addDisposableTo(disposeBag)
```

- 1. subject에 에러 이벤트 추가
- 2. 두번째 구독 생성
- 에러 이벤트가 다음과 같이 두번 출력된다

```swift
1) anError
2) anError
```

- Behavior subject 는 최신 데이터로 뷰를 재구성하고 싶을 때 유용하다. 예를 들어 사용자 프로필 화면에 사용해서 최신의 프로필 상태로 업데이트할 수 있다.
- 그러나 만약 최신 값 이외의 것을 보여주고 싶을 때에는?? 예를 들어 검색화면에서 최근 검색어 5개를 보여주고 싶다면??? 이런 경우 ReplaySubject를 사용하자

## ReplaySubject
- Replay subject는 최신 원소들을 임시저장했다가 특정 크기가 찼을 때 방출한다. 새로운 구독자에게 이것들을 방출한다. 다음 마블 다이어그램은 버퍼 사이즈가 2인 경우를 묘사한다. 첫번째 구독자는 이미 subject를 구독하는 상태이므로 subject가 이벤트를 방출할때마다 원소를 받는다. 두번째 구독자는 2 이후에 구독을 시작하는데 버퍼 사이즈가 2 이므로 최근 2개 원소인 1, 2 를 즉시 받고 그리고 나서 3을 받는다.
- replay subject를 사용할 때 임시저장 버퍼는 메모리에 담긴다. 이미지 같은 큰 메모리를 차지하는 타입의 replay subject를 위해 큰 버퍼 사이즈를 설정하면 자신의 발등을 찍는 행위이다. 또 조심할 것은 배열을 다루는 replay subject를 생성하는 것이다. 각각 방출된 원소는 배열이 되고 버퍼 사이즈는 수많은 배열을 기억할 것이다. 만약 조심하지 않으면 메모리 압력을 만들기 쉬울 것이다
- 새로운 예제를 추가하자

```swift
example(of: "ReplaySubject") {

    let subject = ReplaySubject<String>.create(bufferSize: 2)

    let disposeBag = DisposeBag()

    subject.onNext("1")
    subject.onNext("2")
    subject.onNext("3")

    subject
        .subscribe {
            print(label: "1)", event: $0)
        }
        .addDisposableTo(disposeBag)

    subject
        .subscribe {
            print(label: "2)", event: $0)
        }
        .addDisposableTo(disposeBag)
}
```

- 1. 버퍼 크기가 2인 새로운 replay subject 생성한다
- 2. subject에 2개의 원소를 추가한다
- 3. 두개의 구독을 생성한다
- 최근 2개의 원소는 두 구족자들에게 재생된다. 1은 방출되지 않는다 왜냐하면 구독 이전 최근 2개의 원소는 2, 3 이기 때문이다

```swift
1) 2
1) 3
2) 2
2) 3
```

- 예제에 다음을 추가하자

```swift
subject.onNext("4")

subject
    .subscribe {
        print(label: "3)", event: $0)
    }
    .addDisposableTo(disposeBag)
```

- subject에 새로운 원소 4를 추가했다. 그리고 세번째 구독을 시작했다. 앞의 두 구독은 원소 4를 정상적으로 받을 것이다. 왜냐하면 이미 구독상태이기 때문이다. 반면 세번째 구독자는 최근 두 원소인 3, 4 만 받을 것이다

```swift
1) 4
2) 4
3) 3
4) 4
```

- subject에 4를 추가한 코드 바로 다음, 세번째 구독 이전에 다음 코드를 추가하자

```swift
subject.onError(MyError.anError)
```

- 다음과 같이 출력될 것이다

```swift
1) 4
2) 4
1) anError
2) anError
3) 3
3) 4
3) anError
```

- replay subject는 에러로 종료가 되고 새 구독자에게 다시 최근 2개 원소와 에러를 방출한다. 버퍼가 여전히 메모리에 저장되어 있기 때문이다. 에러 이벤트 바로 다음에 다음 코드를 추가하자

```swift
subject.dispose()
```

- replay subject 를 취소하기 위해 dispose()를 호출하면 새로운 구독자는 에러이벤트만 받게 된다.

```swift
3) Object `RxSwift.ReplayMany<Swift.String>` was already disposed.
```

- dispose bag에 구독을 넣으면 모든 것이 취소되고 메모리에 있던 참조도 해제된다.
- Note: ReplayMany 는 replay subject를 생성할 때 사용되는 내부타입이다
- publish, behavior, replay subject를 사용해서 대부분의 모델링을 할 수 있다. 그래도 만약 observable 타입에게 현재 값이 무엇이냐고 묻는다면 Variable을 사용해야 한다

## Variable
- 앞에서 언급했듯이 variable은 BehaviorSubject를 담고 현재 값을 저장한다. value 프로퍼티를 통해 현재 값에 접근할 수 있다. 다른 subject와 observable 과는 다르게, variable에 새로운 원소를 설정하기 위해 value 프로퍼티를 사용한다. 즉 onNext(_:)를 사용하지 않는다.
- behavior subject를 담기 때문에 variable은 초기값과 함께 생성이 되며 가장 최근 값을 새로운 구독자에게 재생한다. variable 의 bahavior subject에 접근하기 위해서 asObservable()을 호출한다.
- Variable의 또다른 특징은 에러를 방출하지 않는다는 점이다. 구독이 에러 이벤트를 들을 수 있더라도 variable에 에러 이벤트를 추가할 수 없다. variable은 또한 참조해제될 때 자동으로 종료되므로 수동으로 .completed 이벤트를 추가할 필요가 없다
- 다음 예제를 추가하자

```swift
example(of: "Variable") {

    var variable = Variable("Initial value")

    let disposeBag = DisposeBag()

    varable.value = "New initial value"

    variable.asObservable()
        .subscribe {
            print(label: "1)", event: $0)
        }
        .addDisposableTo(disposeBag)
}
```

- 1. 초기값과 함께 varable을 생성한다. variable 타입은 추론되지만 Variable<String>("Initial value") 라고 직접 선언해줘도 된다
- 2. vairable의 value에 새로운 원소를 추가한다
- 3. 구독을 시작하고 이 때 behavior subject에 접근하기 위해 asObservable()을 호출한다
- 구독은 다음과 같이 최근 값을 받는다

```swift
1) New inital value
```

- 예제에 다음을 추가하자

```swift
variable.value = "1"

variable.asObservable()
    .subscribe {
        print(label: "2)", event: $0)
    }
    .addDisposableTo(disposeBag)

 variable.value = "2"   
```

- 1. variable에 새 원소 1 을 추가한다
- 2. 두번째 구독을 시작한다
- 3. variable에 새 원소 2를 추가한다

- 기존의 첫번째 구독은 새 원소 1을 받는다. 두번째 구독은 가장 최근 값인 1을 즉시 받는다. 그리고 두 구독 모두 새로운 값인 2를 받는다.

```swift
1) 1
2) 1
1) 2
2) 2
```

- variable에는 .error 나 .completed 이벤트를 추가할 방법이 없다. 시도하면 컴파일 에러가 뜰것이다. 

```swift
// These will all generate errors
variable.value.onError(MyError.anError)
variable.asObservable().onError(MyError.anError)
variable.value = MyError.anError
variable.value.onCompleted()
variable.asObservable().onCompleted()
```

- variable은 기능이 다양하다. 다른 subject처럼 새로운 이벤트가 방출될 때마다 반응하는 observable을 구독할 수 있다. 최근 값 업데이트 없이 그냥 현재 값만 확인하고 싶을 때에도 사용할 수 있다.