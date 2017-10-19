# 다익스트라 알고리즘 (양수 가중치 그래프 사용)
- 가장 빠른 길 찾기
- 그래프의 구간이 모두 일정한 것이 아니라 걸리는 시간이 다르므로 구간의 가중치를 고려한다
- 방향성 비순환 그래프 이어야 한다

## 단계
- 가격이 가장 싼 정점 찾기
- 이웃 정점에 대해 현재 가격보다 더 싼 경로가 있는지 확인해서 있으면 가격 수정
- 모든 정점에 대해 반복
- 최종 경로 계산