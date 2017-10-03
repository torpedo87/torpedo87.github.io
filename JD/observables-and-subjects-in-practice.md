# observables and subjects in practice
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

- UI를 데이터모델과 결합하거나 새 컨트롤러를 화면에 나타내고 결과를 받는 등의 실제 개발에 observable을 적용시키는 건 약간 어려울 것이다. 아직 적용하는 방법이 익숙지 않더라도 괜찮다. 2장, 3장에서는 이론적인 연습을 했다. 이번에는 완전한 앱을 갖고 연습해보자. 스타터 프로젝트는 non-Rx 코드를 포함한다. 이제 할 일은 RxSwift 프레임워크를 추가하고 반응형 기술을 추가하는 것이다. 이번 장에서는 RxSwift와 observable을 사용해서 사용자가 멋진 사진을 만들 수 있도록 해보자.

## 시작하기
- 스타토 프로젝트인 Combinestagram 폴더를 열자. 팟파일을 설치하고 Combinestagram.xcworkspace 파일을 열자. 메인 스토리보드를 열고 다음과 같은 인터페이스를 볼 수 있을 것이다
- 첫 화면에서 사용자는 현재 사진 콜라주를 볼 수 있고 현재 리스트의 사진을 지우거나 저장할 수 있는 버튼이 있다.
- 사용자가 +버튼을 탭하면 두번째 뷰 컨트롤러로 이동하고 여기서는 카메라롤의 사진목록을 확인할 수 있다. 사용자는 썸네일을 클릭해서 사진을 콜라주에 추가할 수 있다.
- 뷰컨트롤러와 스토리보드는 이미 연결되어 있다. 콜라쥬가 어떻게 합쳐지는지는 UIImage+Collage.swift 파일을 보면 알 수 있다.
- 이번 장에서는 배운 기술을 적용시켜 볼 것이다. 시작하자.

## 뷰 컨트롤러에서 variable 사용하기
- 컨트롤러 클래스에 Variable<UIImage> 프로퍼티를 추가하고 value에 선택된 사진을 넣자. 3장에서 배웠듯이 Variable 클래스는 우리는 기존의 variable을 다루는 데 익숙해졌을 것이다. 원하는 대로 value 프로퍼티 값을 수동으로 바꿀 수 있다. 간단한 예제로 시작해서 나중에는 커스텀 observable까지 다뤄보자.
- 메인뷰 컨트롤러 파일을 열고 다음을 추가하자

```swift
private let bag = DisposeBag()
private let images = Variable<[UIImage]>([])
```

- 다른 클래스는 위의 두 상수를 사용하지 않기 때문에 private 으로 접근제어 했다.
- dispose bag은 뷰컨트롤러 안에 존재하기 때문에 뷰 컨트롤러가 해제되면 뷰 컨트롤러 내의 모든 observable이 취소될 것이다. 이를 통해 Rx 구독 메모리 관리를 쉽게 해준다. 가방에 담기만 하면 뷰 컨트롤러가 소멸할 때 같이 취소된다.
- 그러나 앱이 종료되기 전에는 루트뷰 컨트롤러와 메인뷰 컨트롤러가 소멸되지 않으므로 위의 경우는 발생하지 않을 것이다. 스토리보드의 다른 뷰컨트롤러에서 이를 해결하기 위한 메커니즘을 살펴볼 것이다.
- 우선 이 앱은 항상 같은 사진에 기반한 콜라주를 만들 것이다. 걱정 마라. 이 사진은 멋진 바르셀로나 풍경 사진이며 Assets.xcassets에 이미 담겨 있다. 사용자가 + 버튼을 탭할 때마다 같은 사진을 추가할 것이다.
- actionAdd() 메소드 안에 다음을 추가하자

```swift
images.value.append(UIImage(named: "IMG_1907.jpg")!)
```

