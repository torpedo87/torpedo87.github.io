# Traits

## Trait란
- 인터페이스간 Observable가 소통하는 것을 도와주고 raw Observable에 비해 문맥상 의미를 전달하거나 특정 용도로 사용하기에 좋다
- 따라서 필수가 아니라 선택적이다
- 하나의 읽기전용 Observable의 wrapper struct 이다

```swift

struct Single<Element> {
    let source: Observable<Element>
}

```

- asObservable()을 호출하면 다시 바닐라 Observable 형태가 된다


## RxSwift의  Traits

### 1. Single

- PrimitiveSequence<SingleTrait, Element>
- 0개 혹은 1개의 원소를 포함하는 Observable sequence
- error or one element
- 연속적인 원소를 방출하는 대신, 하나의 원소나 에러를 방출하도록 보장된 Observable의 변형
- side effect를 공유하지 않는다?
- ex) HTTP Requests (서버로부터 성공한 데이터를 반환하거나 에러를 반환)
- raw Observable에 asSingle()을 호출하면 Single 타입으로 변형가능

### 2. Completable

- error or complete
- 어느 원소도 방출하지 않고 에러 또는 완료만 가능한 Observable의 변형
- side effect를 공유하지 않는다?
- ex) 데이터 저장 성공여부
- 어떤 operation의 성공적인 완료여부만 알고 싶을 때 사용

```swift

func saveMeAtUserDefaults() -> Completable {
    return Completable.create { completable in
      // Store some data locally
      UserDefaults.standard.saveMe(user: Me(id: "id", password: "pwd", tokenId: -1, token: "token"))
      
      guard let _ = UserDefaults.standard.loadMe() else {
        completable(.error(Errors.requestFail))
        return Disposables.create {}
      }
      
      completable(.completed)
      return Disposables.create {}
    }
  }

```

```swift

saveMeAtUserDefaults()
      .subscribe(onCompleted: {
        print("save complete")
      }, onError: { error in
        print("error")
      })
      .disposed(by: bag)

```

### 3. Maybe

- one element or error or complete
- Single or Completable
- side effect를 공유하지 않는다
- rawObservable에 asMaybe()를 호출하면 Maybe 타입으로 변형가능 

```swift

//maybe
  func generateString() -> Maybe<String> {
    return Maybe<String>.create { maybe in
      maybe(.success("RxSwift"))
      
      // OR
      
      maybe(.completed)
      
      // OR
      
      maybe(.error(Errors.requestFail))
      
      return Disposables.create {}
    }
  }

```

```swift

generateString()
      .subscribe(onSuccess: { (string) in
        print("success")
      }, onError: { (error) in
        print("error")
      }, onCompleted: {
        print("complete")
      })
      .disposed(by: bag)

```

---

## RxCocoa

### 1. Driver
- 가장 정교한 trait
- UI layer 관련, 데이터 흐름 관련 reactive code를 짤 때 사용
- 에러가 발생하지 않는다
- main scheduler에서 수행
- side effect를 공유한다?, share(replay: 1, scope: .whileConnected)
- stateful?
- ControlProperty -> asDriver() -> Driver
- bind(to:) -> drive

#### driver로 변환 가능한 observable 조건
- 에러가 발생하지 않는다
- main scheduler에서 이벤트를 관찰
- side effect를 공유한다

```swift

let safeSequence = xs
    .observeOn(MainScheduler.instance)
    .catchErrorJustReturn(onErrorJustReturn)
    .shareReplayLatestWhileConnected()

return Driver(raw: safeSequence)

```

### 2. ControlProperty
- ObservableType, ObserverType 모두 준수 (subject??)
- Sequence of values only represents initial control value and user initiated value changes. Programatic value changes won’t be reported.
- UI 특성을 나타내는 Observable에 대한 Trait
- 실패하지 않는다
- shareReplay(1) behavior, stateful?
- deallocated 될 때에 complete 발생
- 에러가 발생하지 않는다
- MainScheduler에서 이벤트를 전달한다 = subscribeOn(ConcurrentMainScheduler.instance)


### 3. ControlEvent
- ObservableType
- ui 이벤트 관련 observable에 대한 Trait
- 실패하지 않는다
- 구독시 초기값을 전달하지 않는다
- 컨트롤이 해지되면 complete
- 에러가 발생하지 않는다
- main scheduler에서 이벤트 전달 = subscribeOn(ConcurrentMainScheduler.instance)