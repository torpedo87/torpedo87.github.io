# Observables
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

- 이제 설치했으니 RxSwift를 사용할 준비가 되었다. 이번챕터에서는 observalbe 을 생성하고 구독하는 예제를 몇가지 해볼 것이다.

## observable 이란?
- observable, observable sequence, sequence, stream 모두 다 같은 말이다. 가장 중요한 것은 비동기적이라는 것이다. Obserables 은 방출이라 불리는 이벤트를 발생시킨다. 이 이벤트는 숫자, 커스텀타입의 인스턴스 등의 값을 갖거나 또는 탭 같은 제스쳐로 인식된다. 마블다이어그램을 보면 쉽게 이해할 수 있다. 왼쪽에서 오른쪽으로 향하는 화살표는 시간의 흐름을 나타내며 숫자가 찍힌 원은 sequence이다. 1이 방출되고 좀 이따가 2와 3이 방출될 것이다. 얼마나 많은 시간이 걸리나? observable의 생애의 어떤 지점이 될 것이다.

### observable의 생애주기
- 위에서 observable은 3개 원소를 방출했다. observable이 원소를 방출할 때 next 이벤트가 발생한다
- 이번에는 수직 바가 존재하는 마블 다이어그램이다. 수직 바는 observable의 끝을 의미한다
이 observable은 3개의 이벤트를 방출하고 마친다. 종료되므로 completed event 라 불린다. 예를들어 뷰가 디스미스되면 더이상 탭을 할 수 없는 것이다. 이제 observable은 더이상 방출할 수가 없다. 이는 정상적인 종료이다
- 그러나 에러가 발생하여 종료하는 경우도 존재한다. 이번 다이어그램에서는 에러가 발생했다. 빨간색 x 표시부분이다. observable은 에러 이벤트를 방출했다. 정상종료할 때 completed 이벤트 방출하는 것 이외에는 다를 게 없다. observable이 에러이벤트를 방출하면 종료되어 더이상 방출하지 않는다.

- 요약해보면, observable은 원소를 포함하는 next 이벤트를 방출한다. 에러 이벤트를 방출하거나 정상종료할 때까지 말이다. observable이 종료되면 더이상 이벤트 방출하지 않는다
```swift
public enum Event<Element> {
    case next(Element)
    case error(Swift.Error)
    case completed
}
```
- next 이벤트는 원소를 포함한다. 에러 이벤트는 Swift.Error를 포함하고 completed 이벤트는 어떤 데이터도 포함하지 않는 stop 이벤트이다.
- 이제 observable을 생성해보자

## observable 생성하기
- observable을 생성하는 데 사용하는 3가지 연산자를 알아보자
```swift
example(of: "just, of, from") {
    let one = 1
    let two = 2
    let three = 3

    let observable: Observable<Int> = Observable<Int>.just(one)
}
```
- 1. 예제로 사용할 integer 상수를 3개 만들었다
- 2. just 메소드를 갖는 정수 타입의 Observable을 만들었다
- just 메소드가 하는 일은 그저 하나의 원소를 포함하는 observable을 만드는 것이다. just 는 Observable이 갖는 타입 메소드 이다. 그러나 Rx에서 메소드는 operator로 불린다.

```swift
let observable2 = Observable.of(one, two, three)
```
-이번에는 타입을 지정하지 않았지만 옵션키를 눌러 들어가보면 타입추론이 Integer로 이미 되어있다. 연산자는 이처럼 원소를 통해 타입추론을 한다.

```swift
let observable3 = Observable.of([one, two, three])
```
- 배열 타입을 받고 싶다면 of 연산자를 사용하면 된다
- 위의 경우 추론 타입은 [Int]이 된다. just 연산자는 하나의 원소를 지닐 수 있지만 하나의 배열을 하나의 원소로 간주할 수 있다.

```swift
let observable4 = Observable.from([one, two, three])
```
- from 연산자는 오직 배열만을 담는 연산자로서 배열 안의 원소만을 타입추론한다. 따라서 추론된 타입은 [Int]가 아니라 Int 이다.