- 기존 variable을 다루던 대로 current value 에 이미지를 추가했다. Variable 클래스는 자동으로 value 프로퍼티의 observable 만든다. 초기값은 빈 배열이며 사용자가 + 버튼을 탭할 때마다 이미지에 의해 생성된 observable이 새로운 배열을 포함하는 .next 이벤트를 방출한다. 사용자가 현재 선택한 사진을 지우도록 하기 위해 actionClear() 메소드에 다음을 추가하자

```swift
images.value = []
```

- 위의 코드 몇줄 삽입해서 사용자 입력을 간단히 다룰 수 있게 되었다. 이제 이미지를 관찰하고 스크린에 결과값을 뿌려보자

## 콜라주에 사진 추가하기
- 이제 이미지 변화를 관찰해서 콜라주 프리뷰를 업데이트할 수 있게 되었다. viewDidLoad() 에서 다음과 같이 이미지 구독을 추가하자. Variable 이므로 구독하기 위해서 asObservable()을 사용해야 한다.

```swift
images.asObservable()
    .subscribe(onNext: { [weak self] photos in
        guard let preview = self?.imagePreview else { return }
        preview.image = UIImage.collage(image: photos, size: preivew.frame.size)
    })
    .addDisposableTo(bag)
```

- 이미지가 방출한 .next 이벤트를 구독하기 시작했다. 각각의 이벤트에 대해 UIImage.collage(image:size:) 메소드를 사용해서 콜라주를 생성한다. 마지막으로 이 구독을 dispose bag에 넣어둔다.
- 이번 장에서는 viewDidLoad() 에서 구독을 시작한다. 이 책의 후반부에는 각각의 클래스에서 이들을 추출하는 것과 MVVM 패턴에 대해 학습할 것이다.
- 이제 합쳐진 콜라주를 얻었다. 사용자가 + 버튼을 눌러서 이미지를 업데이트하고 차례대로 UI를 업데이트 한다.
- 앱을 실행해서 테스트해보자. 사진을 4번 추가하면 콜라주가 다음과 같이 보일 것이다.
- 물론 이 앱은 좀 지루할 것이지만 걱정 마라. 카메라롤에서 사진을 선택하는 기능을 추가해보자.

## 복잡한 뷰컨트롤러 UI 다루기
- 사용자 경험을 향상시키기 위해 앱의 UI가 더 수정될 필요가 있다. 예를 들어
- 1. 선택된 사진이 없는 상태 또는 방금 지운 경우 clear 버튼 비활성화
- 2. 선택된 사진이 없는 상태의 경우 save 버튼 비활성화
- 3. 선택된 사진 갯수가 홀수인 경우 save 버튼 비활성화
- 4. 하나의 콜라주에 최대 사진을 6장으로 제한
- 5. 뷰 컨트롤러 제목이 현재의 선택을 반영

- 위 리스트들을 구현할 때 non-reactive 방식으로 구현하려면 꽤 번거로울 것이다.
- 다행히도 RxSwift를 사용해서 이미지를 구독하여 UI를 업데이트 하는 작업을 할 수 있다
- viewDidLoad() 에 다음 구독을 추가하자

```swift
images.asObservable()
    .subscribe(onNext: { [weak self] photos in
        self?.updateUI(photos: photos)
    })
    .addDisposableTo(bag)
```

- 사진을 선택할 때마다 updateUI(photos:)를 호출한다. 아직 우린 이 메소드를 구현하지 않았으므로 클래스 안에 추가하자

```swift
private func updateUI(photos: [UIImage]) {
    buttomSave.isEnabled = photos.count > 0 && photos.count % 2 == 0
    buttonClear.isEnabled = photos.count > 0
    itemAdd.isEnabled = photos.count < 6
    title = photos.count > 0 ? "\(photos.count) photos" : "Collage"
}
```

- 위에서 지정한 규칙에 따라서 UI를 업데이트 하는 메소드이다. 앱을 다시 실행하면 모든 규칙대로 잘 실행될 것이다.
- 앱에 Rx를 적용해서 실제 효과를 보기 시작했다. 이번 장에서 추가한 코드를 살펴보면 단 몇줄로 UI를 업데이트 할 수 있게 된 것을 알 수 있다.

