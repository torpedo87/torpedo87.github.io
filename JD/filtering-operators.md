# Filtering Operators
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

- 연산자는 Rx라는 건물을 짓는 블록과도 같다. 변형, 가공, 이벤트에 반응하기 위해 사용할 수 있다. 더하기 빼기 같은 간단한 연산자를 조합해서 복잡한 수식을 만드는 것 처럼 Rx의 간단한 연산자를 연결해서 복잡한 앱 로직을 구현할 수 있다.
- 필터링 연산자는 .next 이벤트에 적용되어 구독자가 원하는 원소만을 받을수 있다. 스위프트 기본 라이브러리의 filter 메소드를 사용해봤다면 반은 성공이다.

## ignore 연산자

- 마블 다이어그램에서 보듯이 ignoreElements는 .next 이벤트를 차단한다. 하지만 stop 이벤트인 .complted 또는 .error 이벤트는 받는다.

```swift
example(of: "ignoreElements") {

    let strikes = PublishSubject<String>()

    let disposeBag = DisposeBag()

    strikes
        .ignoreElements()
        .subscribe { _ in
            print("you're out")
        }
        .addDisposableTo(disposeBag)
}
```

- 1. strkes 라는 subject 생성
- 2. strkes의 모든 이벤트를 구독하지만 .next 이벤트는 걸러내기
- ignoreElements 는 observable이 끝나는 시점만 알림받고 싶을 때 유용하다. 다음을 추가하자

```swift
strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")
```

- 위의 코드를 호출해도 아무것도 출력되지 않는다. 왜냐하면 .next 이벤트는 무시되기 때문이다. 

```swift
strikes.onCompleted()
```

- 이제 구독자는 .completed 이벤트를 받고 출력을 한다

```swift
you're out
```

- 모든 .next 이벤트를 무시하는 것 말고 n번째 index의 이벤트만 받고 싶을 때가 있다. 이 때 elementAt 연산자를 사용해서 받고 싶은 이벤트만 받고 나머지는 무시할 수 있다. 예를 들어 elementAt(1) 이라면 두번째 이벤트만 받고 나머지는 무시한다

```swift
example(of: "elementAt") {

    let strikes = PublishSubject<String>()

    let disposeBag = DisposeBag()

    strikes
        .elementAt(2)
        .subscribe(onNext: { _ in
            print("you're out")
        })
        .addDisposableTo(disposeBag)
}
```

- 1. subject 생성
- 2. 3번째 next 이벤트만 받고 나머지는 무시

```swift
strikes.onNext("X")
strikes.onNext("X")
strikes.onNext("X")
```

- 세번째 이벤트만 받아서 출력한다

```swift
you're out
```

- filter는 predicate 클로저이다. 각각의 원소에 적용되어 predicate가 true로 되는 원소만을 받는다
- 예를 들어 filter { $0 < 3 } 이라면 1, 2 는 받고 3은 무시한다