## observable 구독하기
- ios 개발자로서 notificationcenter 에 익숙할 것이다. 이것은 observer에게 신호를 보내는데 이것은 RxSwift의 observable과 다른점이다. 다음 UIKeyboardDidChangeFrame 예제를 살펴보자.
```swift
let observer = NotificationCenter.default.addObserver(
    forName: .UIKeyboardDidChangeFrame,
    object: nil,
    queue: nil
) { notification in
}
```
- RxSwift의 observable 구독 또한 위의 것과 비슷하다. addObserver() 대신에 subscribe()를 사용한다. NotificationCenter는 .default라는 싱글톤 인스턴스를 사용하는 반면, Rx에서 observable은 각각 다른 인스턴스이다.
- 중요한 것은, observable이 subscriber를 갖기 전까지 이벤트를 보내지 않는다. observable 구독은 스위프트 표준 라이브러리의 Iterator의 next()를 호출하는 것과 같다

```swift
let sequence = 0..<3
var iterator = sequence.makeIterator()
while let n = iterator.next() {
    print(n)
}
```
- observable 구독은 위 보다 더 간소하며 handler 를 추가할 수 있다. observable이 .next, .error, .complted 이벤트를 방출할 수 있다는 것을 기억하자. .next 이벤트는 방출된 원소를 handler 에 전달한다. .error  이벤트는 error 인스턴스를 갖는다.
```swift
example(of: "subscribe") {
    let one = 1
    let two = 2
    let three = 3

    let observable = Observable.of(one, two, three)

    observable.subscribe { event in
        print(event)
    }
}
```
- 앞의 예제와 흡사하다. 대신 이번에는 간편하게 of 연산자를 사용했다. subscribe 연산자를 옵션 클릭해보면 연산자가 Int 타입의 이벤트를 갖는 탈출클로저를 갖는다. 그리고 연산자는 Disposable을 반환한다. 
- 프린트 결과를 살펴보자
```swift
next(1)
next(2)
next(3)
completed
```
- obervable은 각각의 원소에 대한 .next 이벤트를 방출한다. 그리고 나서 .complted 이벤트를 방출하고 종료한다. obervable을 다룰 때, 이벤트 자체보다는 .next 이벤트에 의해 방출된 원소에 관심이 있을 것이다.
```swift
observable.subscribe { event in
    if let element = event.element {
        print(element)
    }
}
```
- Event 객체는 element 프로퍼티를 갖는다. .next 이벤트 객체만 element 값을 가지므로 elemtent는 옵셔널 값이다. 따라서 옵셔널 바인딩을 사용해서 nil 이 아닌 경우의 element 를 벗겨내야한다. 이제 이벤트의 원소값만 출력된다. .completed 이벤트는 element 값이 nil이다
```swift
1
2
3
```
- 위의 코드를 옵셔널 바인딩 없이 더 간략하게 수정해보자
```swift
observable.subscribe(onNext: { element in
    print(element)
})
```
- .next 이벤트만 다루므로 옵셔널 바인딩이 필요없어졌다.
- empty elemnent 의 경우 어떻게 observable 을 생성할까? empty 연산자는 empty observable을 만든다. 이는 .completed 이벤트만 방출한다
```swift
example(of: "empty") {
    let observable = Observable<Void>.empty()
}
```
- observable은 타입 추론이 안될 경우 특정 타입을 지정해주어야한다.. 위 경우 empty 는 타입추론이 안되므로 Void 타입을 사용한다.

