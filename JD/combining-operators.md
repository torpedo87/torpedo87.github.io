# Combining Operators
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

## prefixing, concatenating
- observable을 다룰 때 중요한 것은 observer가 초기값을 가져야하는 상황인지 파악하는 것이다. 예를 들어 현재위치, 현재 네트워크 연결 상태와 같이 현재 상태를 필요로하는 상황이 있다.

```swift
example(of: "startWith") {

    let numbers = Observable.of(2, 3, 4)

    let observable = numbers.startWith(1)

    observable.subscribe(onNext: { value in
        print(value)
    })
}
```

- startWith 연산자는 observable에 초기값을 추가한다. 이 값은 observable 원소와 같은 타입이다
- 1. observable 생성
- 2. observable의 초기값을 추가
- startWith 연산자가 observable 뒤에 연결되더라도 초기값은 제일 먼저 방출한다.

```swift
1
2
3
4
```

- concat 연산자는 두 observable을 연결시킨다

```swift
example(of: "Observable.concat") {

    let first = Observable.of(1, 2, 3)
    let second = Observable.of(4, 5, 6)

    let observable = Observable.concat([first, second])
    observable.subscribe(onNext: { value in
        print(value)
    })
}
```

```swift
1
2
3
4
5
6
``` 

- 위의 경우 concat은 클래스 메소드로 사용되었다. Observable.concat 은 static func 로서 배열과 같은 순서가 있는 컬렉션을 받는다.

```swift
example(of: "concat") {
    let germanCities = Observable.of("Berlin", "Minch", "Frankfurt")
    let spanishCities = Observable.of("Madrid", "Barcelona", "Valencia")

    let observable = germanCities.concat(spanishCities)
    observable.subscribe(onNext:{ value in
        print(value)
    } )
}
```  

- 이번에는 concat 이 연산자로 사용되었다.
- concat 연산자는 앞의 배열에 뒤의 배열을 추가한다. 각 배열의 원소가 방출되는 시점과는 관계없이 합친다.

```swift
example(of: "concat one element") {
    let numbers = Observable.of(2, 3, 4)

    let observable = Observable
        .just(1)
        .concat(numbers)

    observable.subscribe(onNext: { value in
        print(value)
    })
}
``` 

```swift
1
2
3
4
``` 

- 위의 startWith 연산자 얘제를 보다 읽기 쉽게 바꿔보았다

- Note: observable을 합칠 때에는 같은 타입의 원소이어야 한다. 그렇지 않으면 컴파일 에러가 발생한다

## Merging
- 각 원소들이 방출되는 순서대로 합친다

```swift
example(of: "merge") {
    let left = PublishSubject<String>()
    let right = PublishSubject<String>()

    let source = Observable.of(left.asObservable(), right.asObservalbe())

    let observable = source.merge()
    let disposable = observable.subscribe(onNext: { value in
        print(vlaue)
    })

    var leftValues = ["Berlin", "Minch", "Frankfrut"]
    var rightValues = ["Madrid", "Barcelona", "Valencia"]

    repeat {
        if arc4random_uniform(2) == 0 {
            if !leftValues.isEmpty {
                left.onNext("left: " + leftValues.removeFirst())
            }
        } else if !rightValues.isEmpty {
            right.onNext("Right: " + rightValues.removeFirst())
        }
    } while !leftValues.isEmpty || !rightValues.isEmpty
    disposable.dispose()
}
``` 

```swift
Right: Madrid
Left: Berlin
Right: Barcelona
Right: Valencia
Left: Minch
Left: Frankfurt
``` 

## combining elements
- 