```swift
example(of: "filter") {

    let disposeBag = DisposeBag()

    Observable.of(1, 2, 3, 4, 5, 6)
        .filter { integer in
            integer % 2 == 0
        }
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. 미리 정의된 정수를 갖는 observable 생성
- 2. 홀수를 무시하기 위해 필터 연산자 사용한다.
- 3. 이를 구독해서 필터링된 원소만을 출력한다

```swift
2
4
6
```

## skip 연산자
- 첫번째 원소부터 시작해서 원하는 갯수의 원소를 건너뛰고 싶을 때 사용한다

```swift
example(of: "skip") {
    let disposeBag = DisposeBag()

    Observable.of("A", "B", "C", "D", "E", "F")
        .skip(3)
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. observable 만들기
- 2. 처음부터 3개 원소 거르고 구독 시작해서 출력

```swift
D
E
F
```

- skipWhile은 predicate에서 skip이 안되는 것이 나올때 까지 계속 skip하고 만약 skip이 안되는 것이 나와서 방출하면 그 이후로는 skip하지 않는다.

```swift
example(of: "skipWhile") {
    let disposeBag = DisposeBag()

    Observable.of(2, 2, 3, 4, 4)
        .skipWhile { integer in
            integer % 2 == 0
        }
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. 정수의 observable 생성
- 2. 홀수가 방출될때까지 skip

```swift
3
4
4
```

- 지금까지 정적인 필터링을 알아보았다. 그렇다면 동적인 필터링을 하기 위해서는 두가지 연산자가 필요하다. 첫번째로 skipUntil 은 trigger subject가 .next 이벤트를 방출할 때까지 계속 skip한다.

```swift
example(of: "skipUntil") {
    let disposeBag = DisposeBag()

    let subject = PublishSubject<String>()
    let trigger = PublishSubject<String>()

    subject
        .skipUntil(trigger)
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. 우리가 구독할 subject 만들고, 이것을 다루는 방식을 바꿔줄 trigger subject 만들기
- 2. skipUntil을 사용해서 trigger subject가 .next 이벤트를 방출할 때까지 subject의 이벤트를 skip하기

```swift
subject.onNext("A")
subject.onNext("B")
```

- 위에서 subject가 .next 이벤트를 두개 방출하더라고 skip하므로 아무것도 출력하지 않는다

```swift
trigger.onNext("X")
```

- 위에서 trigger가 .next 이벤트를 방출했으므로 이후부터는 subject의 이벤트를 스킵하지 않는다

```swift
subject.onNext("C")
```

```swift
C
```

## take 연산자
- take 연산자는 스킵과 반대이다. 즉 앞에서부터 몇개의 원소를 받을 것인지 결정한다

```swift
example(of: "take") {
    let disposeBag = DisposeBag()

    Observable.of(1, 2, 3, 4, 5, 6)
        .take(3)
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. 정수의 observable 생성
- 2. take 연산자를 사용해서 처음 3개의 원소를 받는다

```swift
1
2
3
```

- takeWhile 은 skipWhile과 비슷하다. 즉 받지 못하는 것이 나올 때 까지 계속 받는다. takeWhileWithIndex 는 predicate에 index값을 사용할 수 있다.

```swift
example(of: "takeWhileWithIndex") {
    let disposeBag = DisposeBag()

    Observable.of(2, 2, 4, 4, 6, 6)
        .takeWhileWithIndex { integer, index in
            integer % 2 == 0 && index < 3
        }
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. 정수의 observable 만들기
- 2. takeWhileWithIndex 연산자를 사용해서 predicate를 만족하지 않는 값이 나올 때까지 계속 받기
- 3. 필터링 된 원소를 구독해서 출력하기

```swift
2
2
4
```

- takeUntil 연산자도 skipUntil 연산자와 비슷하다. trigger 의 .next 이벤트를 받을 때까지 계속 이벤트를 받는다

```swift
example(of: "takeUntil") {
    let disposeBag = DisposeBag()

    let subject = PublishSubject<String>()
    let trigger = PublishSubject<String>()

    subject
        .takeUntil(trigger)
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)

    subject.onNext("1")
    subject.onNext("2")   
}
```

- 1. primary subject, trigger subject 생성
- 2. takeUntil 연산자 사용해서 trigger가 .next 이벤트를 방출할 때까지 계속 원소 받기
- 3. subject에 원소 2개 추가

```swift
1
2
```

- 이제 trgger가 이벤트를 방출하도록 하자

```swift
trigger.onNext("X")
subject.onNext("3")
```

- trigger가 방출한 이후로는 더이상 출력하지 않는다
- takeUntil 연산자는 disposeBag 없이 구독 해제에도 사용될수 있다

```swift
someObservable
    .takeUntil(self.rx.deallocated)
    .subscribe(onNext: {
        print($0)
    })
```

- 이전 코드의 trigger를 self의 해제로 대체했다. 

## distinct 연산자
- distinctUntilChanged 연산자는 연속적인 중복을 막는다

```swift
example(of: "distinctUntilChanged") {
    let disposeBag = DisposeBag()

    Observable.of("A", "A", "B", "B", "A")
        .distinctUntilChanged()
        .subscribe(onNext: {
            print($0)
        })
        .addDidposableTo(disposeBag)
}
```

- 1. observable 생성
- 2. distinctUntilChanged 연산자 사용해서 연속적인 중복을 방지
- distinctUntilChanged는 오직 연속적인 중복만을 방지한다. 그래서 마지막 A는 무시되지 않는다. 

```swift
A
B
A
```

- 위의 경우 문자열이라서 Equatable을 준수한다. 그러나 Equatable 이외에 커스텀 비교로직을 사용해서 중복을 비교할 수도 있다.

```swift
example(of: "distinctUntilChanged(_:)") {
    let disposeBag = DisposeBag()

    let formatter = NumberFormatter()
    formatter.numberStyle = .spellOut

    Observable<NSNumber>.of(10, 110, 20, 200, 210, 310)
        .distinctUntilChanged { a, b in

            guard let aWords = formatter.string(from: a)?
            .components(seperatedBy: " "),
            let bWords = formatter.string(from: b)?
            .components(seperatedBy: " ")
            else {
                return false
            }

            //커스텀 비교로직(같은 것을 포함하는지)
            var containsMatch = false
            for aWord in aWords {
                for bWord in bWords {
                    if aWord == bWord {
                        containsMatch = true
                        break
                    }
                }
            }

            return containsMatch
        }
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. 숫자를 하나씩 추출하는 formatter 생성
- 2. observable 생성
- 3. distinctUntilChanged 연산자를 사용해서 연속된 한 쌍의 원소를 받는다
- 4. 빈칸을 기준으로 원소를 분리하기
- 5. 쌍의 원소의 숫자를 돌면서 같은 숫자를 포함하는지 체크
- 6. 구독하고 출력하기

```swift
10
20
200
```

- 이외에도 Equatable을 준수하지 않는 타입의 중복을 방지하는 데에도 유용하다