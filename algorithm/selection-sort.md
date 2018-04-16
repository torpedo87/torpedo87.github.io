# Selection Sort

```swift

var arr = [1, 3, 8, 4, 2, 7, 13, 6]

func getMinIndex(target: [Int]) -> Int {
  
  var minIndex = 0
  var min = target[minIndex]
  for i in 0..<target.count {
    if target[i] < min {
      minIndex = i
      min = target[minIndex]
    }
  }
  return minIndex
}

func selectionSort(target: [Int]) -> [Int] {
  var tempArr = target
  var resultArr = [Int]()
  for _ in 0..<target.count {
    let minIndex = getMinIndex(target: tempArr)
    resultArr.append(tempArr.remove(at: minIndex))
  }
  return resultArr
}

selectionSort(target: arr)

```
