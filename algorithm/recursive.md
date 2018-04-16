# Recursive

```swift

func countdown(number: Int) {
  print(number)
  if number <= 0 {
    return
  } else {
    countdown(number: number - 1)
  }
}

countdown(number: 3)

func factorial(i: Int) -> Int {
  if i == 1 {
    return 1
  } else {
    return i * factorial(i: i-1)
  }
}

factorial(i: 4)


```