**`UIActivityIndicatorView`** 是一个比较简单的视图，多用于表示应用正在处理某个任务中，比如网络请求。

其定义：

```swift
import Foundation
import UIKit
import _SwiftUIKitOverlayShims

extension UIActivityIndicatorView {

    
    public enum Style : Int {

        // 大小 CGSize(width: 37, height: 37)
        case whiteLarge

        // 大小 CGSize(width: 22, height: 22)
        case white

      	// 大小 CGSize(width: 22, height: 22)
        case gray
    }
}

@available(iOS 2.0, *)
open class UIActivityIndicatorView : UIView, NSCoding {

    
    public init(style: UIActivityIndicatorView.Style) // sizes the view according to the style

    public init(frame: CGRect)

    public init(coder: NSCoder)

    // 默认样式是 UIActivityIndicatorView.Style.white
    open var style: UIActivityIndicatorView.Style // default is UIActivityIndicatorViewStyleWhite

    open var hidesWhenStopped: Bool // default is YES. calls -setHidden when animating gets set to NO

    
    @available(iOS 5.0, *)
    // 定义转动菊花颜色
    open var color: UIColor!

    // 开始动画
    open func startAnimating()

    // 关闭动画
    open func stopAnimating()

    // 是否正在动画
    open var isAnimating: Bool { get }
}

```

另外还可以给indicator设置一个 **`backgroundColor`**：

```swift
let v = UIActivityIndicatorView(style: .whiteLarge)
v.color = .yellow
Dispatch.main.async {
  v.backgroundColor = UIColor(white: 0.2, alpha: 0.6)
}
v.layer.cornerRadius = 10 // 给背景的矩形添加一个圆角
self.view.addSubview(v)
v.startAnimating() // 开始动画
```



如果是网络请求，还可以在状态栏上添加一个网络活动指示器，可以设置 **`UIApplication`** 的 **`isNetworkActivityIndicatorVisible`** 为 **`true`**, 当网络活动完成后，再将其设置为 **`false`**。



**活动指示器是一个比较简单的组件，我们不能改变其绘制方式**。一般要创建自定义活动指示器有2种方式：

1. 使用一个 **`UIImageView`**, 包含一个动画图片（比如gif）

2. 使用 **`CAReplicatorLayer`** 拷贝多个子图层，然后将子图层进行动画，这是一种很常见的方法（**`UIActivityIndicatorView`** 实际也是通过这种方式实现的）

   ```swift
   import UIKit
   
   class ViewController: UIViewController {
     override func viewDidLoad() {
       super.viewDidLoad()
     }
     
     override func viewDidAppear(_ animated: Bool) {
       super.viewDidAppear(animated)
       
       let lay = CAReplicatorLayer()
       lay.frame = CGRect(x: 0, y: 0, width: 100, height: 20)
       
       // 将对这个图层进行拷贝
       let bar = CALayer()
       bar.frame = CGRect(x: 0, y: 0, width: 10, height: 20)
       bar.backgroundColor = UIColor.red.cgColor
       lay.addSublayer(bar) // 将其添加到可复制图层中
       lay.instanceCount = 5 // 复制5个
       // bar图层之间的间距是 20
       lay.instanceTransform = CATransform3DMakeTranslation(20, 0, 0)
       // 对图层透明度进行动画
       let anim = CABasicAnimation(keyPath: #keyPath(CALayer.opacity))
       anim.fromValue = 1.0
       anim.toValue = 0.2
       anim.duration = 1
       anim.repeatCount = .greatestFiniteMagnitude
       bar.add(anim, forKey: nil)
       lay.instanceDelay = anim.duration / Double(lay.instanceCount)
       
       self.view.layer.addSublayer(lay)
       lay.position = CGPoint(x: self.view.layer.bounds.midX, y: self.view.layer.bounds.midY)
     }
   }
   ```


