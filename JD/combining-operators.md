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

- 초기값을 추가한 observable을 반환
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

- 복수의 observable의 원소를 순서대로 연결한 observable을 반환
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

- concatMap: 원소를 변형한 후 합친 observable을 반환

```swift
example(of: "concatMap") {
  // dictionary 타입
  let sequences = [
    "Germany": Observable.of("Berlin", "Münich", "Frankfurt"),
    "Spain": Observable.of("Madrid", "Barcelona", "Valencia")
  ]

  // 나라명 observable을 도시명 observable로 변형후 합치기
  let observable = Observable.of("Germany", "Spain")
    .concatMap { country in sequences[country] ?? .empty() }

  // 3
  _ = observable.subscribe(onNext: { string in
      print(string)
    })
}

```



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
- merge: observable의 모든 원소를 합한 새로운 observable 반환
- 각 원소들이 방출되는 순서대로 합친다

```swift
example(of: "merge") {
    let left = PublishSubject<String>()
    let right = PublishSubject<String>()

    //observalbe타입의 원소를 갖는 observable 생성
    let source = Observable.of(left.asObservable(), right.asObservalbe())

    let observable = source.merge()

    //합친 observable을 구독
    let disposable = observable.subscribe(onNext: { value in
        print(vlaue)
    })

    var leftValues = ["Berlin", "Minch", "Frankfrut"]
    var rightValues = ["Madrid", "Barcelona", "Valencia"]

    repeat {
        if arc4random_uniform(2) == 0 {
            if !leftValues.isEmpty {
                //observable에 이벤트 전달
                left.onNext("left: " + leftValues.removeFirst())
            }
        } else if !rightValues.isEmpty {
            //observable에 이벤트 전달
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
- combineLatest: observable의 모든 최신원소를 resultSelector 연산을 통해 합친 후 그것을 포함하는 observable을 반환
- observable이 원소를 방출할 때마다 클로저 호출
- 합쳐질 observable이 모두 원소를 하나 방출하기 전까지는 아무일도 발생하지 않는다
- result selector를 사용하면 서로 다른 타입도 합칠 수 있는 연산자가 된다

```swift
example(of: "combineLatest") {
  let left = PublishSubject<String>()
  let right = PublishSubject<String>()

  // observable의 모든 최신원소를 resultSelector 연산을 통해 합친 후 그것을 포함하는 observable을 반환
  let observable = Observable.combineLatest(left, right, resultSelector: {
    lastLeft, lastRight in
    "\(lastLeft) \(lastRight)"
  })

  //구독생성
  let disposable = observable.subscribe(onNext: { value in
    print(value)
  })

  // 2
  print("> Sending a value to Left")
  left.onNext("Hello,")
  print("> Sending a value to Right")
  right.onNext("world")
  print("> Sending another value to Right")
  right.onNext("RxSwift")
  print("> Sending another value to Left")
  left.onNext("Have a good day,")

//구독해지
  disposable.dispose()
}
```

```swift
> Sending a value to Left
> Sending a value to Right
Hello, world
> Sending another value to Right
Hello, RxSwift
> Sending another value to Left
Have a good day, RxSwift
```

- 아래와 같이 서로 다른 두 타입을 합쳐서 새로운 타입의 observable로 변형가능

```swift
example(of: "combine user choice and value") {
  //서로 다른 타입의 observable
  let choice : Observable<DateFormatter.Style> = Observable.of(.short, .long)
  let dates = Observable.of(Date())
  
  //서로 다른 두 타입을 합친 String타입의 observable
  let observable = Observable.combineLatest(choice, dates) {
    (format, when) -> String in
    let formatter = DateFormatter()
    formatter.dateStyle = format
    return formatter.string(from: when)
  }

  observable.subscribe(onNext: { value in
    print(value)
  })
}
```

- zip 연산자: left observable이 complete 되어서 right observable은 Vieena를 방출하기 전에 complete 된다 indexed sequencing 이라고 한다

```swift
example(of: "zip") {
  enum Weather {
    case cloudy
    case sunny
  }
  
  //다른 타입의 observable
  let left: Observable<Weather> = Observable.of(.sunny, .cloudy, .cloudy, .sunny)
  let right = Observable.of("Lisbon", "Copenhagen", "London", "Madrid", "Vienna")
  
  //합칠 때 쌍이 맞아야 되는건가
  let observable = Observable.zip(left, right) { weather, city in
    return "It's \(weather) in \(city)"
  }
  observable.subscribe(onNext: { value in
    print(value)
  })
}