```swift
observable
    .subscribe(
        onNext: { element in
            print(element)
        },
        onCompleted: {
            print("Completed")
        }
    )
```
- 1. .next 이벤트를 다루는 것은 이전 예제와 같다
- 2. .completed 이벤트는 element 를 방출하지 않으므로 단순히 메시지만 출력하기로 하자
```swift
Completed
```
- empty observable은 대체 어디에 사용될까? 즉시 종료되거나 의도적으로 zero 값을 갖는 observable을 반환하고 싶을 때 유용하다.
- empty 연산자와는 반대로, never 연산자는 아무것도 방출하지 않으며 종료되지 않는 observable을 생성한다. 무한대를 나타낸다.
```swift
example(of: "never") {
    let observable = Observable<Any>.never()

    observable
        .subscribe(
            onNext: { element in
                print(element)
            },
            onCompleted: {
                print("Completed")
            }
        )
}
```
- 위의 경우 아무것도 출력되지 않는다.
- 이번에는 영역의 값으로부터 observable 을 생성하는 것을 알아보자.
```swift
example(of: "range") {

    let observable = Observable<Int>.range(start: 1, count: 10)

    observable
        .subscribe(onNext: { i in
            
            let n = Double(i)
            let fibonacci = Int(((pow(1.61803, n) - pow(0.61803, n)) /
                            2.23606).rounded())
            print(fibonacci)
        })
}
```
- 1. range 연산자를 사용해 observable을 생성했다.
- 2. n번째 피보나치 수를 계산하고 출력한다
- 위의 예제에서 onNext 핸들러를 사용하는 것보다 더 좋은 방법이 있는데 이는 7장에서 다루기로 하자.
- never() 연산자를 제외하고는 자동으로 .completed 이벤트를 방출하고 정상종료한다.

## 버리기(취소), 종료하기
- observable은 구독하기 전에는 아무것도 할 수 없다. 구독을 하면 에러 이벤트나 완료이벤트 발생 전 혹은 정상종료 전까지 이벤트를 방출한다. 구독을 취소해서 수동으로 observable을 종료할 수도 있다
```swift
example(of: "dispose") {

    let observable = Observable.of("A", "B", "C")
    let subscription = observable.subscribe { event in
        print(event)
    }
}
```
- 1. String 타입을 다루는 observable을 생성한다
- 2. 구독을 한다. 이번에는 subscribe가 반환하는 Disposable을 subscription이라는 상수에 담아놓는다
- 방출되는 이벤트를 출력한다

- 구독을 취소하기 위해서는 dispose()를 호출해야한다. 구독을 취소한 후에는 observable이 이벤트 방출을 멈춘다.
```swift
subscription.dispose()
```
- 각각의 observable을 취소하는 것은 지루한 작업이다. RxSwift는 DisposeBag 타입을 갖는데 이것은 .addDisposableTo() 메소드를 이용해 disposable을 담아놓을 수 있다. 그리고 dispose bag 이 deallocate 될 때 각각 dispose()가 호출된다. 
```swift
example(of: "DisposeBag") {

    let disposeBag = DisposeBag()

    Observable.of("A", "B", "C")
        .subscribe {
            print($0)
        }
        .addDisposableTo(disposeBag)
}
```
- 1. dispose bag 만들기
- 2. observable 만들기
- 3. observable 구독하고 이벤트 출력하기
- 4. subscribe가 반환하는 값을 disposeBag에 담기
- 위의 경우는 자주 사용할 패턴이다. observable 생성하고 구독하고 dispose bag에 넣기
- 만약 dispose bag에 담지 않으면 구독이 마칠때마다 수동으로 dispose 를 호출해야한다. 그렇지 않으면 메모리 누수가 발생한다. 하지만 걱정하지 마라. 스위프르 컴파일러가 경고해 줄 것이다.
- 이전 예제에서는 .next 이벤트를 방출하는 observable을 만들었다. observable이 방출하는 모든 이벤트를 다루는 또다른 방법은 create 연산자를 사용하는 것이다.
```swift
example(of: "create") {

    let disposeBag = DisposeBag()
    Observable<String>.create { observer in
    
    }
}
```
- create 연산자는 subscribe 파라미터를 갖는다. observable에 대해 subscribe를 호출하는 것을 담당한다. 다시맗하면 방출될 모든 이벤트를 설명한다.
- subscribe 파라미터는 AnyObserver 를 갖는 탈출클로저이며 Disposable을 반환한다. AnyObserver는 observable에 값을 넣는 제네릭 타입이며 방출될 것이다. 
```swift
Observable<String>.create { observer in

    observer.onNext("1")
    observer.onCompleted()
    observer.onNext("?")
    return Disposables.create()
}
```
- 1. observer 에 .next()를 추가한다
- 2. observer 에 .completed를 추가한다
- 3. 다시 .next 이벤트를 추가한다
- 4. disposable을 반환한다
- Note: subscribe 연산자는 disposable을 반환한다. 여기서 Disposables.create() 는 empty disposable이지만 어떤 disposable은 side effect를 갖는다

