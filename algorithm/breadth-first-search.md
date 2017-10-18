# 너비 우선탐색
- 최단 경로 찾기

## 그래프
- 항목들이 서로 어떻게 연결되어 있는지 모형화하는 방법
- 정점(node)과 간선(edge)으로 이루어짐

## 최단 경로 찾기
- 여러분의 네트워크에 망고 판매상이 있는가? (나올 때까지 친구의 친구의 친구를 뒤진다)
- 누가 가장 가까운 망고 판매상인가? (목록에 더해지는 순서대로 사람을 탐색한다 = 따라서 큐를 사용해야 한다)
- 실행시간: O(정점수+간선수)

```swift
func breadthFirstSearch(_ graph: Graph, source: Node) -> [String] {
  
  //탐색할 큐 생성
  var queue = Queue<Node>()
  queue.enqueue(source)
  
  //탐색한 사람을 담아놓을 배열
  var nodesExplored = [source.label]
  source.visited = true

  //탐색할 큐가 nil이 될 때까지 반복
  while let node = queue.dequeue() {
    for edge in node.neighbors {
      let neighborNode = edge.neighbor

      //체크 안된 이웃이면
      if !neighborNode.visited {
        
        //큐에 넣어 놓고
        queue.enqueue(neighborNode)

        //체크했다고 표시
        neighborNode.visited = true
        nodesExplored.append(neighborNode.label)
      }
    }
  }

  return nodesExplored
}
```