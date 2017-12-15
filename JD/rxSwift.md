# rxSwfit
- 버튼 토글 시 state 변화에 따라 업데이트 하고 싶음
- 함수가 호출될때만 하는게 아니라 값이 업데이트될때마다 비동기적으로 발생
- 값에 접근해서 업데이트를 하지 못하게 감싸고 단지 값이 변화했을 때 반응만 할 수 있도록 하여서 서로 다른 곳에서 동시에 하나의 값을 변화시키는 충돌이 나는 것을 방지할 수 있다
- rxCocoa = rxSwift에 관련한 UIKit

## 1. variable
- 변화하는 값을 갖고 있음 값이 변화할 때 이벤트 방출.
- asObservable을 달면 이 변화를 계속 관찰하겠다
- 어떤 타입의 데이터든 보관가능

## 2. observable
- variable로부터 true, false 를 받는다


## 3. 연산자
- filter = true 만, 데이터 업데이트시 반응을 할지 안할지 조건 주기
- throttle = 시간에 따른 필터링
- map = bool 타입을 다른 타입으로, "이제 불이 켜졌어"

## 4. subscribe
- 구독 시작


```swift

let names = Variable(["jun", "crong"])

names.asObservable()
  //값이 바뀌는 인터벌이 최소 0.5초 이상일때에만 반응
  .throttle(0.5, scheduler: MainScheduler.instance)
  // 값 업데이트시 반응할지 결정하는 조건
  .filter { value in
    return value.count > 1
  }
  //이 단계에서 건내주는 값 확인
  .debug()
  // [Stirng] -> String
  .map { value in
    return "users : \(value)"
  }
  // 시작
  .subscribe { value in
  print(value)
}

names.value = ["jun", "hoemoon"]
names.value = ["smith", "jj"]

delay(seconds: 2) {
  names.value = ["hanna", "knight"]
}
delay(seconds: 3) {
  names.value = ["jj", "jk"]
}
```