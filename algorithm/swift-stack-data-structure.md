# Swift algorithm club: Swift Stack Data Structure
> [Swift Algorithm Club: Swift Stack Data Structure](https://www.raywenderlich.com/149213/swift-algorithm-club-swift-stack-data-structure)를 번역하였습니다.

## Getting Started
- stack은 배열같지만 제한된 기능을 지녔다. 스택 위에 새로운 원소를 넣을 때 push만 가능하고, 위에서 원소를 제거하기 위해 pop을 사용하고, 제거하지 않고 위의 원소를 리턴하는 peek을 사용할 수 있다
- 왜 이렇게 하고 싶을까? 많은 알고리즘에서 우리는 리스트에 원소를 더하고 뺀다. 그리고 이 때 넣고 빼는 순서가 중요하다.
- 스택은 Last In, First Out을 보장한다. 가장 최근에 삽입한 원소가 가장 먼저 제거된다. (Queue는 First In, First Out 을 보장한다.)

### Note
- 배열을 사용해서 스택을 구현해볼 것이다. 이것은 가장 효율적인 접근은 아니지만 이번 튜토리얼 목적을 위해 해보자

## Stack Operations
- 스택은 제한된 기능을 지닌다. 책더미를 예로 들면 스택의 기능을 알 수 있다.

### Push
- 스택에 원소를 추가하고 싶을 때 스택위에 push 하면 된다. 책더미 위에 책을 추가하는 것으로 생각하면 된다.

### Peek
- 스택은 가장 위에 있는 책 말고는 접근할 수 없다. peek 메소드는 스택의 가장 위에 있는 것을 체크할 수 있다.

### pop
- 스택에서 책 한권을 빼고 싶을 때 pop을 사용할 수 있다. 책더미에서 가장 위의 책을 한권 빼는 것과 같다.

## stack 구현해보기
```swift
struct Stack {
    fileprivate var array: [String] = []
}
```
- 배열을 사용해 Stack을 선언했다. 이제 push, pop, peek을 구현해보자

### Push
```swift
mutating func push(_ element: String) {
    array.append(element)
}
```
- push 메소드는 스택 맨 위에 더할 element 를 매개변수로 갖는다
- push 메소드는 배열의 맨 마지막에 삽입되어야 한다. 배열의 가장 앞에 추가하는 것은 O(n) 작업으로서 고비용 작업이다. 왜냐하면 기존의 원소의 위치를 변경해야하기 때문이다. 그러나 맨 마지막에 삽입하는 것은 O(1) 작업으로서 배열의 크기에 상관없이 같은 시간이 든다.

### Pop
- pop 메소드 구현 또한 쉽다.
```swift
mutating func pop() -> String? {
    return array.popLast()
}
```
- 옵셔널 String을 리턴한다. 스택이 비어있을 경우 리턴값이 없으므로 옵셔널이다.
- 스위프트의 배열은 가장 마지막 원소를 제거하는 데 용이한 popLast 메소드를 포함하므로 이걸 그냥 사용하면 된다.

### Peek
- 스택의 가장 위 원소를 체크하는 것으로서 구현하기 쉽다. 스위프트 배열은 last 프로퍼티를 가지며 이것은 배열을 변경하지 않은 상태로 배열의 가장 마지막 원소를 리턴한다.
```swift
func peek() -> String? {
    return array.last
}
```
- peek 메소드는 pop과 유사하지만 peek은 기존의 배열을 변형시키지 않으므로 mutating 키워드가 필요없다.

## Give it a Whirl!
- 이제 스택을 테스트할 준비가 되었다. 플레이그라운드 끝부분에 다음을 추가하자
```swift
var rwBookStack = Stack()

rwBookStack.push("3D Games by Tutorials")

rwBookStack.peek()

rwBookStack.pop()

rwBookStack.pop()
```
- 플레이그라운드 오른쪽 부분에서 각 라인의 결과를 볼 수 있다
- 1. 스택을 변형시킬 것이므로 var 로 선언했다
- 2. 스택에 string을 push했다
- 3. peek을 호출했을 때 스택의 가장 윗부분에 있는 문자열인 "3D Games by Tutorials"를 리턴해야 한다
- 4. pop을 호출 시 스택의 가장 위에 있는 문자열인 "3D Games by Tutorials"를 리턴하고 이것은 배열에서 제거된다
- 5. 다시 한번 pop 호출시 스택은 비어있으므로 nil을 반환한다

## CustomStringConvertible
- 현재까지 스택에 있는 원소들을 시각화 하는 것은 꽤 어렵다. 다행히도 스위프트는 CustomStringConvertible이라는 프로토콜을 가지며 이는 object를 문자로 나타내는 방식을 설정할 수 있도록 도와준다. 아래 코드를 위에서 구현한 스택의 아래에 추가하자
```swift
extension Stack: CustomStringConvertible {
    var description: String {
        let topDivider = "---Stack---\n"
        let bottomDivider = "\n-----------\n"

        let stackElements = array.reversed().joined(seperator: "\n")
        return topDivider + stackElements + bottomDivider
    }
}
```
- 1. extension을 만들어서 프로톹콜 적용하기
- 2. description 프로퍼티는 프로토콜을 준수하기 위해 요구되는 것이다
- 3. \n 은 밑에 줄로 건너뛰기를 의미한다
- 4. 스택 안의 원소를 보여주기 위해서는 배열에 원소들을 쌓아야 한다. 배열의 끝부분에 원소를 추가해왔기 때문에 우선 배열을 뒤집어야 한다. 그리고 나서 joined 메소드를 사용해서 배열의 모든 원소들 사이에 "\n"를 넣고 합친다.
- 5. 마지막으로 topDivier와 bottomDivider 사이에 stackElements를 넣고 이를 description의 결과로 반환한다

- 이전의 테스트 코드를 제거하고 플레이그라운드 끝부분에 다음을 추가하자
```swift
var rwBookStack = Stack()
rwBookStack.push("3D Games by Tutorials")
rwBookStack.push("tvOS Apprentice")
rwBookStack.push("iOS Apprentice")
rwBookStack.push("Swift Apprentice")
print(rwBookStack)
```
- 플레이그라운드 끝부분의 콘솔에서 스택의 내용을 다음과 같이 나타낼 것이다
```swift
---Stack---
Swift Apprentice
iOS Apprentice
tvOS Apprentice
3D Games by Tutorials
-----------
```

## 제네릭
- 현재 당신의 스택은 문자열만 저장할 수 있다. 만약 integer를 저장하는 스택을 만들고 싶다면 완전 새로운 스택을 새로 만들어야 한다. 다행히도 스위프트는 제네릭을 지원한다
스택을 정의하는 부분을 다음과 같이 바꾸자
```swift
struct Stack<Element> {

}
```
- 대괄호는 제네릭 표시이며 스택이 스위프트의 모든 타입을 담을 수 있도록 해준다. String이라고 썼던 부분을 Element로 변경하면 당신의 스택은 다음과 같다
```swift
struct Stack<Element> {
  fileprivate var array: [Element] = []
  
  mutating func push(_ element: Element) {
    array.append(element)
  }
  
  mutating func pop() -> Element? {
    return array.popLast()
  }
  
  func peek() -> Element? {
    return array.last
  }
}

```
- 마지막으로 description 프로퍼티를 변경하자.
```swift
// previous
let stackElements = array.reversed().joined(separator: "\n")

// now
let stackElements = array.map { "\($0)" }.reversed().joined(separator: "\n")
```
- 배열의 원소들을 문자화한 후 합치는 것으로 변경했다. 당신의 스택은 제네릭이므로 당신이 합칠 원소를 문자화해야한다.
- 마지막으로 스택 인스턴스 생성한 부분을 String 타입으로 구체화하자
```swift
var rwBookStack = Stack<String>()
```
- 이제 당신의 스택은 모든 타입을 담을 수 있다.

## Finishing Touches
- 스택에서 자주 쓰이는 두가지 프로퍼티가 더 있다. 스택이 비어있는지 확인하고 싶을 때 isEmpty를 사용하고, 원소 갯수를 파악하고 싶을 때 count를 사용한다. Stack 구현부분 내부에 다음의 연산 프로퍼티를 추가하자
```swift
var isEmpty: Bool {
  return array.isEmpty
}

var count: Int {
  return array.count
}
```