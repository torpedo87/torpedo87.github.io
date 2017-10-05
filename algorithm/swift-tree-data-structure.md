# Swift Algorithm Club: Swift Tree Data Structure
> [Swift Algorithm Club: Swift Tree Data Structure](https://www.raywenderlich.com/138190/swift-algorithm-club-swift-tree-data-structure)를 번역하였습니다.

## tree 구조
- 다이어그램은 0 ~ 5 단계의 트리구조를 보여준다. root 는 0 단계이고 아래로 내려갈 수록 레벨이 1씩 증가한다
- tree는 다음과 같은 문제를 해결하는 데 도움을 준다
- 1. 사물간의 관계도 표현
- 2. 검색을 빠르고 효율적으로
- 3. 정렬된 리스트의 데이터 제공
- 4. 텍스트필드에서 접두어 매칭

## 용어

### 1. root
- 트리의 0단계의 노드를 나타낸다. 트리 구조의 시작접이다

### 2. node
- 트리구조의 데이터 블록이다. 노드가 갖는 데이터는 트리의 타입에 따라 다르다. root 또한 노드이다.

### 3. leaf
- 자식이 없는 노드로서 terminal node 로 불린다

## 스위프트로 트리 구현하기
- 제한이 없는 일반적인 트리를 구현해보자. 트리는 노드로 구성된다. 그러므로 노드 클래스부터 만들자.

```swift
class Node {

}
```

### 1. value
- 노드는 연관값을 갖는다. 간단하게 문자타입의 데이터를 다루는 트리를 만들자. 노드 클래스를 다음과 같이 추가하자

```swift
class Node {
    var value: String

    init(value: String) {
        self.value = value
    }
}
```

- 문자 타입의 value 프로퍼티를 선언했다. 또한 생성자를 만들어서 비옵셔널 프로퍼티를 초기화하도록 했다.

### 2. children
- 노드는 자식 리스트를 갖는다.

```swift
class Node {
    var value: String
    var children: [Node] = []

    init(value: String) {
        self.value = value
    }

}
```

- 자식을 노드가 담긴 배열로 선언했다. 각각의 자식은 노드이며 현재 노드보다 1단계식 증가한다

### 3. parent
- 노드가 부모노드 또한 포함하면 더 용이하다. 자식은 기존 노드 하위의 노드들이며 부모는 기존 노드 상위의 노드이다. 노드는 하나의 부모만 가질 수 있으나 자식은 여럿 가질 수 있다.

```swift
class Node {
    var value: String
    var children: [Node] = []
    weak var parent: Node?

    init(value: String) {
        self.value = value
    }
}
```

- 부모는 옵셔널이다. 왜냐하면 root 노드인 경우 부모가 없기 때문이다. 또한 순환참조를 피하기 위해 약한 참조를 했다

### 4. insertion
- 트리에 자식노드를 삽입하기 위해 토드 클래스에 add(child:) 메소드를 추가하자

```swift
class Node {
    var value: String
    var children: [Node] = []
    weak var parent: Node?

    init(value: String) {
        self.value = value
    }

    func add(child: Node) {
        children.append(child)
        child.parent = self
    }
}
```

- 클래스 밖에 다음 테스트코드를 추가해보자

```swift
let beverages = Node(value: "beverages")
let hotBeverages = Node(value: "hot")
let coldBeverages = Node(value: "cold")

beverages.add(child: hotBeverages)
beverages.add(child: coldBeverages)
```

- beverages 자식에 hot, cold 가 들어가는 상하관계가 나타난다

## 트리 출력하기

```swift
extension Node: CustomStringConvertible {
    var description: String {

        var text = "\(value)"

        if !children.isEmpty {
            text += " {" + children.map { $0.description }.joined(seperator: ", ") + "} "
        }

        return text
    }
}
```

- 1. 노드클래스의 extension을 선언해서 프로토콜을 적용했다. description이라는 문자열 타입의 연산프로퍼티를 선언했다
- 2. description은 읽기전용이며 문자열을 반환한다
- 3. text 변수를 선언하여 전체 문자열을 담을 것이다. 처음에는 노드의 값만 담고 있다
- 4. 노드의 값 이외에 자식계통을 출력하기 위해서 재귀적으로 자식의 description을 담는다. 자식관계를 구분하기 위해 괄호를 넣는다
- 이제 트리를 출력해보자

```swift
"beverages {hot {tea {black, green, chai} , coffee, cocoa} , cold {soda {ginger ale, bitter lemon} , milk} } \n"
```

- 위의 map 고차함수 대신에 다음으로 대체해도 된다

```swift
if !children.isEmpty {
    text += " {"
    
    for child in children {

        if children.last?.value != child.value {
            text += child.description + ", "
        } else {
            text += child.description
        }
    }

    text += "} "
}
```

## 검색
- 트리가 특정 값을 갖는지 확인하기 위해 노드 클래스를 사용할 수 있다.

```swift
extension Node {

    func search(value: String) -> Node? {

        if value == self.value {
            return self
        }

        for child in children {

            if let found = child.search(value: value) {
                return found
            }
        }

        return nil
    }
}
```

- 1. search 메소드는 트리에 값이 있는지 찾는 것이다. 있으면 그 값을 갖는 노드를 반환한다. 없으면 nil을 반환한다
- 2. 현재 노드의 값이 그 값이면 self를 반환한다
- 3. 자식배열을 돌면서 각 자식의 search 메소드를 호출해서 재귀적으로 자식을 찾도록 한다. 만약 찾으면 노드를 반환한다
- 4. 없으면 nil을 반환한다

## 제네릭으로 개선 하기
- 모든 타입에 적용하기 위해 제네릭을 사용하여 개선해보자

```swift
// 1. 
class Node<T> {
  // 2. 
  var value: T
  weak var parent: Node?
  // 3.
  var children: [Node] = []
  
  // 4.
  init(value: T) {
    self.value = value
  }
  
  // 5. 
  func add(child: Node) {
    children.append(child)
    child.parent = self
  }
}
```

```swift
// 1. 
extension Node where T: Equatable {
  // 2. 
  func search(value: T) -> Node? {
    if value == self.value {
      return self
    }
    for child in children {
      if let found = child.search(value: value) {
        return found
      }
    }
    return nil
  }
}
```

- 1. search 메소드를 구현하기 위해서는 value 타입인 제네릭타입 T 가 Equatable을 준수해야 한다
- 2. value 타입은 제네릭타입이다

## 트리의 종류
- binary tree: 자식의 갯수를 최대 2로 제한
