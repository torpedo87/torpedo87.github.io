# selection sort
- n개의 배열에서 가장 큰 수를 찾아서 배열의 마지막 원소와 바꿔치기
- n-1 배열에서 가장 큰 수 를 찾아서 이 배열의 마지막 원소와 바꿔치기
- 반복
- O(n^2)

# bubble sort
- 물고기 잡을 때 물고기 몰아가기
- 최대값을 찾아서 배열의 제일 마지막 위치로 이동시키는 방법이 selection sort와 다르다
- 배열의 첫번째 원소를 순서대로 비교해서 비교한 수 보다 크면 서로 위치를 바꾸기
- 다음 원소로 이동해서 반복
- O(n^2)

# insertion sort
- selection sort 나 bubble sort의 절반정도 소요된다
- n 크기의 배열이 있을 때 첫 원소부터 n 크기의 배열을 순서대로 정렬해나가는 방법
- 즉 정렬된 앞의 배열에 그 다음 원소를 적절한 위치에 삽입하는 방법
- 최악의 경우 O(n^2)
- 최선의 경우 O(n)

---

# merge sort
- 분할정복법: 작은 크기의 '동일한' 문제로 분할, 합병이 중요
- 데이터가 저장된 배열을 절반으로 나눔
- 각각을 순환적을 정렬
- 정렬된 두 배열을 합쳐 전체를 정렬 (이 때 합 친 원소를 담을 새로운 배열이 필요한 단점)
- 정렬된 두 배열을 합칠 때 두 배열의 첫번째 원소를 비교해서 그 중 작은 것을 새배열의 왼쪽에 넣기
- O(n * logn)

# quick sort
- 분할정복법: 피봇을 기준으로 분리하는 파티션이 중요
- pivot을 선정(주로 배열의 제일 마지막 원소로 선정)해서 피봇보다 작은 그룹과 피봇보다 큰 그룹으로 나눈다
- 각 그룹을 재귀 호출해서 정렬
- 합칠 필요 없음
- 최악의 경우: 피봇이 최대값 또는 최소값인 경우 O(n^2)
- 최선의 경우: 피봇이 중값값일 경우. O(n * logn)
- 평균 시간복잡도 역시 빠르다: O(n * logn)

# heap sort
- 최악: O(n * logn)
- 정렬하기 위해 추가 배열 필요없음
- binary heap 자료구조를 사용

## heap
- complete binary tree: 계층적 관계를 나타내는 트리에서 자식노드를 최대 2개를 가질 수 있는 트리이며, 마지막 레벨 이외의 레벨 노드가 꽉 차있고, 마지막 레벨의 오른쪽에서부터 연속된 노드가 비어있을 수도 있다.
- heqp property를 만족한다: 부모가 자식보다 크거나 같다(max heap). 또는 부모는 자식보다 작거나 같다(min heap)
- 최대값은 루트
- 1차원 배열로 표현 가능하며 인덱스를 통해 계층관계 정보를 유지할 수 있다

## max heapify
- complete birary tree 에 임의의 루트를 추가
- 새로 추가된 루트가 두 자식중 더 큰 쪽이 루트보다 크다면 자리 교체
- 만족시킬 때까지 순환
- O(log n),  n = 노드 갯수, log n = 트리의 높이

## heap 정렬 순서
- 배열을 트리 모양으로 만들기
- 맨 밑에서 잎이 아닌 노드를 포함하는 가장 작은 서브 트리부터 heapify 하기
- 최대값인 루트를 배열의 맨 마지막 원소와 자리 바꾸기
- 루트가 들어간 맨 마지막 리프노드를 없애고 나면 n-1 크기의 힙이 되고, 새로운 루트를 제외하고는 힙을 만족하는 2개의 서브트리가 되므로 max heapify 를 한다
- 순환
- O(n * logn)

---

# 최대 우선순위 큐
- max heap을 이용하여 구현
- insert, extractMax 연산을 지원
- insert: 새 원소를 힙의 마지막 자리에 넣은 후, 부모 노드와 비교해서 제 자리 찾아가기 O(log n)
- extractMax: 힙을 유지하기 위해 맨 마지막 노드와 루트의 자리를 바꿔서 최대값을 삭제한 후, max heapify 한다. O(log n)

---

# comparison sort
- 크기 관계에 따라서 정렬하는 알고리즘
- 최악의 시간복잡도는 desicion tree의 높이이며 최소 높이는 log n
- 최악의 경우가 O(n * log n)보다 더 좋은 알고리즘은 존재하지 않는다

# non comparison sort
- 사전지식을 이용해서 정렬하는 알고리즘 (bucket, radix)

## 선형시간 정렬 알고리즘
- O(n)

### 1. counting sort
- 사전지식이 존재
- 예) n명의 학생들의 시험점수를 정렬하라 (사전지식: 점수가 100 이하의 양의 정수)
- 모든 값에 대해 몇개씩인지 카운팅
- 누적 개수 구하기
- 기존 배열의 뒤에서부터 원소의 누적갯수는 곧 해당 원소 이하의 갯수가 누적갯수이므로 새 배열에 담을 때 누적갯수 인덱스까지의 최대값은 그 원소가 되므로 해당 인덱스에 원소 넣기
- 해당 인덱스에 자리가 찼으면 그 앞의 인덱스 자리에 넣기
- k진법의 n개의 정수: O(n+k)

### 2. radix sort
- n개의 3자리 정수를 작은 수부터 정렬하라
- 가장 낮은 자리수부터 정렬(각 자릿수 정렬할 때마다 counting sort 사용)
- stable 해야 한다: 그래야 두번째 자리수를 기준으로 정렬했을 때 첫번째와 두번째 자리수 를 모두 기준으로 정렬한 결과가 나온다
- d자리 정수: O(d(n+k))

# 시간복잡도 비교
- O(n^2) : bubble, insertion, selection
- 최악 O(n^2), 평균 O(n * log n) : quick
- O(n * log n) : merge, heap
- O(d(n+k)) : radix
- O(n+k) : counting

