# 퀵 정렬

## 분할 정복
- 토지를 균등하게 나눌 수 있는 가장 큰 정사각형 찾기
- 기본단계 : 토지의 가로 세로 중 작은 쪽을 기준으로 정사각형을 만들기 (가장 간단한 경우로 가정해보기)
- 재귀단계 : 남은 땅에 대해 다시 재귀 호출 (문제의 사이즈를 줄이기)
- 반복문을 재귀적으로 바꿀 수 있다(함수형 프로그래밍의 예)
- 재귀함수는 함수 안에서 또 자신을 호출하므로 내부에서부터 풀어야 한다. 따라서 기본단계를 만나기 전까지 완료되지 않으므로 상태를 추적한다
- 기본단계를 만나기 전까지 부분적으로 실행된 함수의 호출 상태를 모두 저장한다.

## 퀵 정렬
- 빠르다
- 선택 정렬 O(n*n)보다 훨씬 빠름 O(n * log n)

```swift
func quicksort<T: Comparable>(_ arr: [T]) -> [T] {

  guard arr.count > 1 else { return arr }

  //기준원소
  let pivot = arr[arr.count/2]

  //분할
  let less = a.filter { $0 < pivot }
  let equal = a.filter { $0 == pivot }
  let greater = a.filter { $0 > pivot }
  
  //합치기
  return quicksort(less) + equal + quicksort(greater)
}
```