## subject를 통해 뷰 컨트롤러간 소통하기
- 이번에는 사용자가 카메라롤에서 사진을 선택할 수 있게 하기 위해 PhotosViewController와 메인 뷰컨트롤러를 연결해보자.
- 우선 네비게이션 스택에 PhotosViewController를 넣는다. 메인뷰 컨트롤러를 열고 actionAdd()를 찾아서 항상 사용하는 이미지인 IMG_1907.jpg를 주석처리하고 다음을 추가하자

```swift
let photosViewController = storyboard!.instantiateViewController(withIdentifier: "PhotosViewController") as! PhotosViewController

navigationController!.pushViewController(photosViewController, animated: true)
```

- 위에서 PhotosViewController 인스턴스를 생성하고 네비게이션 스택에 푸시한다. 앱을 실행해서 + 버튼을 탭하면 카메라롤 접근을 묻는 알림창을 볼 수 있다.
- 한번 승낙하고나면 이제부터는 카메라롤을 볼 수 있다. 시뮬레이터로 실행하면 실제로 사진들이 당신의 기기에 있는 사진과 다를 것이다. 안되면 다시 뒤로갔다가 들어와보자.
- 기존의 Cocoa 패턴을 사용해서 앱을 만들면 다음으로, photos controller가 메인컨트롤러와 소통하기 위해 delegate 프로토콜을 추가해야 할 것이다.
- 이와 달리 RxSwift를 사용한다면, 어느 두 클래스간 의사소통하는 일반적인 방법은 Observable이다. Observable은 어느 종류의 메시지도 전달이 가능하기 때문에 특별한 프로토콜을 정의할 필요가 없다.

## 선택된 사진에 observable 생성하기
- 사용자가 카메라롤의 사진을 탭할때마다 .next 이벤트를 방출하는 subject를 PhotosViewController에 추가할 것이다.
- PhotosViewController.swift 파일을 열고 윗부분에 다음을 추가하자

```swift
import RxSwift
```

- 사진이 선택되는 것이 접근가능하도록 PublishSubject을 추가하고 싶을 것이다. 하지만 반드시 subject를 접근가능하도록 만들 필요는 없다. 왜냐하면 다른 클래스가 onNext(_)를 호출해서 subject가 값을 방출하도록 할 수도 있기 때문이다. 물론 그렇게 하고 싶은 경우도 있겠지만 이번에는 아니다.
- PhotosViewController에 다음 프로퍼티를 추가하자

```swift
fileprivate let selectedPhotosSubject = PublishSubject<UIImage>()

var selectedPhotos: Observable<UIImage> {
    return selectedPhotosSubject.asObservable()
}
```

- 선택된 사진을 방출하는 private PublishSubject와 subject의 observable을 접근가능하게 하는 selectedPhotos 프로퍼티를 정의했다. 이 프로퍼티를 구독하는 것은 메인 컨트롤러가 방해 없이 사진의 observable을 관찰하는 방법이다.
- PhotosViewController는 이미 카메라롤의 사진을 읽고 컬렉션뷰에 나타내는 코드를 포함한다. 이제 남은 것은 사용자가 사진을 탭했을 때 선택된 사진을 방출하는 코드를 추가하는 것이다.
- collectionView(_:didSelectItemAt:) 메소드에서 선택된 이미지를 매치해서 셀에 뿌려준다.
- 다음으로 imageManager.requestImage()는 선택된 사진을 받아서 이미지와 info를 전달한다. 클로저에서는 selectedPhotosSubject에서 .next 이벤트를 방출하고 싶을 것이다.
- 클로저 내부의 가드구문 뒤에 다음을 추가하자

