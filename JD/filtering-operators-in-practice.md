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

## 모든 이벤트 무시하기 (ignoreElements)
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

- 다음의 구독은 모든 이미지를 무시하고 사용자가 메인뷰컨트롤러에 돌아왔을 때 onComplted 클로저를 실행한다. Xcode 에러를 피하기 위해 메인뷰컨트롤러에 다음 메소드를 추가하자

```swift

private func updateNavigationIcon() {
    let icon = imagePreview.image?
        .scaled(CGSize(width: 22, height: 22))
        .withRenderingModel(.alwaysOriginal)

    navigationItem.leftBarButtonItem = UIBarButtonItem(image: icon, style: .done, target: nil, action: nil)
}
```

- 앱을 실행해서 새로운 콜라주를 만들자. 사진을 추가하고 돌아올 때마다 새로운 구독이 프리뷰를 업데이트한다.

## 필요 없는 원소 필터링하기 (filter)
- 특정 이벤트만 무시하고 싶다면 어떻게 할까? filter 연산자를 사용하자. 예를 들어 콜라주의 틀이 가로방향인경우 세로사진은 무시하고 가로사진만 받고 싶다.
- actionAdd() 의 위로 스크롤해서 newPhotos의 첫구독의 첫 연산자로 filter를 추가하자

```swift
newPhotos
    .filter { newImage in
        return newImage.size.width > newImage.size.height
    }
    .........
```

- newPhotos가 방출하는 각각의 사진은 구독하기 전에 다음과 같은 filter 테스트를 거친다. filter 연산자는 이미지 가로가 세로보다 큰지 체크해서 크면 통과 시킨다. 세로 사진은 무시된다
- 앱을 실행해서 사진을 추가하자. 세로사진은 콜라주에 추가되지 않을 것이다

## 중복 사진 필터링 (filter)
- 현재 이 앱은 같은 사진을 중복해서 추가할 수 있는 상태이다. 중복을 제거하자
- 물론 중복을 제거하는 더 좋은 방법이 있지만 현재 배우고 있는 RxSwift 기술로 연습해보자
- 방출된 원소가 유니크한지 확인하기 위해서는 observable의 과거 방출내역을 살펴봐야한다
- 방출된 이미지의 인덱스를 기억해놓는 것은 도움이 되지 않는다 왜냐하면 같은 이미지를 나타내는 두개의 이미지는 인덱스가 달라서 다른 것으로 취급되기 때문이다. 우리는 byte length를 사용해보자. 메인뷰컨트롤러에 다음 프로퍼티를 추가하자

```swift
private var imageCache = [Int]()
```

- 위의 배열에 각 이지미의 바이트 길이를 저장할 것이다. 아까 추가한 filter 연산자 밑에 다음을 추가하자

```swift
newPhotos
    //가로 사진만 받기
    .filter { newImage in
        return newImage.size.width > newImage.size.height
    }
    //PNG 데이터의 바이트 길이를 받아서 배열에 있나 확인해서 있으면 중복으로 간주
    .filter { [weak self] newImage in
        let len = UIImagePNGRepresentation(newImage)?.count ?? 0
        guard self?.imageCache.contains(len) == false else {
            return false
        }

        self?.imageCache.append(len)

        return true
    }
```

- PNG 데이터의 바이트 길이를 받아서 이미지캐시 배열에 이미 존재하면 중복으로 간주한다
- actionClear()에 다음을 추가하자

```swift
imageCache = []
```

- 앱을 실행하고 같은 사진을 넣으려고 해보면 한장만 추가될 것이다

## 조건이 만족하는 동안 계속 받기 (takeWhile)
- 이 앱에는 현재 버그가 있다. 6장을 추가하면 더이상 + 버튼이 활성화되지 않는다. 하지만 포토뷰컨트롤러에서는 6개를 넘는 사진을 선택할 수 있다. takeWhile 연산자를 사용해보자. 
- actionAdd() 윗부분에 첫 구독을 찾아서 다음을 추가하자

```swift
newPhotos
    .takeWhile { [weak self] image in
        return (self?.images.value.count ?? 0) < 6
    }
```

- takeWhile 연산자는 콜라주를 구성하는 이미지의 총 갯수가 6 미만이면 계속 받는다. 만약에 nil 이면 0 으로 간주한다. 이것은 컴파일러를 만족하고 self의 !를 피하기 위함이다.
- 앱을 실행하고 콜라주에 많은 사진을 넣어보자. 6장을 만족하면 더이상 추가할 수 없다

## 사진 선택 개선하기
- 포토뷰컨트롤러를 열고 새로운 Observable을 만들고 filter 연산자를 적용하자

