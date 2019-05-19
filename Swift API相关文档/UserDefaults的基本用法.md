这个工具类的作用类似浏览器中的 **`localStorage`**, 但是个人感觉使用起来没有localStorage方便(主要还是web端可以直接存储json字符串的缘故吧)。

一般iOS应用都会在应用启动时加载已存储的少量的用户设置和偏好，这些数据都是提前存好的，对于这种类型数据一般使用 **`UserDefaults`** 进行存储。



可以存储的类型有：数字（Int, Float, Double），字符串，布尔值，数组，字典，日期， Data, URL等。具体支持哪些类型，可以参考官方文档

- [UserDefaults - apple开发者文档](https://developer.apple.com/documentation/foundation/userdefaults)

如果数据量比较大（比如大于100kb的数据量），如果使用 **`UserDefaults`**会影响应用加载速度，一般会使用 **`NSKeyedArchiver/NSkeyedUNarchiver`** 进行归档和解档。



用法：

- **`set(_:forKey:String)`**: 设置某个值
- **`object(forKey:String)->Any? | url(forKey:String)-> URL?`** 等等 读取值

```swift
let defaults = UserDefaults.standard

// 存储
// 数字
defaults.set(20, forKey: "Age")
defaults.set(CGFloat.pi, "PI")

// 字符串
defaults.set("James", forKey: "name")

// 布尔值
defaults.set(true, forKey: "isMale")

// 日期
defaults.set(Date(), forKey: "timestamp")

// 数组
let arr = ["apple", "banana"]
defaults.set(arr, forKey: "fruit")

// 字典
let info = ["country": "china", "name": "james"]
defaults.set(info, forKey: "INFO")
```

上面的 **`forKey`** 是任意的唯一字符串，**但是读取相应值时需要对应的字符串值**

```swift
// 读取
let defaults = UserDefaults.standard

// 数字的读取 defaults.integer | defaults.float | defaults.double
defaults.integer(forKey: "Age")
defaults.double(forKey: "PI")

// 字符串读取
defaults.string(forKey: "name")

// 布尔值读取
defaults.bool(forKey: "isMale")

// 对 Date, 数组, 字典的读取，一般使用
// defaluts.object(forKey:) -> Any?
// 因为数组和字典返回类型分别是 [Any]? , [String: Any]?
// 而object返回的是 Any?
// 可以自定义转换的类型
defaults.object(forKey: "timestamp") as? Double ?? 0
defaults.object(forKey: "fruit") as? [String] ?? [String]()
// 将获取的字典转换为[String:String]类型
// 如果获取的字典为nil，则给一个默认字符串字典
defaults.object(forKey: "INFO") as? [String:String] ?? [String:String]()
```



> 关于默认值

使用 **`UserDefaults`** 读取时，如果key不存在，则会返回一个默认值：

- **`integer(forKey:)`**: 如果key存在则返回一个数字类型，如果不存在则返回 **`0`**
- **`bool(forKey:)`**: 如果key存在则返回一个布尔值，如果不存在则返回 **`false`**
- **`float(forKey:) | double(forKey:)`**: 如果key存在则返回一个浮点数，如果不存在则返回 **`0.0`**
- **`object(forKey)`**: 会返回 **`AnyObject?`**, 因此你需要自定义转换为你想要的返回类型，比如上面的 **`UserDefaults.standard.object(forKey: "INFO")`**, 自定义返回类型为 **`[String: String]`**, 如果 **`INFO`** key 不存在，则给一个默认的空字符串字典 **`[String:String]()`**





> 关于大量数据或者数据结构比较复杂的存储

对于大量数据一般使用数据库进行存储：

- SQLite
- fmdb
- realm
- Core Data

对于结构比较复杂的数据存储，一般会涉及到下面知识：

- **`NSKeyedArchiver & NSkeyedUnarchiver`**： **如果对象是一个属于使用了NSCoding协议的Cocoa类**， 可以将数据进行归档（使用NSKeyedArchiver），将数据转换为 **`Data`** 类型，然后再使用 **`UserDefaults`** 进行存储
- **`Codable`** 协议：swift新的序列化方式，如果是自定义class，则可以让自定义的class遵循 Codable 协议，然后使用 **`PropertyListEncoder`** 对其进行归档
- **`NSCoding| NSSecureCoding`**：Cocoa Foundation框架中的大部分类采用这个协议，一般使用 **`NSKeyedArchiver`** 将对象转换为 **`NSData`**类型，使用时，再使用 **`NSKeyedUnarchiver`** 将其还原
- **`JSONEncoder & JSONDecoder`**

关于这些内容后面再讲



参考文章

- [How to save user settings using UserDefaults - hackingwithswift](https://www.hackingwithswift.com/example-code/system/how-to-save-user-settings-using-userdefaults)
- [Reading and writing basics: UserDefaults - hackingwithswift](https://www.hackingwithswift.com/read/12/2/reading-and-writing-basics-userdefaults)

参考视频项目：

- [Project UserDefaults - hackingwithswift](https://www.hackingwithswift.com/read/12/1/setting-up)



另外，如何对上面定义的 **`key`** 进行优雅的管理，可以参考：

- [Swift中安全优雅的使用UserDefaults - 简书](https://www.hackingwithswift.com/read/12/1/setting-up)



2019年05月05日21:40:39



