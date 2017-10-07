# transforming operators in practice
> rxswift-reactive-programming 책을 번역 및 재가공하였습니다

## 웹에서 데이터 가져오기
- 우리는 URLSession을 사용했다. url을 포함하는 URLRequest를 생성해서 인터넷에 요청하면 잠시 후 서버응답을 받는다. RxCocoa는 Rx 관련 api를 제공하는 rxSwift와는 달리, uikit과 cocoa 관련 api를 제공한다. 여기서 우리는 default RxCocoa URLSession extension을 사용해서 깃헙 api로부터 빨리 json 파일을 가져올 것이다.

### 1. map 연산자 사용해서 url request 생성하기
- 깃헙 서버에 보낼 URLRequest를 생성해야 한다. ActivityController.swift 를 열고 살펴보자. viewDidLoad 에서 뷰컨트롤러의 UI를 구성하고 완료되면 refresh()를 호출하고 fetchEvents()를 호출해서 repo 이름을 전달한다.
- fetchEvent() 에 다음을 추가하자

```swift
let response = Observable.from([repo])
```

- 레퍼지토리의 풀네임 문자열을 가지고 request 생성을 시작한다
- map 연산자를 통해 string을 url로 변형하자

```swift
.map{ urlString -> URL in
    return URL(string: "https://api.github.com/repos/\(urlString)/events")!
}
```

- 이제 url을 만들었으니 요청을 생성하자.

```swift
.map { url -> URLRequest in
    return URLRequest(url: url)
}
```

- 우리는 웹주소를 가지고 map 연산자를 통해 url을 urlrequest로 변형했다.

### 2. flatmap 연산자 사용해서 웹서버 응답 기다리기

```swift
.flatMap { request -> Observable<(HTTPURLResponse, Data)> in
    return URLSession.shared.rx.response(request: request)
}
```

- shared URLSession 객체에 대해 RxCocoa 의 response 메소드를 사용한다. 이 메소드는 Observable<(HTTPURLResponse, Data)>를 반환하는데 이는 앱이 웹 서버로부터 응답을 완전히 받을 때 완료된다.
- flatMap 연산자는 프로토콜이나 델리게이트 없이 웹서버에 요청을 보내고 응답을 받는 것을 도와준다. 마지막으로 요청 결과를 구독하기 위해서 shareReply(1) 연산자를 사용한다. 가장 최근에 방출된 이벤트 1개를 받는다

```swift
.shareReply(1)
```

### share 대신에 shareReply를 사용한 이유
- URLSession.rx.response 는 요청을 서버에 보내서 받은 응답을 방출한다. 그런데 만약 observable이 완료된 후 다시 구독한다면 새로운 구독을 생성하고 서버에 똑같은 요청을 보낼 것이다. 이를 방지하기 위해 shareReply를 사용한다. 이 연산자는 가장 최근의 이벤트를 받아놨다가 새로운 구독자에게 전달한다. 그러므로 요청이 완료되고 새 구독을 시작하더라도 즉시 최근 응답을 받을 수 있다.
- shareReply 연산자는 완료가 기대되는 observable에 사용하기 좋다. 그래서 구독을 다시 하더라고 observable이 재생성되는 것을 방지한다. 또한 observable이 자동으로 최신 이벤트를 받도록 할 때 사용할 수 있다

## 응답 변형하기
- URLSession은 Data 객체를 주는데 이것은 우리가 바로 쓸 수 없는 형태이므로 JSON 형태로 변형해야 한다.

```swift
response
    .filter { response, _ in
        return 200..<300 ~= response.statusCode
        // ~= 는 왼쪽의 범위가 오른쪽 값을 포함하는지의 Bool값이다
    }
```

- filter 연산자를 사용해 에러 응답은 무시하고 성공 응답만 받는다.

```swift
.map { _, data -> [[String:Any]] in
    guard let jsonObject = try? JSONSerialization.jsonObject(with: data, options: []),
        let result = jsonObject as? [[String:Any]] else {
            return []
        }
    return result    
}
```

- 1. 이전과 달리 response object를 무시하고 오직 response data 만 받는다
- 2. JSON 객체의 배열타입을 반환할 것이라고 컴파일러에게 알린다
- 3. JSONSerialization을 사용해서 response data를 decode 하고 결과를 반환
- 4. 실패하면 빈 배열을 반환한다

```swift
.filter { objects in
    return objects.count > 0
}
```

- 에러 응답이나 빈 응답을 버린다.
- Event.swift를 열어보면 json object를 파라미터로 받는 생성자가 있다. dictionary 프로퍼티는 json Object 형태의 이벤트를 방출한다.
- ActivityController.swift로 돌아가서 다음을 추가하자

