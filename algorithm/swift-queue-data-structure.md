# Swift Algorithm Club: Swift Queue Data Structure
> [Swift Algorithm Club: Swift Queue Data Structure](https://www.raywenderlich.com/148141/swift-algorithm-club-swift-queue-data-structure)를 번역하였습니다.

- 이번 튜토리얼에서 스위프트3 로 큐를 구현하는 방법을 배울 것이다. 큐는 가장 인기있는 데이터구조 중 하나이며 스위프트로 구현하기 꽤 간단하다.

## 시작하기
- 큐는 끝부분에 아이템을 추가하거나 앞부분의 아이템을 제거하는 리스트이다. 즉 큐는 가장 처음 추가한 아이템이 가장 먼저 삭제되는 것을 보장한다. First in, First out
- 왜 이것이 중요할까? 많은 알고리즘에서 리스트에 아이템을 추가하고 삭제하는데 이때 이 순서가 중요하다.
- 가장 먼저 추가한 아이템이 가장 먼저 나온다. 이것은 공정하다. 반면 스택은 가장 늦게 추가한 것이 가장 먼저 나온다.

### 예제
- 큐를 이해하는 가장 쉬운 방법은 어떻게 사용되는지 보는 것이다
- 큐가 있다고 가정하자. 다음은 숫자를 큐에 넣는 방법이다
```swift
queue.enqueue(10)
```
- 큐는 이제 [10]이 된다. 큐에 다음 숫자를 넣어보자
```swift
queue.enqueue(3)
```
- 이제 [10, 3]이 된다. 또 넣어보자
```swift
queue.enqueue(57)
```
- 이제 [10, 3, 57] 이다. 가장 앞의 숫자를 추출해보자
```swift
queue.dequeue()
```
- 위에서는 10을 반환할 것이다. 왜냐하면 첫번째로 넣은 숫자가 가장 먼저 추출되기 때문이다. 큐는 이제 [3, 57]이다. 큐 안의 숫자가 한칸씩 앞으로 이동했다
```swift
queue.dequeue()
```
- 위에서 3을 반환할 것이다. 그 다음엔 57을 반환할 것이다. 만약 큐가 비었다면 nil을 반환할 것이다.

## 큐 구현하기
- 이번 섹션에서 Int를 담는 간단한 큐를 구현할 것이다. 비어있는 큐를 만들자
```swift
public struct Queue {

}
```

### Enqueue
- 큐는 enqueue 메소드를 필요로 한다. 큐를 구현하기 위해서 linked list를 사용할 것이다. 괄호 안에 다음을 추가하자
```swift
fileprivate var list = LinkedList<Int>()

public mutating func enqueue(_ element: Int) {
    list.append(element)
}
```
- 1. LinkedList 변수를 추가했다 이것은 큐의 아이템을 담는데 사용될 것이다.
- 2. enqueue 메소드를 추가했다. 이것은 기존의 LinkedList를 변화시킬 것이므로 mutating 키워드를 붙였다

### Dequeue
- 큐는 dequeue 메소드도 필요하다
```swift
public mutating func dequeue() -> Int? {
    guard !list.isEmpty, let element = list.first else { return nil }

    list.remove(element)

    return element.value
}
```
- 1. 큐의 첫번째 아이템을 반환하는 dequeue 메소드를 추가했다. 큐가 비어있을 때 nil을 반환하므로 반환타입은 옵셔널이다. 이 메소드는 linkedlist의 아이템을 제거하므로 mutating을 붙여야 한다
- 2. 큐가 비어있는 상황을 처리하기 위해 guard 구문을 사용했다. 큐가 비어있으면 else 블록으로 넘어간다

### Peek
- 큐는 아이템 제거없이 큐의 첫 아이템을 반환하는 peek 메소드를 갖는다
```swift
func peek() -> Int? {

    return element.value
}
```

### isEmpty
- 큐는 비어있을 수 있다. 현재 LinkedList의 상태를 반환하는 isEmpty 프로퍼티를 추가하자
```swift
public var isEmpty: Bool {
    return list.isEmpty
}
```

### 큐 출력하기
- 새로운 큐를 만들어보자. 큐 구현부분 바깥쪽에 다음을 추가하자
```swift
var queue = Queue()
queue.enqueue(10)
queue.enqueue(3)
queue.enqueue(57)
```
- 생성한 큐를 콘솔에 출력해보자
```swift
print(queue)
```
- 다음과 같이 출력된다
```swift
Queue
```
- 이 출력값은 도움이 안된다. 자세히 들여다보기 위해 큐에 CustomStringConvertable 프로토콜을 적용하자.

```swift
extension Queue: CustomStringConvertible {
    public var description: String {
        return list.description
    }
}

```
- 1. extension을 선언하여 프로토콜을 적용했다. 프로토콜을 통해서 String 타입의 연산 프로퍼티인 description을 구현했다
- 2. description은 연산 프로퍼티로서 읽기 전용이고 String을 반환한다
- 3. 현재 LinkedList의 description을 반환한다
- 다시 큐를 출력해보면 다음과 같이 나온다
```swift
"[10, 3, 57]"
```

## 제네릭 큐 구현하기
- 앞에서 Int를 담는 큐를 구현했다. 이번에는 제네릭을 적용해서 모든 타입을 담을 수 있는 큐를 구현해보자
```swift
// 1
public struct Queue<T> {

  // 2
  fileprivate var list = LinkedList<T>()

  public var isEmpty: Bool {
    return list.isEmpty
  }
  
  // 3
  public mutating func enqueue(_ element: T) {
    list.append(element)
  }

  // 4
  public mutating func dequeue() -> T? {
    guard !list.isEmpty, let element = list.first else { return nil }

    list.remove(element)

    return element.value
  }

  // 5
  public func peek() -> T? {
    return list.first?.value
  }
}
```
- 1. 제네릭 타입인 T를 갖는 큐로 변경했다
- 2. 우리 목적은 큐가 어떤 타입이든 담을 수 있게 하는 것이다. 따라서 아이템을 담는 LinkedList역시 T를 담아야 한다
- 3. enqueue 메소드 역시 T 타입을 담는다
- 4. dequeue 메소드 역시 T 타입을 반환
- 5. peek 메소드 역시 T 타입 반환

- 테스트 코드도 수정하자
```swift
var queue = Queue<Int>()
queue.enqueue(10)
queue.enqueue(3)
queue.enqueue(57)
```
- 다른 타입을 담는 큐 인스턴스를 생성할 수도 있다
```swift
var queue2 = Queue<String>()
queue2.enqueue("mad")
queue2.enqueue("lad")
if let first = queue2.dequeue() {
  print(first)
}
print(queue2)
```