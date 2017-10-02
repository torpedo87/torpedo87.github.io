# Swift Algorithm Club: Swift Linked List Data Structure
> [Swift Algorithm Club: Swift Linked List Data Structure](https://www.raywenderlich.com/144083/swift-algorithm-club-swift-linked-list-data-structure)를 번역하였습니다.

## 시작하기
- Linked list는 데이터 아이템의 연결이다. 각각의 아이템은 node라고 불린다. Linked list는 주요 2가지 타입이 존재한다. 
- Singly linked list는 각각의 노드가 다음 노드의 참조만을 갖는다.
- Doubly linked list는 각각의 노드가 이전 노드와 다음노드의 참조를 갖는다
- 어디서 리스트가 시작하고 끝나는지를 추적할 필요가 있다. head, tail이라고 불리는 포인터에서 주로 시작하고 끝이 난다.

## 노드 구현하기
- linked list는 노드로 구성된다. 먼저 기본적인 노드 클래스를 만들자.
```swift
public class Node {

}
```

### Value
- 노드는 값을 갖는다. 노드 클래스 안에 다음을 추가하자
```swift
var value: String

init(value: String) {
    self.value = value
}
```
- String 타입의 value 프로퍼티를 선언했다. 다른 타입으로 해도 상관없다. 또한 생성자를 선언해서 모든 비옵셔널 프로퍼티를 초기화한다.

### Next
- 노드는 value 이외에도 다음 노드를 참조하는 포인터를 갖는다.
```swift
var next: Node?
```
- 노드 타입의 next 프로퍼티를 선언했다. 이 때 next는 옵셔널이다. 왜냐하면 마지막 노드는 다음 노드가 없기 때문이다.

### Previous
- doubly linked list는 이전 노드의 참조를 갖는 포인터 또한 필요하다
```swift
weak var previous: Node?
```
- Note: 순환참조를 피하기 위해서 weak 를 붙여 약한 참조로 선언했다. 만약 default 참조인 강한 참조를 하면 삭제를 한 후에도 참조가 지속된다.
---

## Linked list 구현하기
- 위에서 노드를 만들었으니 이를 활용해서 linked list를 만들자
```swift
public class LinkedList {
    fileprivate var head: Node?
    private var tail: Node?

    public var isEmpty: Bool {
        return head == nil
    }

    public var first: Node? {
        return head
    }

    public var last: Node? {
        return tail
    }
}
```
- 리스트의 시작과 끝을 선언했다. 다른 helper 메소드도 구현해보자

### append
- 리스트에 새로운 노드 추가할 때 append(value:) 메소드를 사용한다
```swift
public func append(value: String) {
    let newNode = Node(value: value)

    if let tailNode = tail {
        newNode.previous = tailNode
        tailNode.next = newNode
    } else {
        head = newNode
    }

    tail = newNode
}
```
- 1. value를 갖는 새로운 노드를 생성한다
- 2. tailNode가 nil이 아니면 리스트에 무언가 존재한다는 뜻이다. 이 경우 새로운 노드를 삽입하면 새 노드의 previous는 tailNode를 가리킨다. 또한 tailNode의 next는 newNode 가 된다.
- 3. 마지막으로 리스트의 tail이 곧 newNode가 된다

## linked list 출력하기
```swift
let dogBreeds = LinkedList()
dogBreeds.append(value: "Labrador")
dogBreeds.append(value: "Bulldog")
dogBreeds.append(value: "Beagle")
dogBreeds.append(value: "Husky")

print(dogBreeds)
```
- 콘솔에 다음과 같이 출력될 것이다
```swift
LinkedList
```
- 좀 더 리스트를 자세히 파악하기 위해 리스트에 CustomStringConvertable 프로토콜을 적용하자. 
```swift
extension LinkedList: CustomStringConvertible {

    public var description: String {

        var text = "["
        var node = head

        while node != nil {
            text += ?\(node.value)"
            node = node.next
            if node != nil { text += ", " }
        }

        return text + "]"
    }
}
```
- 1. 프로토콜을 적용하고 연산 프로퍼티인 description을 추가했다
- 2. descriptoin 프로퍼티는 읽기전용이며 String을 반환한다.
- 3. text 변수를 선언했다. 이것은 리스트의 모든 노드가 갖는 value를 모을 것이다
- 4. 반복문을 돌면서 노드의 value를 text에 추가한다
- 5. text 변수를 닫는다
```swift
"[Labrador, Bulldog, Beagle, Husky]"
```

## 특정 node에 접근하기
- index를 이용해서 링크드 리스트의 노드에 접근해보자
```swift
public func nodeAt(index: Int) -> Node? {

    if index >= 0 {
        var node = head
        var i = index

        while node != nil {
            if i == 0 { return node }
            i -= 1
            node = node!.next
        }
    }

    return nil
}
```
- 1. index가 0 이상인지 체크해서 무한루프를 방지한다
- 2. 특정 index 의 노드에 도착할때까지 루프를 돈다
- 3. index가 음수이거나 노드의 총 갯수보다 크면 nil을 반환한다

## 노드 전부 삭제하기
- 모든 노드를 제거하는 것은 쉽다. head와 tail을 nil 로 지정하면 된다
```swift
public func removeAll() {
    head = nil
    tail = nil
}
```

## 특정노드 삭제하기
- 1. 첫번째 노드를 삭제하는 경우: head, previous 를 업데이트 해야한다
- 2. 중간 노드를 삭제하는 경우: previous, next를 업데이트 해야한다
- 3. 마지막 노드를 삭제하는 경우: next, tail 업데이트 해야한다
```swift
public func remove(node: Node) -> String {

    let prev = node.previous
    let next = node.next

    //첫번째 노드 삭제가 아닌경우
    if let prev = prev {
        prev.next = next
    } else {
        //첫번째 노드 삭제하는 경우
        head = next
    }
    next?.previous = prev

    //마지막 노드 삭제하는 경우
    if next == nil {
        tail = prev
    }

    //삭제할 노드의 포인터 삭제
    node.previous = nil
    node.next = nil

    return node.value
}
```
- 1. 첫번째 노드 삭제가 아닌경우 next 업데이트
- 2. 첫번째 노드 삭제인 경우 head 업데이트
- 3. 삭제한 노드의 previous 업데이트
- 4. 마지막 노드 삭제하는 경우 tail 업데이트
- 5. 삭제된 노드의 포인터 연결 해제
- 6. 삭제된 노드의 값 반환

## 제네릭
- 지금까지 String 값을 담는 linked list를 살펴보았다. 그리고 append, remove, access 메소드를 만들었다. 이번에는 제네릭을 사용해 임의의 타입을 담을 수 있는 Linked list로 변경해보자
```swift
public class Node<T> {

    var value: T
    var next: Node<T>?
    weak var previous: Node<T>?

    init(value: T) {
        self.value = value
    }
}
```
- 노드 클래스를 제네릭 타입으로 변경했다
- 이번에는 링크드리스트에 제네릭 타입을 적용해보자
```swift
// 1. Change the declaration of the Node class to take a generic type T
public class LinkedList<T> {
  // 2. Change the head and tail variables to be constrained to type T
  fileprivate var head: Node<T>?
  private var tail: Node<T>?

  public var isEmpty: Bool {
    return head == nil
  }
  
  // 3. Change the return type to be a node constrained to type T
  public var first: Node<T>? {
    return head
  }

  // 4. Change the return type to be a node constrained to type T
  public var last: Node<T>? {
    return tail
  }

  // 5. Update the append function to take in a value of type T
  public func append(value: T) {
    let newNode = Node(value: value)
    if let tailNode = tail {
      newNode.previous = tailNode
      tailNode.next = newNode
    } else {
      head = newNode
    }
    tail = newNode
  }

  // 6. Update the nodeAt function to return a node constrained to type T
  public func nodeAt(index: Int) -> Node<T>? {
    if index >= 0 {
      var node = head
      var i = index
      while node != nil {
        if i == 0 { return node }
        i -= 1
        node = node!.next
      }
    }
    return nil
  }

  public func removeAll() {
    head = nil
    tail = nil
  }

  // 7. Update the parameter of the remove function to take a node of type T. Update the return value to type T.
  public func remove(node: Node<T>) -> T {
    let prev = node.previous
    let next = node.next

    if let prev = prev {
      prev.next = next
    } else {
      head = next
    }
    next?.previous = prev

    if next == nil {
      tail = prev
    }

    node.previous = nil
    node.next = nil
    
    return node.value
  }
}
```
- 다음 테스트 코드를 넣어 잘 동작하는지 확인해보자
```swift
let dogBreeds = LinkedList<String>()
dogBreeds.append(value: "Labrador")
dogBreeds.append(value: "Bulldog")
dogBreeds.append(value: "Beagle")
dogBreeds.append(value: "Husky")

let numbers = LinkedList<Int>()
numbers.append(value: 5)
numbers.append(value: 10)
```