```swift
.map { objects in
    return objects.map(Event.init)
}
```

- map 연산자는 [[String:Any]]를 받아서 [Event]를 반환한다. 즉 각각의 [String:Any]를 Event(dictionary: [String:Any])로 변형한다.
- 위에서 map이 2개 사용되었는데 첫번째는 Observable에 대한 비동기 map 이고 두번째는 array에 대한 동기적 map 이다.
- 코드를 단순화 하기 위해 UI 관련 코드는 분리하자.

```swift
.subscribe(onNext: { [weak self] newEvents in
    self?.processEvents[newEvents]
})
.addDisposableTo(bag)
```

### 1. 응답 가공하기
- 우리는 string을 이용해서 요청을 만들고 깃헙에 전송해서 응답을 받았다. 그리고 json 형태로 변형해서 object 형태로 만들었다. 이제는 사용자에게 보여줄 차례다
- ActivitiyController의 아무곳에 다음을 추가하자

```swift
func processEvents(_ newEvents: [Event]) {

}
```

- 이 메소드는 최신 50개의 이벤트를 받아서 뷰 컨트롤러의 events 프로퍼티에 저장하는 기능이다. 메소드에 다음을 추가하자

```swift
var updatedEvents = newEvents + events.value
if updatedEvents.count > 50 {
    updatedEvents = Array<Event>(updatedEvents.prefix(upTo: 50))
}

events.value = updatedEvents
```

- 새로 가져온 이벤트를 events.value 리스트에 추가한다. 그리고 리스트에 들어가는 이벤트를 50개로 제한한다.
- processEvents() 의 마지막에 다음을 추가하자

```swift
tableView.reloadData()
```

- 앱을 실행하면 깃헙으로부터 최신활동을 볼 수 있다. 현재는 우리가 스레드를 관리하지 않으므로 테이블뷰에 결과가 출력되는데 좀 걸릴 수 있다. 왜냐하면 background 스레드에서 UI를 업데이트하기 때문이다. 이 책의 후반후에서 이를 수정할 것이니 기다려라
- viewDidLoad 에서 table refresh control을 설정했으므로 테이블을 아래로 당기면 refresh()가 호출되어 reload 할 것이다.
- 이벤트를 가져오는 것을 마쳤을 때 refresh control을 숨기기 위해서 tableView.reloadData() 다음에 다음 코드를 추가하자

```swift
refreshControl?.endRefreshing()
```

- endRefreshing 은 refresh control을 숨기고 tableView를 디폴트 상태로 리셋한다


## 잘못된 입력값 처리하기
- Event.swift를 열고 생성자를 살펴보자. 서버로부터 받은 object가 틀린 이름의 key를 포함한다면? 앱이 충돌할 것이다. init을 실패생성자로 바꾸자

```swift
init?(dictionary: AnyDict)
```

- 이를 통해 앱이 충돌하는 대신 생성자는 nil을 반환할 수 있다. fatalError() 를 다음으로 대체하자

```swift
return nil
```

- Xcode에 에러가 뜰 것이다. ActivityController 의 구독이 [Event]를 기대하지만 [Event?]를 받는다고 컴파일 에러가 뜬다.
- AcitivyController 에서 우리는 map연산자를 통해 jsonOjects를 이벤트로 변형한다. 하지만 nil을 거를 수 없으므로 결과가 변할 것이다. 우리가 원하는 것은 nil을 반환하는 이벤트를 거르는 것이다. 다행히 flatMap 연산자를 사용하면 된다

```swift
objects.flatMap(Event.init)
```

- 어떤 Event.init은 nil을 반환할 것이나 flatMap이 nil 값을 제거할 것이다. 따라서 이제 결과값은 비옵셔널이 된다. 

## 디스크에 객체 저장
- 사용자가 앱을 실행했을 때 지난번에 가져온 이벤트를 즉시 볼 수 있어야 한다.
- plist 파일에 저장해보자. 객체의 크기는 작아서 plist 면 충분하다. realm에 저장하는 것은 나중에 배운다.
- ActivityController에 새로운 프로퍼티를 추가하자

```swift
private let eventsFileURL = cachedFileURL("events.plist")
```

- eventsfileurl은 이벤트파일을 디스크에 저장할 주소이다. 저장할 주소를 만들기 위해 cachedFileURL 함수를 구현하자. 클래스의 바깥에 추가하자

```swift
func cachedFileURL(_ fileName: String) -> URL {

    return FileManager.default
        .urls(for: .cachedDirectory, in: .allDomainsMask)
        .first!
        .appendingPathComponent(fileeName)
}
```

