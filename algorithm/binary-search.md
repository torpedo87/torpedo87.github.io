# 이진탐색
 - 로그시간, O(log n)

## 실행시간
- N개의 원소 중에서 특정 원소를 탐색할 때 단순 탐색(선형시간)을 사용하면 최대 N번이 소요되지만, 이진탐색(로그시간)을 사용하면 최대 logN 번이 소요된다 (여기서 로그의 밑은 2이다)

## 전제조건
- 정렬이 되어 있어야 한다.
 
## 반복문으로 구현

 ```swift
func binarySearch<T: Comparable>(_ arr: [T], target: T) -> Int? {
    var low = 0
    var high = arr.count - 1

   while low <= high {
       let mid = (low + high) / 2
       if arr[mid] == target {
           return mid
       }
       if arr[mid] > target {
           high = mid - 1
       }
       else {
           low = mid + 1
       }
   }
    return nil
}
 ```

## 빅오 표기법
- O(연산횟수)
- 크기에 따른 최악의 실행횟수를 나타낸다
- 실행횟수: log n < n < n * log n < n*n < n!
