前面文章中 [UserDefaults 的基本用法](https://www.jianshu.com/p/f76cf4c9471d) 中对UserDefaults 进行了简单的介绍，它可以将一些简单的数据类型存储在本地，需要使用的时候再去读取。

如果对于复杂对象的存储则需要将其进行序列化，将对象转化为 **`NSData`**(Swift Data)类型之后再进行操作，比如，将其存在本地的某个文件（eg.people.plist, people.txt等）中。

有2种序列化的方式：

1. **`NSCoding`**： 老的Cocoa方式，OC的方式
2. **`Codable`**： 新的swift方式



## NSCoding

这个协议在Cocoa的Foundation框架中定义，内置的大多数Cocoa类都采用了NSCoding协议，比如 **`UIColor`** 等。

采用了这个协议的对象可以转换为 **`NSData`** 类型，然后再转换回来。使用 **`NSKeyedArchiver`** 和 **`NSKeyedUnarchiver`** 分别进行归档和解档。

采用这个协议的对象需要实现 **`encode(with:)`** 进行归档，以及 **`init(coder:)`** 进行解档。

比如，自定义的 **`Person`** 类，有2点值得说明的， 来源[NSCoding - hackingwithswift](https://www.hackingwithswift.com/read/12/3/fixing-project-10-nscoding)：

- **为什么使用class，而不是struct？** 因为NSCoding需要使用对象，或者在字符串，数组，字典的情况下，使用可以与对象互换的结构，如果把Person当做一个struct，我们不能在NSCoding中使用它
- **为什么要继承 `NSObject`?** 因为使用NSCoding，必须使用NSObject,否则应用会崩溃

```swift
class Person: NSObject, NSCoding {
    var firstName: String
    var lastName: String
    var age: Int
    
    // 如果定义一个实例Person，打印结果将是这里定义的描述字符串
    override var descirption: String {
        return "\(self.firstName) \(self.lastName) \(age)"
    }
    
    init(firstName: String, lastName: String, age: Int) {
        self.firstName = firstName
        self.lastName = lastName
        self.age = age
    }
    
    // 实现NSCoding 协议中的方法
    func encode(with aCoder: NSCoder) {
        // 如果Person 还有一个父类，假设Human也采用了NSCoding协议
        // 则必须先调用父类的 super
        // 这里不需要
        aCoder.encode(self.firstName, forKey: "first")
        aCoder.encode(self.lastName, forKey: "last")
        aCoder.encode(self.age, forKey: "age")
    }
    
    required init?(coder aDecoder: NSCoder) {
        // 同上，如果存在父类采用NSCoding协议，则也需要先调用父类的构造器
        
        // 注意这里返回的是 NSString 类型
        self.firstName = aDecoder.decodeObject(of: NSString.self, forKey: "first")! as String
        self.lastName = aDecoder.decodeObject(of: NSString.self, forKey: "last")! as String
        // 对于Int类型
        self.age = aDecoder.decodeInteger(forKey: "age")
    }
}
```



**在 `iOS12`中，苹果推荐使用 `NSSecureCoding` 协议，这个协议在NSCoding的基础上，还需要实现一个静态的 `static var supportsSecureCoding: Bool {return true} 属性`**。



```swift
class Person: NSObject, NSSecureCoding {
    static var supportsSecureCoding: Bool { return true } // 需要添加这个静态属性
    
    var firstName: String
    var lastName: String
    var age: Int
    
    // 如果定义一个实例Person，打印结果将是这里定义的描述字符串
    override var descirption: String {
        return "\(self.firstName) \(self.lastName) \(age)"
    }
    
    init(firstName: String, lastName: String, age: Int) {
        self.firstName = firstName
        self.lastName = lastName
        self.age = age
    }
    
    // 实现NSCoding 协议中的方法
    func encode(with aCoder: NSCoder) {
        // 如果Person 还有一个父类，假设Human也采用了NSCoding协议
        // 则必须先调用父类的 super
        // 这里不需要
        aCoder.encode(self.firstName, forKey: "first")
        aCoder.encode(self.lastName, forKey: "last")
        aCoder.encode(self.age, forKey: "age")
    }
    
    required init?(coder aDecoder: NSCoder) {
        // 同上，如果存在父类采用NSCoding协议，则也需要先调用父类的构造器
        
        // 注意这里返回的是 NSString 类型
        self.firstName = aDecoder.decodeObject(of: NSString.self, forKey: "first")! as String
        self.lastName = aDecoder.decodeObject(of: NSString.self, forKey: "last")! as String
        // 对于Int类型
        self.age = aDecoder.decodeInteger(forKey: "age")
    }
}
```



将数据存储在本地的documents文件夹中的 **`person.txt`** 文件中

```swift
let fm = FileManager.default
// 获取documents 文件夹所在的URL
let docsurl = try fm.url(.documentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
let personFile = docsurl.appendingPathComponent("person.txt")

let person = Person(firstName: "louis", lastName: "lili", age: 20)

// 将上面的person使用 NSKeyedArchiver 进行存储
// 先转换为 NSData 类型
// 如果使用 NSSecureCoding 协议， 则requiringSecureCoing则需要使用true
// 如果使用 NSCoding 协议， 则requiringSecureCoing为false
let personData = try NSKeyedArchiver.archivedData(withRootObject: person, requiringSecureCoing: true)

// 使用 write(to:) 方法写入文件
// 哪些数据类型可以使用 write(to:) 方法，下面会介绍
personData.write(to: personFile, options: .atomic, encoding: .utf8)
```

**`NSString`** 和 **`NSData`** 对象可以直接的将内容写入到文件中：

```swift
try "funny".write(to: someFileUrl/file.txt, atomically: true, encoding: .utf8)
```

对于 **`NSArray`** 和 **`NSDictionary`**，实际上他们是 **属性列表(property lists)**, 它们需要其包含的内容都是 **属性列表类型(`property list types`)**,这些类型包括：

- **`NSString`**
- **`NSData`**
- **`NSDate`**
- **`NSNumber`**
- **`NSArray`**
- **`NSDictionary`**

如果是以上类型则都可以直接写入文件，比如：

```swift
// 数组
let arr = ["hello", "world"]
let temp = FileManager.default.temporaryDirectory
let f = temp.appendingPathComponent("pep.plist")
// 转化为 NSArray类型
try (arr as NSArray).write(to: f)
```

回到正题，刚才将数据person转化为 **`NSData`** 后写入了文件，下面是读取数据的方法：

```swift
// 将 personFile 路径下文件的内容读取为 NSData 格式
let personData = try NSData(contentsOf: personFile)
// 然后进行解档
// 注意这里的ofClass是 Person.self
// 如果存入的数据是 [Person]数组，则这里相对应的则是 [Person].self
let personObj = try NSKeyedUnarchiver.unarchivedObject(ofClass: Person.self, from: personData)!

print(person) // louis lili 20
```





## Codable

这个是swift4.0中引入的新协议，主要是为了解决数据（比如JSON）序列化问题。它实际上是 **`Encodable`** 和 **`Decodable`** 协议的结合.

使用Codable的对象，类实例，结构体实例，枚举实例（**`RawRepresentable`** 类型的枚举，即拥有 raw value）等都可以被编码

```swift
protocol Codable: Encodable & Decodable {}
```

任何对象只要遵守Encodable协议，都可以被序列化（归档），任何遵循Decodable协议的对象都可以从序列化形式恢复（解档）。

存在3种形式的序列化模式：

- **`property list`**: 使用 **`PropertyListEncoder`** 的 **`encode(_:)`** 进行编码，使用 **`PropertyListDecoder`** 的 **`decode(_:from:)`** 进行解码
- **`JSON`**: 使用 **`JSONEncoder`** 的 **`encode(_:)`** 进行编码，使用 **`JSONDecoder`** 的 **`decode(_:from:)`** 进行解码
- **`NSCoder`**: 使用 **`NSKeyedArchiverEncoder`** 的 **`encodeEncodable(_:forKey:)`** 进行编码，使用 **`NSKeyedUnarchiverDecoder`** 的 **`decodeDecodable(_:forKey:)`** 进行解码

大多数内置的Swift类型都是默认的Codable，**`encode(to:)`** 和 **`init(from:)`** 类似于 NSCoding中的 **`encode(with:)`** 和 **`init(coder:)`**, 但是通常不需要我们去实现，**因为通过扩展协议的方式，提供了默认的实现**

上面的存储 **`Person`** 实例的方式，这里可以写为：

```swift
// 不需要写 encode(with:) 和 init(coder:) 的协议方法
// 因为协议扩展 extension Codable 中提供了默认实现
class Person: NSObject, Codable {
 	var firstName: String
    var lastName: String
    var age: Int
    
    // 如果定义一个实例Person，打印结果将是这里定义的描述字符串
    override var descirption: String {
        return "\(self.firstName) \(self.lastName) \(age)"
    }
    
    init(firstName: String, lastName: String, age: Int) {
        self.firstName = firstName
        self.lastName = lastName
        self.age = age
    }   
}
```

推荐使用 **`PropertyListEncoder & PropertyListDecoder`**, 这个的实现方式和 **`NSKeyedArchiver & NSKeyedUnarchiver`** 类似

```swift
// 对数据进行存储
let docsurl = try FileManager.default.url(for: .docmentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
let filePath = docsurl.appendingPathComponent("person.txt")
let person = Perosn(firstName: "yuuki", lastName: "lili", age: 20)
let encodedPerson = try PropertyEncoder().encode(person) // 编码
// 写入到该文件
encodedPerson.write(to: filePath, options: .atomic)


// 对数据进行读取
let contents = try Data(contentOf: filePath)
let decodedPerson = try PropertyListDecoder().decode(Person.self, from: contents) // 解码
print(decodedPerson) // "yuuki lili 20"
```



## 示例

主要有以下几个方面：

- NSCoding 和 Codable 结合使用，因为Cocoa中很多类采用了 **`NSCoding`** 协议，而不是 Codable协议，有时候需要将2者结合起来一起用
- http请求返回的JSON数据的编码和解码
- 使用**`CodingKeys`**枚举都字段进行自定义命名



### 示例1.使用Codable存储NSCoding数据

比如 **`UIColor, UIImage`** 都使用的NSCoding协议，比如存储下面数据

```swift
struct Person {
    var name: String
    var favoriteColor: UIColor
}
```

这需要自己手动实现 **`init(from:)`** 和 **`encode(to:)`** 协议方法，并且使用 **`CodingKeys`** 对 **`key`** 一个接一个的进行匹配。

需要4个步骤：

1. 扩展 **`Person`**, 存放 **`Codable`** 功能
2. 创建自定义coding keys，用来描述存储的数据是什么
3. 创建一个 **`init(from:)`** 方法，将原始数据转换回一个 **`UIColor`**, 使用 **`NSKeyedUnarchiver`** 进行解档
4. 创建一个 **`encode(to:)`** 方法，将UIColor转化为原始数据， 使用 **`NSKeyedArchiver`** 进行归档

上面的**`3 & 4`**步骤前面说过，对于所有采用Codable的类型一般可以省略，这里需要自己实现转换

```swift
extension Person: Codable {
	// 因为我们需要显示的声明编码和解码的内容
	// 因此需要在这里写出CodingKeys
	// CodingKeys 遵循 String, CodingKey
    enum CodingKeys: String, CodingKey {
        case name
        case favoriteColor
    }
    
    init(from decoder: Decoder) throws {
        // 注意这里的 keyedBy
        let container = try decoder.container(keyedBy: CodingKeys.self)
        // 字符串类型，直接解码
        name = try container.decode(String.self, forKey: .name)
        
        let colorData = try container.decode(Data.self, forKey: .favoriteColor)
        favoriteColor = try NSKeyedUnarchiver.unarchiverTopLevelObjectWithData(colorData) as? UIColor ?? UIColor.black
    }
    
    func encode(to encoder: Encoder) throws {
        let container = try encoder.container(keyedBy: CodingKeys.self)
        try container.encode(name, forKey: .name)
        
        let colorData = try NSKeyedArchiver.archivedData(withRootObject: color, requiringSecureCoding: false)
        try container.encode(colorData, forKey: .favoriteColor)
    }
}

// 使用
let taylor = Person(name: "Taylor Swift", favoriteColor: .blue)
let encoder = JSONEncoder()
let decoder = JSONDecoder()

do {
    // 编码
    let encoded = try encoder.encode(taylor)
    let str = String(decoding: encoded, as: UTF8.self)
    print(str)
    
    // 解码
    let person = try decoder.decode(Person.self, from: encoded)
    print(person.favoriteColor, person.name)
} catch {
    print(error,localizedDescription)
}

```

示例来源：

- [how to stort nscoding data using codable](<https://www.hackingwithswift.com/example-code/language/how-to-store-nscoding-data-using-codable>)





## 示例2：Decodable & Encodable

假设网络请求返回的数据是一个json，格式：



> 1.返回一个普通的字典

```swift
{
    "id":1,
    "name":"Instagram Firebase",
    "link":"https://www.letsbuildthatapp.com/course/instagram-firebase",
    "imageUrl":"https://letsbuildthatapp-videos.s3-us-west-2.amazonaws.com/04782e30-d72a-4917-9d7a-c862226e0a93",
    "number_of_lessons":49
}
```

这个可以定义一个结构体，使用 **`JSONDecoder`** 实例的 **`decode`** 方法对返回的数据进行解析：

```
// 这个结构体遵循 Decodable协议
struct Course: Decodable {
 	let id: Int
    let name: String
    let link: String
    let imageUrl: String
    let number_of_lessons: Int
}

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let urlStr = "https://api.letsbuildthatapp.com/jsondecodable/course"
        guard let url = URL(string: urlStr) else { return }
        
        let task = URLSession.shared.dataTask(with: url) { (data, response, err) in
        	guard let data = data else { return }
            
             do {
             	// 使用 JSONDecoder对数据进行解析
                 let course = try JSONDecoder().decode(Course.self, from: data)
                 print("course", course)
             } catch {
                 print(error.localization)
             }
        }
        task.resume()
    }
}
```



>  2.如果返回的数据是一个 **`Course`** 数组：

```json
// 20190506205007
// https://api.letsbuildthatapp.com/jsondecodable/courses

[
  {
    "id": 1,
    "name": "Instagram Firebase",
    "link": "https://www.letsbuildthatapp.com/course/instagram-firebase",
    "imageUrl": "https://letsbuildthatapp-videos.s3-us-west-2.amazonaws.com/04782e30-d72a-4917-9d7a-c862226e0a93",
    "number_of_lessons": 49
  },
  {
    "id": 2,
    "name": "Podcasts Course",
    "link": "https://www.letsbuildthatapp.com/course/podcasts",
    "imageUrl": "https://letsbuildthatapp-videos.s3-us-west-2.amazonaws.com/32f98d9d-5b9b-4a22-a012-6b87fd7158c2",
    "number_of_lessons": 39
  }
]
```

则上面需要改动的地方为：

```swift
// Course.self 更改为 [Course].self
let courses = try JSONDecoder().decode([Course].self, from: data)
```



>  3.如果返回数据类型是多种数据类型组合

```json
// 20190506205524
// https://api.letsbuildthatapp.com/jsondecodable/website_description

{
  "name": "Lets Build That App",
  "description": "Teaching and Building Apps since 1999",
  "courses": [
    {
      "id": 1,
      "name": "Instagram Firebase",
      "link": "https://www.letsbuildthatapp.com/course/instagram-firebase",
      "imageUrl": "https://letsbuildthatapp-videos.s3-us-west-2.amazonaws.com/04782e30-d72a-4917-9d7a-c862226e0a93",
      "number_of_lessons": 49
    },
    {
      "id": 4,
      "name": "Kindle Basic Training",
      "link": "https://www.letsbuildthatapp.com/basic-training",
      "imageUrl": "https://letsbuildthatapp-videos.s3-us-west-2.amazonaws.com/a6180731-c077-46e7-88d5-4900514e06cf",
      "number_of_lessons": 19
    }
  ]
}
```

则需要再定义一个结构体,这个结构体也使用 **`Decodable`** 协议：

```swift
struct Website: Decoable {
    let name: String
    let description: String
    let courses: [Course] // 可以进行组合
}
```

则修改部分：

```swift
// [Course].self 更改为 Website.self
let website = try JSONDecoder().decode(Website.self, from: data)
```



> 4.如果返回的数据某些可能为空

```json
// 20190506210445
// https://api.letsbuildthatapp.com/jsondecodable/courses_missing_fields

[
  {
    "id": 1,
    "name": "Instagram Firebase",
    "link": "https://www.letsbuildthatapp.com/course/instagram-firebase",
    "imageUrl": "https://letsbuildthatapp-videos.s3-us-west-2.amazonaws.com/04782e30-d72a-4917-9d7a-c862226e0a93",
    "number_of_lessons": 49
  },
  {
    "id": 4,
    "name": "Kindle Basic Training",
    "link": "https://www.letsbuildthatapp.com/basic-training",
    "imageUrl": "https://letsbuildthatapp-videos.s3-us-west-2.amazonaws.com/a6180731-c077-46e7-88d5-4900514e06cf",
    "number_of_lessons": 19
  },
  {
    "name": "Yelp"
  }
]
```



则需要将结构体中某些类型定义为可选类型

```swift
struct Course: Decodable {
    let id: Int? // 可选类型
    let name: String
    let link: String?
    let number_of_lessons: Int?
}
```

将其修改为：

```swift
let courses = try JSONDecoder().decode([Course].self, from: data)
```

示例来源：

- [Parsing JSON Just Became Super Easy in Swift 4 with Decodable - YouTube](https://www.youtube.com/watch?v=YY3bTxgxWss)



**上面的示例只对获取到的数据进行了解析，如果要上传数据，则需要使用到 `JSONEncoder` 实例的 `encode` 方法对数据进行编码操作， 过程和上面示例的过程类型**， 可参考：

- [Encodable and Decodable  - YouTube](https://www.youtube.com/watch?v=P6NEUpLhpUM)



### 3.CodingKeys

上面的示例，Course 结构体的属性名要和后台保持一致，如果想要自定义属性名，则需要添加 **`CodingKeys`** 枚举，上面 **示例1** 中其实已经出现过了，这里单独拿出来说明一下：

```swift
// 原来的
struct Course: Decodable {
 	let id: Int
    let name: String
    let link: String
    let imageUrl: String
    let number_of_lessons: Int
}

// 将 name，link, number_of_lessons 属性分别进行修改
struct Course: Decodable {
    let id: Int
    let courseName: String
   	let courseLink: String
	let imageUrl: String
    let courseCount: Int
    
    // CodingKeys 遵循String 和 CodingKey 协议
    enum CodingKeys: String, CodingKey {
        case id, imageUrl
        case courseName = "name"
        case courseLink = "link"
        case courseCount = "number_of_lessons"
    }
}

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let urlStr = "https://api.letsbuildthatapp.com/jsondecodable/course"
        guard let url = URL(string: urlStr) else { return }
        
        let task = URLSession.shared.dataTask(with: url) { (data, response, err) in
        	guard let data = data else { return }
            
             do {
             	// 使用 JSONDecoder对数据进行解析
                 let course = try JSONDecoder().decode(Course.self, from: data)
                 print("course", course.courseCount) // 使用自定义的属性名
             } catch {
                 print(error.localization)
             }
        }
        task.resume()
    }
}
```





## 总结

这些协议对数据结构的处理，存储还是很重要的，主要涉及知识点：

- **`NSCoding & NSSecureCoding`**
- **`NSObject`**
- **`Codable & Encodable & Decodable`**
- **`PropertyListEncoder & PropertyListDecoder`**
- **`NSKeyedArchiver & NSKeyedUnarchiver`**
- **`JSONEncoder & JSONDecoder`**
- **`NSCoder`**

另外还提到了：

- **`Property List & Property List types`**
  - **`NSData`**
  - **`NSString`**
  - **`NSNumber`**
  - **`NSArray`**
  - **`NSDictionary`**

- **`FileManager`**
- **`URLSession`** 网络请求



 2019年05月06日21:26:46