```swift
if let isThumbnail = info[PHImageResultIsDegradedKey as NSString] as? Bool, !isThumbnail {
    self?.selectedPhotosSubject.onNext(image)
}
```

- 이미지가 썸네일인지 풀버전인지 체크하기 위한 info 딕셔너리를 사용한다. imageManager.requestImage()는 각 사이즈마다 클로처를 한번씩 호출할 것이다. 풀 사이즈 이미지를 받았을 때 onNext(_) 를 호출하고 풀 사진을 내보낸다.
- 뷰컨트롤러 끼리 보내는 정보는 observable이 전부다. delgate 프로토콜이나 다른 것들은 필요 없다. 그리고 프로토콜을 사용하지 않으면 컨트롤러간의 관계는 더욱 간단하다.

## 선택된 사진 관찰하기
- 메인뷰컨트롤러로 돌아가서 선택된 사진을 관찰하는 부분을 완성시키자.
- actionAdd() 의 네비게이션 스택에 푸시하는 부분 바로 앞에 다음 코드를 추가하자

```swift
photosViewController.selectedPhotos
    .subscribe(onNext: { [weak self] newImage in
    
    }, onDisposed: {
        print("completed photo selection")
    })
    .addDisposableTo(bag)
```

- 컨트롤러를 푸시하기 전에, 선택된 사진에 대한 이벤트를 구독해야한다. 사용자가 사진을 탭하는 이벤트와 구독이 해제되는 이벤트에 집중하자.
- onNext 클로저에 다음 코드를 추가하자. 이전 코드와 같지만 이번에는 카메라롤의 사진을 추가한다.

```swift
guard let images = self?.images else { return }
images.value.append(newImage)
```

- 이제 앱을 실행하고 카메라롤에서 사진을 몇장 선택하면 콜라주가 완성될 것이다

## 어떤 dispose bag을 사용할까?
- 이제 뷰 컨트롤러간 서로 소통하는 객체를 갖게 되었고 다음으로 Observable 구독을 메모리 관리할 더 많은 dispose bag이 필요하다.
- 더 높은 수준의 observable 사용을 소개한다. Dispose bag은 구독 주기를 다른 객체의 주기와 결함하는 데 좋다. 그러나 어떤 다른 객체가 이 구독의 주기를 결정할 것인지 생각해야 한다
- 앱의 현재 버전의 문제점을 살펴보자. 앱을 실행하고 콜라주를 만들고 콘솔의 출력값을 보자
- "completed photo selection"이라는 메시지를 볼 수 없을 것이다. 우리는 onDispose 클로저에 프린트를 넣었다. 그러나 호출되지 않았다. 즉 구독이 해지되지 않아서 메모리에 남아있다는 뜻이다.
- RxSwift의 디버깅 툴로 Resources가 있다. 이는 observable, observer, disposable의 현재 남아있는 참조개수를 알려준다.
- RxSwift는 기본적으로 resource 카운팅을 비활성화한다. 활성화하는 방법을 배워보자

### resource counting 활성화하기
- build scheme에 custom swift compiler flag를 추가해서 활성화할 수 있다. RxSwift는 flag의 값에 따라 resource counting을 활성화 또는 비활성화한다.
- 코코아팟으로 RxSwift를 설치한 경우 팟파일에 flag를 추가하는 간단한 방법이 있다. 팟파일을 열고 다음을 추가하자

```swift
# enable tracing resources
post_install do |installer|
  installer.pods_project.targets.each do |target|
    if target.name == 'RxSwift'
      target.build_configurations.each do |config|
        if config.name == 'Debug'
          config.build_settings['OTHER_SWIFT_FLAGS'] ||= ['-D',
'TRACE_RESOURCES']
        end
end end
end end
```

- 각각의 타겟 구성요소에 루프를 돌면서 RxSwift에 대한 Debug schema를 찾아 flag를 추가한다.
- pod install 하면 완료

- Carthage를 사용하는 경우에는 resources.xcconfig 파일을 만들어 다음을 넣는다