### 1. PHPhotoLibrary authorization observable
- 앱을 처음 실행하면 포토라이브러리 접근을 허용해야 한다. 과연 사용자가 잘 할 수 있을까.
앱이 처음 포토라이브러리 접근을 시도할 때 OS는 비동기적으로 사용자의 허가를 요청한다. 딱 한번만 요청한다. 그러므로 이번 장에서는 앱의 첫 실행을 구현하기 위해 시뮬레이터의 내용을 리셋하자.
- 시뮬레이터를 열고 메인메뉴에서 Reset Content and Setting을 열고 Reset을 클릭한다. 초기화할 것이다. 다시 앱을 실행해서 + 버튼을 눌러보자. 접근 경고창이 뜰 것이다. ok를 클릭하면 사진들이 나타나지 않을 것이다. 메인뷰로 돌아갔다가 다시 +버튼을 누르면 그제서야 사진이 보인다...
- 포토뷰컨트롤러에서 photos 프로퍼티에서 모든 사진을 로드한다. 한번 접근이 허용되기만 하면 사진을 reload할 방법이 없다.
- PHPhotoLibrary+rx.swift 파일을 생성하고 다음을 추가하자

```swift
import Foundation
import Photos
import RxSwift

extension PHPhotoLibrary {
    static var authorized: Observable<Bool> {
        return Observable.create { observer in
            return Disposables.create()
        }
    }
}
```

- 새로운 Observable 인 authorized 프로퍼티를 추가한다.
- 이 observable은 사용자가 접근을 허용하는지 아닌지에 따라 2 가지 경로가 있다. create 클로저안에 return Disposable.create() 바로 위에 다음 코드를 추가하자

```swift
DispatchQueue.main.async {
    if authoriztionStatus() == .authorized {
        observer.onNext(true)
        observer.onComplted()
    } else {
        observer.onNext(false)
        requestAuthorization { newStatus in
            observer.onNext(newStatus == .authorized)
            observer.onComplted()
        }
    }
}
```

- 사용자가 이전에 접근을 허용했다면 true를 방출한다. 그렇지 않으면 사용자에게 허가를 요청하고 결과를 방출한다.
- 비동기로 호출하는 이유는 observable이 현재 스레드를 막아서는 안되기 때문이다.

## 2. 접근 허용시 사진컬렉션 reload 하기
- 접근이 허용되는 경우는 두가지다
- 처음 앱 실행해서 ok 하는경우(-false-true-.completed)와, 이전에 허용된 경우(-true-.completed)
- 포토뷰컨트롤러를 열고 viewDidLoad()에 다음을 추가하자

```swift
let authorized = PHPhotoLibrary.authorized.share()
```

- authorized라는 새로운 공유 observable을 생성했다. 이 Observable을 구독하는 2개의 구독을 생성할 것이다
- viewDidLoad()에 다음을 추가하자

```swift
authorized
    .skipWhile { $0 == false }
    .take(1)
    .subscribe(onNext: { [weak self] _ in
        self?.photos = PhotosViewController.loadPhotos()

        //UI 업데이트는 메인 스레드에서 동작해야 한다
        DispatchQueue.main.async {
            self?.collectionView?.reloadData()
        }
    })
    .addDisposableTo(bag)
```

- skipWhile 연산자를 사용해서 true가 나올때까지 false 이벤트를 무시한다. 사용자가 접근을 허용하지 않으면 구독은 시작하지 않는다
- take(1) 연산자를 사용하여 true가 나오면 처음 1개의 원소(true)를 받고 나머지는 무시한다
- true를 받으면 collectionView를 reload한다
- 구독 클로저에서 컬렉션뷰를 reload 하기전에 메인스레드로 이동한다. 왜냐하면 requestAuthoriztion 메소드는 completion closure가 어느 스레드에서 동작할지 보장하지 않는다. background 스레드에서 동작할 수도 있다.  next 이벤트를 호출하면 모든 구독에게 알린다. 그러면 구독에서 self?.collectionView?.reloadData()를 호출하는데 만약 background 스레드에 있다면 UIKit가 충돌날 것이다. UI 업데이트는 반드시 메인스레드에서 동작해야하기 때문이다.
- RxSwift에서는 GCD 를 사용하는 대신에 스케줄러를 사용할 수 있다. 15장에서 살펴볼 것이다

### 3. 접근 허가 없으면 에러제시지 띄우기
- 사용자가 접근을 거부하는 경우의 수는 두가지다
- 처음 앱 실행시 no 클릭(-false-false-.completed) 또는 이전에 거부한 경우(-false-false-.completed)
- viewDidLoad()에 다음을 추가하자

