# Filtering Operators in Practice
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

- 연산자는 Observable<E> 클래스의 간단한 메소드이고 어떤 것은 ObservableType 프로토콜에 정의된다. 

## 구독 공유하기
- subscribe 를 호출하면 Observable이 일어나서 원소를 생산하기 시작한다. 즉 subscribe롤 호출할 때마다 observable은 create 클로저를 호출한다.

```swift
let numbers = Observable<Int>.create { observer in
    let start = getStartNumber()
    observer.onNext(start)
    observer.onNext(start+1)
    observer.onNext(start+2)
    observer.conCompleted()
    return Disposables.create()
}
```

- 위의 Observable<Int> 는 start, start+1, start+2 를 방출한다
- getStartNumber()를 살펴보자

```swift
var start = 0

func getStartNumber() -> Int {
    start += 1
    return start
}
```

- 이 메소드는 start 변수를 1 증가시켜서 반환한다. 이제 구독을 시작해보자

```swift
numbers
    .subscribe(onNext: { el in
    print("element [\(el)]")
    }, onCompleted: {
        print("-------")
    })
```

- 다음과 같이 출력될 것이다

```swift
element [1]
element [2]
element [3]
-----------
```

- 위의 구독 코드를 복사해서 한번 더 구독해보자 이번에는 출력값이 다르다

```swift
element [2]
element [3]
element [4]
```

- subscribe를 호출할 때마다 create 클로저를 호출하므로 새로운 observable을 만든다. 따라서 이전 구독과 같은 구독이라고 보장할 수 없다. 구독을 공유하기 위해서 share() operator를 사용할 수 있다. 메인뷰컨트롤러를 열고 actionAdd() 의 photosViewController.selectedPhotos 부분을 다음으로 수정하자

```swift
let newPhotos = photosViewController.selectedPhotos.share()

newPhotos.........
```

- share() 연산자를 사용해서 이제 우리는 각 구독마다 새로운 observable을 생성하는 대신, 복수의 구독이 같은 observalbe을 소비하도록 할 수 있다.
- 이제 newPhotos 싱글톤에 대한 두번째 구독을 생성해서 우리가 필요하지 않은 원소를 필터링하자. 그 전에 share, shareReplay, shareReplayLatestWhileConnected를 분별하자
- share는 공유 구독자수가 없을 때에만 구독을 생성을 하고 만약 공유 구독자수가 있으면 기존의 구독을 사용한다. 만약 모든 공유 구독자들이 해지되면 share는 공유된 observable 또한 해자할 것이다.
- share는 새로운 구독자들에게 이전에 방출된 정보를 제공하지 않지만 shareReplay는 최신의 값을 새로운 구독자에게 제공한다
- 공유 연산자를 사용하는 가장 큰 이유는 완료되지 않는 observable이나 완료 후 새로운 구독이 더이상 안된다는 것을 보장하고 싶을 때 사용하기 안전하기 때문이다.

## 모든 이벤트 무시하기
- 모든 원소를 무시하는 필터링 연산자부터 시작하자. 값이나 타입에 상관없이 ignoreElements() 연산자는 모든 원소를 통과시키지 않는다
- 사용자가 사진을 선택할 때마다 newPhotos 는 UIImage 를 방출한다. 우리는 스크린 좌측상단에 콜라주 미리보기 아이콘을 추가할 것이다. 
- 아이콘을 한번만 업데이트 하고싶기 때문에 사용자가 메인뷰컨트롤러로 돌아가면 모든 UIImage 이벤트는 무시하고 .completed 이벤트에만 관심 가져야 한다
- ignoreElements() 는 모든 .next 이벤트를 무시하고 오직 .complted 또는 .error만 받는다.
- actionAdd() 에서 추가한 코드 바로 아래에 다음을 추가하자

```swift
newPhotos
    .ignoreElements()
    .subscribe(onCompleted: { [weak self] in
        self?.updateNavigationIcon()
    })
    .addDisposableTo(photosViewController.bag)
```

- 