```swift
OTHER_SWIFT_FLAGS = -DTRACE_RESOURCES
```

- 그리고나서 커맨드라인에 다음과 같이 re-build RxSwift 한다

```swift
 XCODE_XCCONFIG_FILE="<full path to>/resources.xcconfig" carthage update
--configuration Debug --platform iOS --no-use-binaries
```

- 시간이 좀 걸린다.

### 메모리 누수 찾기
- 이제 앱이 resource counting을 한다. 메인뷰 컨트롤러를 열어서 viewWillAppear(_:)에서 출력해보자

```swift
print("resources: \(RxSwift.Resources.total)")
```

- 뷰 컨트롤러가 화면에 나타날때마다 메모리에 있는 총 참조 갯수를 출력할 것이다. 앱을 실행해서 메인뷰 컨트롤러와 포토 컨트롤러를 왔다갔다 해보자. 출력창에 다음과 같이 출력될 것이다.

```swift
resources: 6
resources: 10
resources: 12
resources: 14
resources: 16
```

- 포토 뷰컨트롤러를 화면에 띄웠다가 날릴때마다 참조 갯수가 늘어난다. 다행히도 현재 상태는 아주 단순하다. 포토 뷰컨트롤러를 화면에 띄울 때 하나의 구독만을 추가한다. 이것이 유일한 구독이기 때문에 이게 원인임에 틀림없다.
- 구독을 메모리 관리하는 코드를 살펴보자

```swift
photosViewController.selectedPhotos
    .subscribe(onNext: {...}, onDisposed: {...})
    .addDisposableTo(bag)
```

- 구독이 완료되거나 에러가 나거나 구독이 해지될 때까지 구독이 유지되어야 한다.
- 구독이 언제 해지되나? 바로 bag 상수가 메모리에서 해지되거나 수동으로 dispose()를 호출할 때이다.
- 메인뷰 컨트롤러는 이 앱의 root 뷰컨트롤러 이기 때문에 해제되지 않는다. 그러므로 dispose bag도 해지되지 않는다. 우리는 점점 더 구독을 bag에 넣지만 bag은 해지되지 않는다는 말이다.
- 우리가 원하는 것은 구독의 주기를 포토뷰 컨트롤러의 주기에 맞추는 것이다. 이제 몇가지 방법을 살펴보자
- 첫째 방법은 메인뷰컨트롤러의 bag에 구독을 넣지 않고 포토뷰컨트롤러의 bag에 담는 것이다. 사용자가 포토뷰 컨트롤러를 화면에서 날릴때마다 bag이 해지되므로 구독도 해지된다
- 포토뷰컨트로러 파일을 열고 클래스에 bag을 추가하자

```swift
let bag = DisposeBag()
```

- 메인뷰컨트롤러 파일을 열고 bag에 담는 코드 부분을 수정하자

```swift
.addDisposableTo(photosViewController.bag)
```

- 이제 앱을 실행해보면 resource counting이 일정하게 유지되는 것을 볼 수 있다. 그리고 onDisposed handler 또한 이제 호출된다

```swift
resources: 6
resources: 11
completed photo selection
resources: 11
completed photo selection
resources: 11
completed photo selection
```

- 이 방법도 좋지만 더 좋은 방법도 있다. 포토뷰 컨트롤러가 화면에서 사라질 때 .completed 이벤트를 방출하는 것을 observser의 클로저에 추가하자. 이를 통해 구독이 완료되었다는 것을 모든 observer에게 알린다.
- 포토뷰컨트롤러 파일을 열고 viewWillDissappear(_:)에 다음을 추가하자

```swift
selectedPhotosSubject.onCompleted()
```

- 이제는 기존 observable을 custom observable로 바꿔보자

## custom observable 생성하기

### free function을 반응형 클래스로 감싸기
- PhotoWriter.swift 파일을 열고 RxSwift를 import 하자
- callback 프로퍼티와 init을 추가하자

