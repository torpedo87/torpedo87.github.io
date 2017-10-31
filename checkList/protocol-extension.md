# extension

- 모든 타입에 새로운 기능을 추가 가능, but 기존의 기능을 재정의 할 수는 없다
- 클래스의 상속은 수직확장이라면 extension은 수평확장
- mutating method를 만들어서 자기 자신을 변형시킬 수도 있다

```swift
extension UIButton {
  
  func wiggle() {
    let wiggleAnimation = CABasicAnimation(keyPath: "wiggle")
    wiggleAnimation.duration = 0.05
    wiggleAnimation.repeatCount = 5
    wiggleAnimation.autoreverses = true
    wiggleAnimation.fromValue = CGPoint(x: self.center.x + 4.0, y: self.center.y)
    wiggleAnimation.toValue = CGPoint(x: self.center.x - 4.0, y: self.center.y)
    layer.add(wiggleAnimation, forKey: "wiggle")
  }
  
}

```


# 프로토콜

- 프로토콜은 하나의 타입이다.
- 프로토콜이 또 다른 프로토콜을 준수할 수 있다
- 특정 역할을 수행하기 위한 to do list 
- 위임을 위한 프로토콜: 예) UITableViewDelegate

```swift
protocol ColorTransferDelegate {
  func userDidChoose(color: UIColor, withName colorName: String)
  
}


//server vc
var delegate: ColorTransferDelegate? = nil

if delegate != nil {
  delegate?.userDidChoose(color: color, withName: name)
}


//client vc
class ServerVC: UIViewController, ColorTransferDelegate {
  clientVC.delegate = self
  
  func userDidChoose(color: UIColor, withName colorName: String) {
    view.backgroundColor = color
  }
}

```