- 두번째 onNext 원소인 ? 는 방출될 수 있을까? 확인하기 위해 다음 코드를 추가하자
```swift
.subscribe(
  onNext: { print($0) },
  onError: { print($0) },
  onCompleted: { print("Completed") },
  onDisposed: { print("Disposed") }
)
.addDisposableTo(disposeBag)
```
- 우리는 observable을 구독하고 모든 핸들러를 구현했다. 첫번째 .next 이벤트는 "Completed"와 "Disposed"를 출력한다. 두번째 .next 이벤트는 출력하지 않는다 왜냐하면 observable이 이미 .completed 이벤트를 방출헤서 종료했기 때문이다
```swift
1
Completed
Disposed
```

- observer 에 error 를 추가하면 어떻게 될까? 
```swift
enum MyError: Error {
    case anError
}
```
- 다음과 같이 Error 타입을 만들었다. 아래의 코드를 observer.onNext와 observer.onCompleted 호출하는 사이에 넣자
```swift
observer.onError(MyError.anError)
```
- observer는 에러를 방출하고 종료된다
```swift
1
anError
Disposed
```
- 만약 .completed 와 .error 이벤트를 모두 방출하지 않고 disposeBag에 담지 않으면 어떨까? 다음과 같이 주석처리해보자
```swift
example(of: "create") {
  enum MyError: Error {
    case anError
  }
  let disposeBag = DisposeBag()
  Observable<String>.create { observer in
    // 1
    observer.onNext("1")
//    observer.onError(MyError.anError)
    // 2
//    observer.onCompleted()
// 3
    observer.onNext("?")
// 4
    return Disposables.create()
  }
  .subscribe(
    onNext: { print($0) },
    onError: { print($0) },
    onCompleted: { print("Completed") },
    onDisposed: { print("Disposed") }
)
//  .addDisposableTo(disposeBag)
}
```
- 축하한다 이제 메모리 누수가 발생한다. observable 은 종료되지 않고 disposable은 처리되지 않는다. 다음과 같이 출력될 것이다.
```swift
1
?
```
- 누수를 다시 막으려면 .completed 또는 addDisposableTo를 주석해제하면 된다

## observable factory 만들기
- 구독을 기다리는 observable을 만드는 것 이외에 observable을 만들어 구독을 바로 연결해주는 observable factory를 생성할 수 있다.
```swift
example(of: "deferred") {
    let disposeBag = DisposeBag()

    var flip = false

    let factory: Observable<Int> = Observable.deferred {
        flip = !flip

        if flip {
            return Observable.of(1, 2, 3)
        } else {
            return Observable.of(4, 5, 6)
        }
    }
}
```
- 어떤 observable을 반환할지 결정하는 flip Bool값 생성
- deferred 연산자를 사용해서 observable factory 생성
- flip 상태가 바뀐다. factory가 구독될 때 flip 상태가 사용된다
- flip에 따라서 다른 observable을 반환한다
- 외부적으로는 observable factory는 보통의 observable과 분별되지 않는다. 아래의 코드를 추가하자
```swift
for _ in 0...3 {
    factory.subscribe(onNext: {
        print($0, terminator: "")
    })
    .addDisposableTo(disposeBag)
    print()
}
```
- 팩토리가 각각 구독할 때마다 다른 observable이 반환된다. 새로운 구독이 생성될때마다 123, 456 패턴이 반복된다
```swift
123
456
123
456
```

## 도전과제