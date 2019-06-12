[Swift Interview Questions and Answers - raywenderlich](https://www.raywenderlich.com/762435-swift-interview-questions-and-answers)



## 初级问题



> Question #1



```swift
struct Tutorial {
  var difficulty = 1
}

var tutorial1 = Tutorial()
var tutorial2 = tutorial1
tutorial2.difficulty = 2
```

**`tutorial1.difficulty`** 和 **`tutorial2.difficulty`** 的值分别是什么？如果 **`Tutorial`** 是一个 class，而不是结构体会有差别吗？为什么？



**答案**:

**`tutorial1.difficulty`** 和 **`tutorial2.difficulty`** 值分别是 **`1`** 和 **`2`**。在swift中结构体是值类型。复制是通过值而不是引用。

如果 **`Tutorial`** 是一个类，而不是结构体，则 **`tutorial1.difficulty`** 和 **`tutorial2.difficulty`** 值都是 **`2`**，因为swift中类是引用类型。



> Question 2#

你已经使用 **`var`** 声明了 **`view1`**, 然后使用 **`let`** 声明了 **`view2`**. 这2者的差别是什么？最后一行能否编译？

```swift
import UIKit

var view1 = UIView()
view1.alpha = 0.5

let view2 = UIView()
view2.alpha = 0.5 // 这一行能编译吗？
```



**答案**:

最后一行可以编译。 **`view1`** 是一个变量，可以重新给它赋一个 **`UIView`** 实例。而使用 **`let`**， 只能赋一次值，因此下面代码不能编译：

```swift
view2 = view1 // 错误：view2 不可变
```

**`UIView`** 是一个引用类型， 可以改变它的属性。