```swift
authorized
    .skip(1)
    .takeLast(1)
    .filter { $0 == false }
    .subscribe(onNext: { [weak self] _ in
        guard let errorMessage = self?.errorMessage else { return }
        DispatchQueue.main.async(execute: errorMessage)
    })
    .addDisposableTo(bag)
```

- 1. 처음이벤트는 마지막이 아니므로 1개 건너뛰기
- 2. 마지막 이벤트 받아서 false인지 확인해서 맞으면 에러메시지 띄우기
- 포토뷰컨트롤러에 다음 메소드를 추가하자

```swift
private func errorMessage() {
    //경고창
    alert(title: "no access to camera roll", text: "u can grant access from settings")

    //close 탭하면 구독 해지되므로 클로저 호출
    .aubscribe(onDisposed: { [weak self] in
        self?.dismiss(animated: true, completion: nil)
        _ = self?.navigationController?.popViewController(animated: true)
    })
    .addDisposableTo(bag)
}
```

- alert()를 구현하면 사용자가 alert 버튼을 탭하기만 하면 observable이 완료될 것이다. observable 을 해지하고 경고를 숨길 것이다. 그래서 구독의 onDisposed 클로저를 자극하여 포토뷰컨트롤러가 사라질 것이다.

## 시간에 기초한 필터 연산자 만들어보기

### 1. 특정 시간 이후 구독 완료하기
- 사용자가 포로라이브러리 접근을 거부했다면 경고창이 뜨고 close 버튼을 누르면 돌아간다
- 위 경고창을 사용자가 5초 이내에 close 버튼을 누르지 않으면 자동으로 사라지도록 바꾸자
- 포토뷰컨트롤러를 열고 errorMessage() 메소드를 찾아서 alert(title...) 코드 바로 아래에 다음을 넣자

```swift
.take(5.0, scheduler: MainScheduler.instance)
.............
```

- take(_:scheduler:) 연산자는 주어진 시간동안만 원소를 받고 시간이 지나면 구독을 완료한다
- 사용자가 close 버튼을 누르면 5초를 기다리지 않고 즉시 구독을 완료한다. 

### 2. throttle 연산자를 사용해서 연속클릭 필터링

- 메인뷰컨트롤러의 viewDidLoad()를 살펴보자

```swift
images.asObservable()
    .subscribe(onNext: { [unowned self] photos in
        self.imagePreview.image = UIImage.collage(images: photos, size: self.imagePreview.frame.size)
    })
    .addDisposableTo(bag)
```

- 사용자가 사진을 선택할때마다 구독이 새로운 사진을 받아서 콜라주를 만든다. 새로운 사진을 받을때마다 이전 것은 쓸모없어진다. 사용자가 연속으로 빨리 복수의 사진을 탭하면 그럼에도 불구하고 구독은 각각 새로운 콜라주를 생성한다. 따라서 중간에 생산되는 콜라주는 다 쓸모없는 결과물이다. 그렇다면 곧 새로운 이벤트를 받을 것이라는 걸 어떻게 알 수 있을까?
- 이벤트를 계속 방출하지만 제일 마지막 이벤트만 받고 싶다면 throttle(_:scheduler:) 연산자를 사용하자. viewDidLoad의 첫구독의 images.asObservable() 바로 다음에 추가하자

```swift
.throttle(0.5, scheduler: MainScheduler.instance)
................
```

- throttle 연산자는 특정 시간동안 뒤따르는 이벤트를 필터링한다. 만약 사용자가 사진을 선택하고 0.2초 후에 또 다른 사진을 탭하면 throttle은 첫번째 선택된 사진은 무시하고 두번째 사진만 받는다. 사용자가 사진을 5개 선택해서 차례대로 빨리 탭하면 0.5초를 넘어가지 않는 한 throttle은 앞의 4장의 사진은 무시하고 5번째 사진만을 받는다.
- throttle 연산자를 사용하는 경우
- 1. 검색창 구독이 있고 현재 텍스트를 서버 API에 보낸다. 이 때 사용자가 글자를 빠르게 타이핑하면 타이핑이 끝났을 때의 텍스트만을 서버에 보낸다
- 2. 사용자가 bar 버튼을 탭하면 모달뷰를 띄운다. 더블클릭을 방지해서 모달뷰가 두번 나오는 것을 방지한다.
- 3. 사용자가 드래그할 때 멈추는 지점까지의 영역에 관심이 있는 것이다. 따라서 사용자의 드래그 위치가 멈출 때까지 이벤트를 무시할 수 있다

- throttle은 입력값이 과도할 때 유용하게 쓸수 있는 연산자이다.