```

# Triggers 연산자
- UI 작업시 유용

- withLatestFrom(_:) 연산자: 특정 trigger가 발생할 때에만 observable로부터 방출된 최신값을 원할 때 사용
- trigger observable이 원소를 방출할 때마다 data observable 과 trigger observable을 합친 하나의 observable을 반환

```swift
example(of: "withLatestFrom") {
  
  // 서로다른 타입의 subject
  let button = PublishSubject<Void>()
  let textField = PublishSubject<String>()

  //sample 연산자는 button 옵저버가 이벤트를 전달할 때에 textfiled옵저버로부터 받은 최신 원소를 방출하는 observable을 반환
  //trigger는 button 옵저버
  //withLatestFrom 연산자의 파라미터는 data observable인 textfiled
  let observable = button.withLatestFrom(textField)
  _ = observable.subscribe(onNext: { value in
    print(value)
  })


  // 3
  textField.onNext("Par")
  textField.onNext("Pari")
  textField.onNext("Paris")
  button.onNext(())
  button.onNext(())
}
```

```swift
Paris
Paris
```


- 위의 withLatestFrom 연산자를 sample 연산자로 교체해보자
- sample 연산자를 사용하면 버튼이 trigger 하더라도 textField 옵저버가 전달하는 값이 변화가 없으면 아무일도 일어나지 않는다
- sample = distinctUntilChanged() + withLatestFrom(_:)

```swift
example(of: "sample") {
  
  // 서로다른 타입의 subject
  let button = PublishSubject<Void>()
  let textField = PublishSubject<String>()

  //sample 연산자는 button 옵저버가 이벤트를 전달할 때에 textfiled옵저버로부터 받은 최신 원소를 방출하는 observable을 반환
  //sample 연산자의 파라미터는 trigger observable인 button
  let observable = textField.sample(button)
  _ = observable.subscribe(onNext: { value in
    print(value)
  })

  // 3
  textField.onNext("Par")
  textField.onNext("Pari")
  textField.onNext("Paris")
  button.onNext(())
  button.onNext(())
}
```

```swift
Paris
```


# switching 연산자
- amb (ambiguous 둘 중 뭘 구독해야할지 애매할 때)
- 둘다 구독신청 했다가 먼저 나오는걸 계속 구독하고 다른 하나를 해지

```swift
example(of: "amb") {
  let left = PublishSubject<String>()
  let right = PublishSubject<String>()

  // 둘 다 구독하지만 둘 중 먼저 이벤트 발생 시키는 거를 계속 구독하고 다른 하나는 바로 해지
  let observable = left.amb(right)
  let disposable = observable.subscribe(onNext: { value in
    print(value)
  })

  // left가 먼저 원소 방출했으므로 right를 구독해지한다
  left.onNext("Lisbon")
  right.onNext("Copenhagen")
  left.onNext("London")
  left.onNext("Madrid")
  right.onNext("Vienna")

  disposable.dispose()
}
```

- switchLatest: 가장 최근의 구독만 활성화

```swift
example(of: "switchLatest") {
  // 1
  let one = PublishSubject<String>()
  let two = PublishSubject<String>()
  let three = PublishSubject<String>()
  
  //observable을 원소로 갖는 subject
  let source = PublishSubject<Observable<String>>()

  //source의 원소중 가장 최근 원소만을 갖는 observable
  let observable = source.switchLatest()
  let disposable = observable.subscribe(onNext: { value in
    print(value)
  })

  // one만 포함하는 observable
  source.onNext(one)
  one.onNext("Some text from sequence one")
  two.onNext("Some text from sequence two")
  
  //two만 포함하는 observable
  source.onNext(two)
  two.onNext("More text from sequence two")
  one.onNext("and also from sequence one")
  
  //three만 포함하는 observable
  source.onNext(three)
  two.onNext("Why don't you seem me?")
  one.onNext("I'm alone, help me")
  three.onNext("Hey it's three. I win.")
  
  //one만 포함하는 observable
  source.onNext(one)
  one.onNext("Nope. It's me, one!")

  disposable.dispose()
}
```

```swift
Some text from sequence one
More text from sequence two
Hey it's three. I win.
Nope. It's me, one!
```

# 같은 시퀀스 내의 원소들을 합치기
- reduce: 초기값을 이용해서 accumulator 클로저 의 연산을 거친 최종 결과값 하나만을 원소로 갖는 observable 반환. source가 complete 할때에 비로소 observable이 최종 결과값에 해당하는 원소를 방출

```swift
example(of: "reduce") {
  let source = Observable.of(1, 3, 5, 7, 9)

  // 1
  // 1
  let observable = source.reduce(0, accumulator: { summary, newValue in
    return summary + newValue
  })

  observable.subscribe(onNext: { value in
    print(value)
  })
}
```

```swift
25
```

- scan: reduce는 최종 결과값만 포함하는 반면, scan은 각 단계의 결과값을 모두 포함한다

```swift
example(of: "scan") {
  let source = Observable.of(1, 3, 5, 7, 9)

  let observable = source.scan(0, accumulator: +)
  observable.subscribe(onNext: { value in
    print(value)
  })
}
```

``swift
1
4
9
16
25

```