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

- toArray 연산자는 원소들을 담은 하나의 배열을 포함하는 observable 로 변형해 반환
- 1. observable 생성
- 2. toArray 연산자 사용해서 배열 형태로 변형

```swift
["A", "B", "C"]
```

- map 연산자도 살펴보자
- O<(Int, [Cat])> -> map(rx) -> O<[Cat]>
- [Cat] -> map(swift) -> [Cat]
- swift의 map 연산자는 컬렉션 타입일 때에만 사용가능하나 rx에서는 observable 자체가 시퀀스이므로 컬렉션이 아니더라도 적용 가능
- observable 시퀀스의 원소를 변형

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

- 각 원소를 임의의 형태로 변형후 이벤트 타입으로 싸서 반환
- 1. 숫자를 문자로 읽어주는 formatter 생성
- 2. observable 생성
- 3. map 을 사용

```swift
"one hundred twenty three"
"four"
"fifty six"
```

- enumerated 연산자: 인덱스, 원소값를 포함하는 튜플형태를 갖는 observable을 반환
-  Observable<(index: Int, element: Int)>

```swift
example(of: "enumerated and map") {
  
  let disposeBag = DisposeBag()
  
  // 1
  Observable.of(1, 2, 3, 4, 5, 6)
    // 2
    .enumerated()
    // 3
    .map { index, integer in
      index > 2 ? integer * 2 : integer
    }
    // 4
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
}
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

- mapWithIndex = enumerated + map
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
 - 클로저는 Observable을 반환
 - flatmap은 Observalbe< O.E > 를 반환하는데 여기서 O.E는 클로저에서 반환한 Observable의 Element를 나타낸다
- map 연산자는 단순히 observable 시퀀스의 원소를 변형만 했지만, flat map은 각 원소를 observable 타입으로 감시해서 이후에 각 원소가 변할 때마다 그 변화를 감지할 수 있다.

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

- flatmap의 클로저는 변형 역할
- flapMap은 map과 유사하지만 nil일 경우 skip을 한다. unwrapping.
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
- 앞의 observable이 complete할 때까지 기다렸다가 다음 observable을 전달
- 요약해보면 flatMap 연산자는 각각의 모든 observable의 변화를 계속 관찰한다. 그런데 우리는 가장 최신의 observable의 변화만을 관찰하고 싶을 지도 모른다. 이럴 때는 flatMapLatest 연산자를 사용하자. 이것은 map 연산자와 switchLatest를 합친 연산자이다. switchLatest 연산자는 나중에 배울 것이지만 이것은 가장 최신의 observable의 값만 추적하고 이전의 observable은 구독하지 않는다.

```swift
example(of: "flatMapLatest") {
    let disposeBag = DisposeBag()
  
  let ryan = Student(score: BehaviorSubject(value: 80))
  let charlotte = Student(score: BehaviorSubject(value: 90))
  
  let student = PublishSubject<Student>()
  
  student
    .flatMapLatest {
      $0.score
    }
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
  
  //student 옵저버에게 라이언을 전달
  student.onNext(ryan)
  
  //score는 subject이므로 옵저버로서 역할 가능
  ryan.score.onNext(85)
  
  //student 옵저버에게 샬롯을 전달하면서 이전에 전달받은 라이언 무시
  student.onNext(charlotte)
  
  // 라이언이 무시되므로 옵저버가 원소를 전달해도 observable이 받지 못함
  ryan.score.onNext(95)
  
  charlotte.score.onNext(100)
}
```

```swift
80
85
90
100
```

- flatMapLatest 연산자는 네트워킹 작업에 주로 사용된다. 사용자가 검색창에 s, w, i, f, t 를 한 글자씩 입력할 때마다 이전의 결과를 무시하고 새로운 검색을 수행하기를 원할 것이다.
- 가장 최근의 구독만 활성화


- materialize and dematerialize
```swift
example(of: "materialize and dematerialize") {
  
  // 1
  enum MyError: Error {
    
    case anError
  }
  
  let disposeBag = DisposeBag()
  
  // 2
  let ryan = Student(score: BehaviorSubject(value: 80))
  let charlotte = Student(score: BehaviorSubject(value: 100))
  
  let student = BehaviorSubject(value: ryan)
  
  // 1
  let studentScore = student
    .flatMapLatest {
      $0.score.materialize()
    }
  
  // 2
  studentScore
    // materialize 된 observable은 에러 이벤트를 방출하지 않고 우리가 원소의 error에 접근할 수 있어서 에러를 필터링 가능하게 한다
    .filter {
      guard $0.error == nil else {
        print($0.error!)
        return false
      }
      
      return true
    }
    // 원상복구
    .dematerialize()
    .subscribe(onNext: {
      print($0)
    })
    .disposed(by: disposeBag)
  
  ryan.score.onNext(85)
  
  //옵저버가 에러 이벤트를 전달해도 이것을 방출하지 않고 출력하고 필터링된다
  ryan.score.onError(MyError.anError)
  
  ryan.score.onNext(90)
  
  // 4
  student.onNext(charlotte)
}
```

```swift
80
85
anError
100
```

- materialize: 각 원소를 Event 타입으로 변형한 것을 포함하는 observable을 반환. 반환된 observable은 절대 에러를 방출하지 않는다
- dematerialize: 원상복구