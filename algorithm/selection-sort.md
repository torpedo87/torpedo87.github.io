# 메모리가 동작하는 방법
- 메모리에 여러개의 원소를 저장해야 할 때 사용할 방법이 필요하다

## 1. 배열(임의 접근)
- 추가하기 어렵다: 모든 원소의 위치를 바꾸어야 한다, O(n)
- 읽기 쉽다: index로 임의의 원소를 찾기 용이, O(1)

## 2. 연결리스트(순차 접근)
- 추가하기 쉬움: 원소가 가리키는 참조만 바꾸면 됨, O(1)
- 읽기 어렵다: index가 없는 대신 앞의 원소가 그 다음 원소의 주소값을 가지므로 임의의 원소를 찾기 어렵다, O(n)

# 선택정렬
- 느리다
- 배열을 작은 정수부터 큰 정수 순서로 정렬하기
- 가장 작은 수를 찾아서 제일 왼쪽 원소와 바꿔치기를 반복
- O(n^2)

```swift
//제일 작은 수 찾는데 n번 돌기
func findSmallest(arr: [Int]) -> Int {
    let smallest = arr[0]
    let smallestIndex = 0

    for i in 1...arr.count - 1 {
        if arr[i] < smallest {
            smallest = arr[i]
            smallestIndex = i
        }
    }

    return smallestIndex
}

//정렬하는데 n번 돌기
func selectionSort(arr: [Int]) -> Int {
    let newArr = [Int]()
    
    for i in arr {
        let smallestIndex = findSmallest(arr)
        newArr.append(arr[smallestIndex])
        arr.remove(at: smallestIndex)
    }
    return newArr
}
```
