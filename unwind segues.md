内容：

1. The difference between unwind segues and other segues in an Xcode storyboard
2. Why should you use unwind segues instead of dismissing or poppong view controllers?
3. The typical scenario for unwind segues: modal presentation
4. Dismissing a modal view controller with an unwind segue
5. Jumping back more than one view controller in a navigation stack
6. Passing data to a previous view controller in an unwind segue
7. Performing an unwind segue programmatically
8. The flexibility of unwind segues

至于 **`unwind segues`** 怎么翻译，暂时不是很清楚😆。



## 1. 在SB中unwind segues 和其它segues的区别

在iOS storyboard中有**`4`**种类型的segues:

- 关系型segues：添加vc到容器(**containers**)中，比如navigation或者tab bar controllers
- 嵌入式segues: 将一个vc插入到一个自定义的vc中，从而使后者在不需要额外的代码的情况下成为一个自定义容器
- 动作型segues: 触发一个向前的过度，从一个vc导航到另一个vc
- unwind segues: 回滚动作性segues,返回到先前的vc

定义一个unwind segues很简单，但是实际上unwind segues很复杂：

- Unwind segues不限定于最后一次过度，它可以用于任何数量vc之间的跳转
- 当动作型segue和目标vc关联时，将目标vc为unwind segue是一个很复杂的过程
- 在sb中连接unwind segue不如连接其它segues简单，连接其它类型的segues,只需要按住 **`ctrl`**, 然后拖拽到目标源即可，但是连接unwind segue则很复杂。

## 2.为什么要使用unwind segues，而不是显示（pop）或隐藏(dismiss) vcs?

既然设置unwind segues很复杂，为什么我们还要去设置它呢？

毕竟下面方式都是一行代码就完事了：

- 隐藏一个模态式vc，可以调用 **`dismiss(animated:completion:)`**
- 在一个导航controller中返回，可以使用 **`popViewController(animated:)`**

为什么还有使用unwind segues呢？ 简短回答是：**unwind segues不仅仅是简单的返回操作**。

有很多合理的理由使用unwind segues：

1. 当使用sb时，保持一致性很好，比如使用相同的机制前进和后退。**Unwind segues 是action segues的对应项**。 sb和segues的意义是从vc中移除导航代码。因此，当你使用segues向前导航时，你也应该使用segues返回
2. **Unwind segues比代码更加的泛型**。 比如当你在某个时候同时有几个modal vcs在屏幕上时，你必须在正确的vc上调用 **`dismiss(animated:completion:)`**才能返回你想要的位置。 使用 **`popViewController(animated:)`**的另一个问题是，你必须经过vc层级(climb the view controller hierarchy)在导航控制器上调用这个方法
3. 当一个vc有多种方式可以到达时，**你需要决定使用哪一种方式返回**，unwind segues从vcs中移除了导航代码。 我们经常在sb中通过不同的导航路径访问一个vc。有一些可能使用modal形式，而有一些可能使用导航控制器，这种情况下，返回时，我们需要调用正确的方法，这意味着在当前vc中我们需要些更多的代码去做决定。
4. **有时，我们需要返回不止一层vc**。在复杂的导航结构中，我们可能在用户操作之后需要往后返回基层。Unwind segues会省很多事



## 3.使用unwind segues常见场景： Modal显示

