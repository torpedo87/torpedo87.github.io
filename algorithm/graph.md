# 그래프
- 노드의 집합과 에지의 집합에 의해 정의된다
- 개체들 간의 이진관계를 표현
- 방향 그래프: 에지가 방향을 갖는다
- 가중 그래프: 에지가 가중치를 갖는다
- 노드간 경로가 있다: 연결되어 있다
- 노드간 에지 1개: 인접해 있다

## 그래프 표현방법

### 1. 인접행렬
- 정점의 개수가 n개인 그래프를 n * n 행렬로 표현
- i와 j 사이에 에지가 있으면 a(i,j) = 1 (가중치 그래프이면 가중치), 없으면 a(i,j) = 0
- 인접한 모든 노드 찾기: O(n)
- 특정 에지 찾기: O(1)

### 2. 인접리스트
- 정접 집합을 표현하는 하나의 배열
- 각 정점마다 인접한 정점들의 연결리스트
- 인접한 모든 노드 찾기: O(연결리스트의 길이)
- 특정 에지 찾기: O(연결리스트의 길이)

## 그래프 순회방법: 모든 노드들을 중복없이 방문하기

### 1. 너비우선 = 이진트리의 level-order 순회
- 동심원 형태로 탐색
- 방문했으면 check 표시
- 같은 단계의 노드들이 담긴 큐에서 하나씩 꺼내면서 해당 노드의 모든 인접 노드 중 방문하지 않은 노드들을 새로운 큐에 넣기를 반복
- 최단 경로의 길이를 구할 수 있다

### 2. 깊이우선 = 이진트리의 inorder 순회
- 인접한 모든 노드 중 아직 방문하지 않은 노드를 방문하다가 막히면 다시 되돌아가면서 방문하지 않은 노드 체크해서 방문
- recursion

## 방향그래프
- 