- 이제 processEvents() 끝에 다음을 추가하자

```swift
let eventsArray = updatedEvents.map{ $0.dictionary } as NSArray

eventsArray.write(to: eventsFileURL, atomically: true)
```

- updatedEvents 를 NSArray 타입의 json 객체로 만들어서 eventsArray에 저장한다. 일반 배열과 달리 NSArray는 내용을 파일에 저장하는 직관적인 메소드가 특징이다.
- write()를 사용해서 원하는 곳에 배열을 저장한다
- viewDidLoad()에서 디스크에 저장된 파일을 읽어오자

```swift
let eventsArray = (NSArray(contentsOf: eventsFileURL) 
    as? [[String:Any])]) ?? []

events.value = eventsArray.flatMap(Event.init)
```

- init을 사용해 NSArray를 생성하고 plist 파일로부터 objects 리스트를 가져오고 [[String:Any]] 형태로 타입캐스팅 한다
- flatmap을 사용해 json을 이벤트객체로 변형하고 실패를 거른다.
- 시뮬레이터에서 앱을 지우고 다시 실행하고 이벤트가 테이블에 보일때까지 기다려라 그리고 다시 앱을 실행하면 전보다 빨리 보일 것이다

## 최근에 변경된 헤더를 다음 요청에 추가하기
- 이번에는 GitFeed 가 이전에 가져오지 않은 이벤트만 요청하도록 개선해보자. 
- ActivityController에 파일 이름을 저장하는 새 프로퍼티를 추가하자

```swift
private let modifiedFileURL = cachedFileURL("modified.txt")
```

- 단순히 문자만 저장할 것이기 때문에 이번에는 plist 가 필요없다. 서버가 json 형식으로 응답하는 헤더의 값을 저장할 것이다. 받은 헤더를 다시 다음 요청과 함께 서버에 보낼 것이다. 그래서 이전에 가져온 이벤트가 있는지 확인할 것이다.
- ActivityController에 변수 헤더값을 담는 새 프로퍼티를 추가하자

```swift
fileprivate let lastModified = Variable<NSString?>(nil)
```

- nsstring 타입이 디스크에 읽고 쓰기 용이하므로 이 타입을 사용할 것이다.
- viewDidLaod의 refresh 호출 바로 위에 다음을 추가하자

```swift
lastModified.value = try? NSString(contentsOf: modifiedFileURL. usedEncoding: nil)
```

- 이전에 헤더의 값을 파일에 저장했었다면, NSString을 생성할 것이다. 없다면 nil을 반환할 것이다.
- fetchEvents() 에서 에러 응답을 필터링하는 두번째 구독을 추가하자

```swift
response
    .filter { response, _ in
        return 200..<400 ~= response.statusCode
    }
```

- 이제 헤더가 없는 응답을 거르고 헤더의 값을 NSString으로 변형해야 한다

```swift
.flatMap { response, _ -> Observable<NSString> in
    guard let value = response.allHeaderFields["Last-Modified"] as? NSString else {
        return Observable.never()
    }

    return Observable.just(value)
}
```

- guard 구문을 사용해서 응답이 http header를 갖는지 체크한다. 헤더를 NSString으로 타입캐스팅해서 Observable<NSString>으로 반환한다. 아니면 절대 방출하지 않는 Observable을 반환한다

- 이제 lastModifeid 프로퍼티를 업데이트 하고 디스크에 저장하자

```swift
.subscribe(onNext: { [weak self] modifiedHeader in
    guard let strongSelf = self else { return }
    strongSelf.lastModified.value = modifiedHeader

    try? modifiedHeader.wirte(to: strongSelf.modifiedFileURL, atomically: true, encoding: String.Encoding.urf8.rawValue)
})
.addDisposableTo(bag)
```

- lastModified의 값을 업데이트한다. 그리고 디스크에 저장한다. 마지막으로 dispose bag에 담는다
- fetchEvents의 url을 urlrequest로 변형하는 map을 다음으로 수정해서 서버로부터 받은 헤더를 다음 요청에 추가하자

```swift
.map { [weak self] url -> URLRequest in
    var request = URLRequest(url: url)

    if let modifiedHeader = self?.lastModified.value {
        request.addValue(modifiedHeader as String, forHTTPHeaderField: "Last-Modified")
    }

    return request
}
```

- 이 추가된 헤더는 헤더에 표시된 날짜 이전의 이벤트에는 관심없다고 깃헙에게 말한다. 이를 통해 서버 트래픽을 향상시키고 우리의 제한된 에벤트 갯수를 낭비하지 않을 것이다. 