```swift
private var callback: Callback

private init(callback: @escaping Callbak) {
    self.callback = callback
}
```

- 커스텀 init을 통해 인스턴스와 callback 설정을 한번에 할 수 있다.
- UIImageWriteToSavedPhotosAlbum()의 콜백 클로저로 사용될 메소드를 추가하자

```swift
func image(_ image: UIImage, didFinishSavingWithError error: NSError?, contextInfo: UnsafeRawPointer) {
    callback(error)
}
```

- 위의 메소드는 UIImageWriteToSavedPhotosAlbum()의 3개의 파라미터를 콜백에 전달한다. 몇가지만 사용하고 몇가지는 사용하지 않을 지라도 모두 포함하고 있어야 한다.
- 에러의 경우 클래스 프로퍼티에 저장된 기존 콜백에 error 객체를 전달한다. 에러가 발생하지 않으면 nil 처리한다.
- 이제 남은 것은 새로운 클래스와 콜백에 사용할 Observable을 생성하는 것이다. PhotoWriter에 다음 메소드를 추가하자

```swift
static func save(_ image: UIImage) -> Observable<Void> {
    return Observable.create({ observer in
    
    })
}
```

- save(_:) 메소드는 Observable<Void>를 반환한다 왜냐하면 .next 이벤트는 방출하지 않고 다만 에러 또는 완료만 나오기 때문이다.
- Observable.create() 는 새로운 Observable을 생성한다. 클로저 안에 다음을 추가하자

```swift
let writer = PhotoWriter(callback: { error in
    if let error = error {
        observer.onError(error)
    } else {
        observer.onCompleted()
    }
})
```

- 새로운 PhotoWriter 객체를 만들고 에러 발생 여부에 따라 에러 또는 완료를 나타내는 콜백을 설정한다
- 클로저에 다음을 추가하자

```swift
UIImageWriteToSavedPhotosAlbum(image, writer, #selector(PhotoWriter.image(_:didFinishSavingWithError:contextInfo:)), nil)
```

- 위의 함수는 콜라주를 저장하고 writer 객체에게 콜백을 설정한다. 마지막으로 클로저 밖으로 Disposable을 반환해야 하므로 다음을 추가하자

```swift
return Disposables.create()
```

- 완성된 save() 메소드는 다음과 같다

```swift
static func save(_ image: UIImage) -> Observable<Void> {

    return Observable.create({ observer in
        let writer = PhotoWriter(callback: { error in
            if let error = error {
                observer.onError(error)
            } else {
                observer.onCompleted()
            }
        })

        UIImageWriteToSavedPhotosAlbum(image, writer, #selector(PhotoWriter.image(_:didFinishSavingWithError:contextInfo:)), nil)

        return Disposables.create()
    })
}
```

- 왜 아무것도 방출하지 않는 Observable이 필요할까? 앞에서 배웠다시피 observable은 zero, .next, .completed, .error 등 여러가지를 생성할 수 있다.
- PhotoWriter의 경우 우리는 사진을 디스크에 저장하는 것이 완료되는 이벤트에만 관심이 있다. 따라서 완료되면 .completed를, 실패하면 .error를 사용한다.

### custom observable 구독하기
- 메인뷰컨트롤러 파일을 열고 저장버튼 메소드인 actionSave() 내부에 다음을 추가하자

```swift
guard let image = imagePreview.image else { return }

PhotoWriter.save(image)
    .subscribe(onError: { [weak self] error in
        self?.showMessage("Error", description: error.localizedDescription)
    }, onComplted: { [weak self] in
        self?.showMessage("Saved")
        self?.actionClear()
    })
    .addDisposableTo(bag)
```

- 콜라주를 저장하기 위해 PhotoWriter.save(image)를 호출한다. 반환된 Observable을 구독해서 디스크 저장이 완료되거나 에러가 났을 때 메시지를 출력한다. 또한 저장이 성공하고 나면 콜라주를 지워버린다.