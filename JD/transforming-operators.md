# transforming operators
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

## 원소 변형하기
- observable은 각각 원소를 방출하지만 컬렉션으로 작업하고 싶을 때가 있다. observable의 각각의 원소를 배열로 변형하는 쉬운 방법은 toArray 연산자를 사용하는 것이다.

```swift
example(of: "toArray") {
    let disposeBag = DisposeBag()

    Observable.of("A", "B", "C")
        .toArray()
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. observable 생성
- 2. toArray 연산자 사용해서 배열 형태로 변형

```swift
["A", "B", "C"]
```

- map 연산자도 살펴보자

```swift
example(of: "map") {

    let disposeBag = DisposeBag()

    let formatter = NumberFormatter()
    formatter.numberStyle = .spellOut

    Observable<NSNumber>.of(123, 4, 56)
        .map {
            formatter.string(from: $0) ?? ""
        }
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. 숫자를 문자로 읽어주는 formatter 생성
- 2. observable 생성
- 3. map 을 사용

```swift
"one hundred twenty three"
"four"
"fifty six"
```

- mapWithIndex 연산자를 알아보자

```swift
example(of: "mapWithIndex") {

    let disposeBag = DisposeBag()

    Observable.of(1, 2, 3, 4, 5, 6)
        .mapWithIndex { integer, index in
            index > 2 ? integer * 2 : integer
        }
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. observable 생성
- 2. mapWithIndex 사용해서 index가 2를 초과하면 2를 곱하고 아니면 그냥 반환

```swift
1
2
3
8
10
12
```

## 내부 observable 변형하기

```swift
struct Student {
    var score: Variable<Int>
}
```

- flat map 연산자를 알아보자

```swift
example(of: "flatMap") {
    let disposeBag = DisposeBag()

    let ryan = Student(score: Variable(80))
    let charlotte = Student(score: Variable(90))

    let student = PublishSubject<Student>()

    student.asObservable()
        .flatMap {
            $0.score.asObservable()
        }
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)
}
```

- 1. 라이언, 샬롯의 두 학생 인스턴스 생성
- 2. 학생 타입의 source subject 생성
- 3. flatMap 연산자를 사용해서 subject의 score에 접근한다. score는 Variable이므로 이에 대해 asObservable()을 호출한다. 여기서는 score를 변형시키지 않고 그냥 통과시킨다
- 4. .next 이벤트 원소를 출력한다
- 아직 아무것도 출력되지 않는다. 이제 라이언의 성적을 출력해보자

```swift
student.onNext(ryan)
```

```swift
80
```

- 라이언의 성적을 바꿔보자

```swift
ryan.score.value = 85
```

```swift
85
```

- 샬롯의 점수를 출려해보자

```swift
student.onNext(charlotte)
```

```swift
90
```

- 라이언의 점수를 또 바꿔보자

```swift
ryan.score.value = 95
```

```swift
95
```

- flatMap 연산자는 연산자가 생성한 observable 즉 score를 계속 관찰한다. 이제 샬롯의 점수를 바꿔보면 마찬가지로 바뀐 점수가 출력된다
- 요약해보면 flatMap 연산자는 각각의 모든 observable의 변화를 계속 관찰한다. 그런데 우리는 가장 최신의 observable의 변화만을 관찰하고 싶을 지도 모른다. 이럴 때는 flatMapLatest 연산자를 사용하자. 이것은 map 연산자와 switchLatest를 합친 연산자이다. switchLatest 연산자는 나중에 배울 것이지만 이것은 가장 최신의 observable의 값만 추적하고 이전의 observable은 구독하지 않는다.

```swift
example(of: "flatMapLatest") {
    let disposeBag = DisposeBag()

    let ryan = Student(score: Variable(80))
    let charlotte = Student(score: Variable(90))

    let student = PublishSubject<Student>()

    student.asObservable()
        .flatMapLatest {
            $0.score.asObservable()
        }
        .subscribe(onNext: {
            print($0)
        })
        .addDisposableTo(disposeBag)

    student.onNext(ryan)

    ryan.score.value = 85

    student.onNext(charlotte)

    ryan.score.value = 95

    charlotte.score.value = 100
}
```

```swift
80
85
90
100
```

- flatMapLatest 연산자는 네트워킹 작업에 주로 사용된다. 사용자가 검색창에 s, w, i, f, t 를 한 글자씩 입력할 때마다 이전의 결과를 무시하고 새로운 검색을 수행하기를 원할 것이다.

