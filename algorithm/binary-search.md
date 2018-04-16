# Binary Search

 ```swift

var list = [1, 3, 5, 7, 9, 10, 15]

func binarySearch(list: [Int], target: Int) -> Int? {

  var startIndex = 0
  var endIndex = list.count - 1
  var midIndex: Int = (endIndex + startIndex) / 2
  var midValue = list[midIndex]

  while startIndex <= endIndex {
    if midValue > target {
      endIndex = midIndex - 1
      midIndex = (endIndex + startIndex) / 2
      midValue = list[midIndex]
    } else if midValue < target {
      startIndex = midIndex + 1
      midIndex = (endIndex + startIndex) / 2
      midValue = list[midIndex]
    } else {
      return midIndex
    }
  }
  return nil
}

binarySearch(list: list, target: 4)

 ```



