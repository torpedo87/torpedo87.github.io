# Swift Algorithm Club: Swift Merge Sort
> [Swift Algorithm Club: Swift Merge Sort](https://www.raywenderlich.com/154256/swift-algorithm-club-swift-merge-sort)를 번역하였습니다.

## 시작하기
- merge sort의 아이디어는 먼저 쪼개어 나눈 후 나중에 합치는 것이다. 포커 카드가 있다고 가정하자. 
- 1. 반으로 나눠서 정렬되지 않은 상태의 두 묶음을 갖는다
- 2. 각 묶음을 계속 반으로 나눠서 각 묶음당 1장이 될 때까지 나눈다
- 3. 이제 각 묶음을 두 묶음씩 합쳐서 정렬해나가자

## Swift merge sort
- 빠르다
- 다음 코드를 추가하자

```swift
let array = [7, 2, 6, 3, 9]

func mergeSort(_ array: [Int]) -> [Int] {

}
```

- 정렬되지 않은 정수 배열이 있다. 정렬된 배열을 반환하도록 함수를 구현해보자

### 1. 쪼개기
- 배열을 반으로 쪼개자.

```swift
func mergeSort(_ array: [Int]) -> [Int] {

    let middleIndex = array.count / 2

    let leftArray = Array(array[0..<middleIndex])

    let rightArray = Array(array[middleIndex..<array.count])
}
```

- 1. 배열 원소갯수를 2로 나누어 중간 인덱스 찾기
- 2. 중간 인덱스를 기준으로 왼쪽배열과 오른쪽 배열로 나누기

### 2. 계속 쪼개기
- 더이상 쪼갤 수 없을 때 까지 위의 활동을 계속해야 한다. 재귀함수를 사용하자

```swift
func mergeSort(_ array: [Int]) -> [Int] {

    guard array.count > 1 else { return array }

    let middleIndex = array.count / 2

    let leftArray = mergeSort(Array(array[0..<middleIndex]))

    let rightArray = mergeSort(Array(array[middleIndex..<array.count]))
}
```

- 1. 재귀를 적용하기 위해서는 기본 케이스와 출구 조건이 필요하다. 위의 경우 출구 조건은 배열의 원소가 1개인 경우이다.
- 2. 나뉘어진 왼쪽 오른쪽 배열에 각각 mergeSort를 호출한다. 반으로 쪼개지자마자 다시 쪼개질 것이다.

### 3. 합치기
- 왼쪽 오른쪽 배열을 합치자. 이를 위해 따로 merge() 함수를 만들자.
- merge() 함수의 기능은 두개의 정렬된 배열을 받아서 합치는 것이다. 출력값도 정렬된 상태를 유지해야 한다.

```swift
func merge(_ left: [Int], _ right: [Int]) -> [Int] {

    var leftIndex = 0
    var rightIndex = 0

    var orderdArray: [Int] = []

    return orderedArray
}
```

- 함수의 틀만 잡았다
- 1. leftIndex와 rightIndex 변수를 선언했다. 이를 통해 두 배열의 진행상황을 추적할 것이다
- 2. orderedArray는 합쳐진 배열을 담을 변수이다
- 왼쪽 배열과 오른쪽 배열의 원소를 하나씩 비교해서 새 배열에 추가하자. 각 단계에서 비교를 거쳐서 작은 원소를 새 배열에 넣는다. 

```swift
func merge(_ left: [Int], _ right: [Int]) -> [Int] {

    var leftIndex = 0
    var rightIndex = 0

    var orderedArray: [Int] = []

    //한 배열이 끝에 도달할때까지 두 배열 비교해서 넣기
    while leftIndex < left.count && rightIndex < right.count {
        let leftElement = left[leftIndex]
        let rightElement = right[rightIndex]

        if leftElement < rightElement {
            orderedArray.append(leftElement)
            leftIndex += 1
        } else if leftElement > rightElement {
            orderedArray.append(rightElement)
            rightIndex += 1
        } else {
            orderedArray.append(leftElement)
            leftIndex += 1
            orderedArray.append(rightElement)
            rightIndex += 1
        }
    }

    //남은 배열 넣기
    while leftIndex < left.count {
        orderedArray.append(left[leftIndex])
        leftIndex += 1
    }

    //남은 배열 넣기
    while rightIndex < right.count {
        orderdArray.append(right[rightIndex])
        rightIndex += 1
    }

    return orderedArray
}
```

- 1. 왼쪽배열 원소와 오른쪽배열 원소를 순서대로 비교한다. 만약 두 배열 중 어느 한 배열의 끝에 도달하면 더이상 비교할 게 없다
- 2. 첫 while 문을 빠져나오면 왼쪽 또는 오른쪽 배열이 빈 배열임을 보장한다. 두 배열이 정렬된 상태이므로 합치고 남은 배열은 새 배열에 있는 원소보다 같거나 크다. 남은 배열은 따라서 비교없이 그냥 새 배열에 추가하면 된다

## 마무리

```swift
func mergeSort(_ array: [Int]) -> [Int] {
    guard array.count > 1 else { return array }

    let middleIndex = array.count / 2

    let leftArray = mergeSort(Array(array[0..<middelIndex]))
    let rightArray = mergeSort(Array(array[middleIndex..<array.count]))

    return merge(leftArray, rightArray)
}
```

- 1. 우리의 전략은 일단 쪼갠 후 나중에 합치는 것이다
- 2. 재귀적으로 배열을 반으로 쪼개는 메소드, 정렬된 상태의 두 배열을 하나로 합치는 메소드가 필요하다
- 3. 정렬된 두 배열을 받아서 합친 후 하나의 정렬된 배열을 반환한다

## 제네릭 적용하기
- 정수 타입의 배열만 확인했다면, 이제 모든 데이터 타입을 다룰 수 있는 정렬로 개선해보자. 제네릭을 사용하면 쉽게 구현할 수 있다. 코드에서 Int 부분을 찾아 T로 대체하고 함수 이름에 <T: Comparable>을 추가한다

```swift
func mergeSort<T: Comparable>(_ array: [T]) -> [T] {
  guard array.count > 1 else { return array }

  let middleIndex = array.count / 2
  
  let leftArray = mergeSort(Array(array[0..<middleIndex]))
  let rightArray = mergeSort(Array(array[middleIndex..<array.count]))
  
  return merge(leftArray, rightArray)
}

func merge<T: Comparable>(_ left: [T], _ right: [T]) -> [T] {
  var leftIndex = 0
  var rightIndex = 0

  var orderedArray: [T] = []
  
  while leftIndex < left.count && rightIndex < right.count {
    let leftElement = left[leftIndex]
    let rightElement = right[rightIndex]

    if leftElement < rightElement {
      orderedArray.append(leftElement)
      leftIndex += 1
    } else if leftElement > rightElement {
      orderedArray.append(rightElement)
      rightIndex += 1
    } else { 
      orderedArray.append(leftElement)
      leftIndex += 1
      orderedArray.append(rightElement)
      rightIndex += 1
    }
  }

  while leftIndex < left.count {
    orderedArray.append(left[leftIndex])
    leftIndex += 1
  }

  while rightIndex < right.count {
    orderedArray.append(right[rightIndex])
    rightIndex += 1
  }
  
  return orderedArray
}
```

- 정렬하고자 하는 원소가 Comparable 하다면 <, > 연산자를 사용할 수 있으므로 merge sort를 사용할 수 있다.