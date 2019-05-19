swift中通过 基于 **`Networking`** 的   **`URLSession`** 相关的类 提供了很强大的网络请求方案。一般会使用第三方封装好的库,下面是几个常用的网络请求库：

- [Alamofire Swift版本 - github](https://github.com/Alamofire/Alamofire)
- [Moya Swift版本 - github](https://github.com/Moya/Moya)
- [AFNetworking ObjectiveC版本- github](https://github.com/AFNetworking/AFNetworking)

前2个是Swift版本，第3个是OC版本，因为可以桥接，所以在Swift中也可以使用。

虽然第三方库用起来很方便，但是了解一下网络相关概念，对自定义封装是必不可少的，下面介绍一下 **`URLSession`** 相关的类，属性，委托，使用方法等。



## 1.创建session

创建session有3种方式，返回一个 **`URLSession`** 类型实例：

1. **`URLSession.shared `**: 创建一个单例，特点：
   - 简单的网路请求
   - 不能添加配置参数，权限验证，cookie,以及和session交互
   - 不存在委托
2. **`init(configuration: URLSessionConfiguration)`**, 通过构造器，传入一个配置信息对象作为参数，特点：
   - 可以通过配置来设定想要的网络环境设定
   - 不存在委托，在网络任务过程中，不能进行交互
3. **`init(configuration: URLSessionConfiguration, delegate: URLSessionDelegate?, delegateQueue queue: OperationQueue?)`**,
   - 可以提供一个配置
   - 使用委托，可以在任务过程中提供交互，比如提供下载进度监控，错误捕获等等
   - 可以指定线程，主线程还是背景线程

```swift
// 方式1
let session1 = URLSession.shared

// 方式2
let configuration = URLSessionConfiguration.default
let session2 = URLSession(configuration: configuration)

// 方式3
let configuration = URLSessionCOnfiguration.ephemeral // 配置方案2 类似浏览器的隐私模式
configuration.waitsForConnectivity = true // 初始网络不存在时，稍后会再次尝试请求 iOS 11
configuration.timeoutIntervalForRequest = 60*1000*2 // 请求超时时间2min
let session3 = URLSession(
	configuration: configuration,
	delegate: self,
	delegateQueue: OperationQueue.main // 主线程
)
```

注意一个应用一般只用一个 **`session`** 即可。



## 2.配置session环境

**`URLSessionConfiguration`** 类提供了3个类属性（class property）,通过类属性创建一个实例，在其实例设置不同的请求配置：

- **`URLSessionConfiguration.default`**: 基础的配置，用得最多的
- **`URLSessionConfiguration.ephemeral`**: cookie 和 caches只存在在内存中，不会被保存，**类似浏览器的隐私模式浏览**，也可以通过上面的 **`default`** 配置来实现，这个是一个便利方式
- **`URLSessionConfiguration.background(withIdentifier: String)`**: 这个多用于背景运行网络环境，比如杀掉应用后仍可下载数据， **`withIdentifier`** 需要是一个唯一值，用来区分其它的apps，一般使用 **app bundle id** 作为这个标识符



下面是可以设置的部分配置：

- **`allowCellularAccess: Bool`**: 是否允许使用手机数据或需要wifi
- **`waitsForConnectivity: Bool`**: 如果为 **`true`**, 则初始网络不存在，稍后网络恢复后再去请求，对于 **`background session`** 这个属性默认为 **`true`**, 并且无法更改， **`iOS 11+`**
- **`httpMaximumConnectionsPerHost: Int`**: 同时请求的最大并发量
- **`timeoutIntervalForRequest: TimeInterval`**: 请求超时时间。对于 **`background session`** 这个属性无效。 默认是 **`1分钟`** （**`TimeInterval`** 是 **`Double`** 类型的一个别名）
- **`timeoutIntervalForResource: TimeInterval`**: 最大的下载过期时间。比如一个文件设置10天内下完，如果10天内没下完，则这个连接请求就无效了。 默认是 **`7天`**

还有其它的配置： **`cookie | caching | credential | proxy | protocol`** 等等



```swift
let config = URLSessionConfiguration.default
config.allowCellularAccess = true
config.waitsForConnectivity = true
config.timeoutIntervalForRequest = 10.0

let session = URLSession(configuration: config)
```





## 3.session 任务

一个网络请求swift中称之为任务（**`URLSessionTask`** 类型),swift中有 **`4`** 种类型的任务：

1. **`URLSessionDataTask`**: 继承自 **`URLSessionTask`** ，一般用于普通的http请求，不适合大量数据，因为数据直接存放在内存中，不支持 **`background sessions`**, 因为背景session不会存放在内存中
2. **`URLSessionUploadTask`**: 继承自 **`URLSessionDataTask`**， 用于上传文件，可以监听上传的进度
3. **`URLSessionDownloadTask`**: 继承自 **`URLSessionTask`** ，增加了一个 **`cancel`** 方法，用于下载任务，不通过内存，以文件的形式对数据进行累积，最后会返回一个 **`file url`**,文件是保存还是丢弃由开发者决定
4. **`URLSessionStreamTask`**:  继承自 **`URLSessionTask`** ,对流类型文件进行处理



> URLSessionTask 属性

列举比较常用的属性：

- **`taskIdentifier: Int`**: 每个任务都有一个唯一的标识符，这个对后面多个任务中如何分辨哪个task是哪一个**很重要**，可以对不同的task可以使用不同的回调

- **`originRequest: URLReques? & currentRequest: URLRequest?`**: 因为重定向，请求可能发生变化

- **`priority`**： 0-1之间的浮点值，表示任务的优先级，数值越大，优先级越高，系统默认提供了3个可选的级别

  - **`URLSessionTask.lowPriority`**: 0.25
  - **`URLSessionTask.defaultPriority`**: 0.5  (默认值)
  - **`URLSessionTask.highPriority`**: 0.75

- **`progress: Progress`**: 进度， 可以用这个属性对任务进度进行监听，因此可以在不使用委托的情况下对任务进度进行监听，但是是 **`iOS 11+`**才可以使用

- **`response: URLResponse?`**: 从服务器初始返回的响应

- **`error: Error?`**: 任务失败

- **`getAllTasks(comletionHandler: @escaping ([URLSessionTask]) -> Void)`**: 在任务完成回调中返回当前存在的所有任务，session会在任务完成或者取消后释放task。如果不存在悬停或者正在运行的任务，则这个函数返回为空

- **`state: URLSessionTask.State`**: 任务的状态

  ```swift
  extension URLSessionTask {
      public enum State: Int {
          case running
          case suspended // 默认是 悬停 状态
          case canceling
          case completed
      }
  }
  ```



> 控制任务状态

上面属性可知，任务是默认 **`suspended`** （悬停） 状态，一般我们第一次请求时，会启动任务（**`resume()`**）

```swift
struct User: Codable {
    let id: Int
    let name: string
}


let url = URL("https://api.com/v1/users")!
let request = URLRequest(url: url)
request.setValue("application/json", forHTTPHeaderField: "Content-Type") // 设置已经存在的请求头
request.addValue("1829-19939-1991", forHTTPHeaderField: "uuid") // 添加自定义请求头
request.httpMethod = "POST" // 请求方式

let user = User(id: 10, name: "wallace")

do {
   	let data = try JSONEncoder().encode(user)
    request.httpBody = data // 设置请求体
} cath {
    print(error)
}

let session = URLSession.shared // 创建一个session

// 返回一个任务
let task = session.dataTask(with: request) { data, response, error in 
                                           // ...
                                           }
// 因为任务默认是 suspended
// 因此第一次需要开启任务
task.resume()
```



除了对任务 **`resume()`**, 还可以 **`cancel()`** 和 **`suspend()`**  操作。**也可以告诉 `progress` 对象进行相同的操作**：

```
task.progress.resume()
task.progress.cancel()
task.progress.suspend()
```



## 4.session 委托

使用委托的一般是为了添加更多的功能，比如监听下载的进度，任务完成或失败之后的回调。

**注意，一个session对应一个委托，而不是一个任务对应一个委托。** 因为一个session可以有多个任务，处理时，需要使用 **`taskIdentifier`** 对任务进行区分。

有以下几种类型委托：

- **`URLSessionDelegate`**
- **`URLSessionTaskDelegate`**: 继承自上面的 **`URLSessionDelegate`**
- **`URLSessionDataDelegate`**: 继承自上面的 **`URLSessionTaskDelegate`**
- **`URLSessionUploadDelegate`**: 继承自上面的 **`URLSessionTaskDelegate`**
- **`URLSessionDownloadDelegate`**: 继承自上面的 **`URLSessionTaskDelegate`**
- **`URLSessionStreamDelegate`**: 继承自上面的 **`URLSessionTaskDelegate`**



添加委托的方式

```swift
// 使用这个构造器
init(configuration: URLSessionConfiguration, delegate: URLSessionDelegate?, queue: OperationQueue?)

// 伪代码
class Person: URLSessionDelegate {
    let config = URLSessionConfiguration.default
    let session = URLSession(configuration: config, delegate: self, queue: Operation.main)
}
```

>  **URLSessioDataDelegate** 中比较常用的几个方法：

```swift
// didReceive 会间歇性的收到数据
// 在下载过程中，可能调用多次，每次都会提供一个新的数据（Data 类型），我们需要对数据进行累计
func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data)

// 是否下载完成，我们可以在这里知道，在这里可以使用累积的数据
// 如果下载出错了 也可以在这里收到通知
func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?)
```



> **URLSessionDownloadDelegate** 中比较常用的几个方法



```swift
// 对可恢复下载任务的开始和暂停
func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didResumeAtOffset fileOffset: Int64, expectedTotalBytes: Int64)

// 用于监控下载的进度
func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didWriteData bytesWritten: Int64, totalBytesWritten: Int64, totalBytesExpectedToWrite: Int64)

// 下载任务完成，返回下载文件存储的file url
func urlSession(_ session: URLSession, downloadTask: URLSessionDownloadTask, didFinishDownloadingTo location: URL)

// 这个在download task中不太重要
func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?)
```





## 示例



> 1. 单张图片下载和UI显示



页面中一个 **`UIViewImage`**,一个 **`UIButton`**, 一个 **`UIProgressView`**,点击按钮下载图片，显示下载进度

```swift
class ViewController: UIViewController {
    @IBOutlet weak var iv: UIImageView!
    @IBOutlet weak var prog: UIProgressView!
    
    @IBAction func downloadImage(_ sender: Any) {
        self.iv.image = nil
        let s = "https://www.apeth.net/matt/images/phoenixnewest.jpg"
        let url = URL(string: s)!
        
        let session = URLSession.shared
        
        let task = session.downloadTask(with: url) { (loc, res, error) in
        	guard error == nil else {
                print("error")
                return
        	}
        	guard let status = (res as! HTTPURLResponse).statusCode, status == 200 else {
                print("status is not 200, and is: \(status)")
                return
        	}
        	
        	if let path = loc, let data = try? Data(contentOf: loc) {
                let im = UIImage(data: data)
                DispatchQueue.main.async {
                	// 注意 UI更新需要在主线程上完成
                    self.iv.image = im
					print("done")
                }
        	}
        }
        
        task.priority = URLSessionTak.defaultPriority
        if #available(iOS 11, *) {
            // task的progress属性只有iOS11+上才存在
            self.prog.observedProgress = task.progress
        }
        
        // task 默认是suspend状态，第一次需要手动的开始
        task.resume()
    }
}
```



> 2. 使用URLSessionDataDelegate委托

```
class ViewController: UIViewController, URLSessionDataDelegate {
    @IBOutlet weak var iv: UIImageView!
    var data = [Int:Data]() // 存储数据的字典 key是task.taskIdentifier
    
    lazy var session: URLSession = {
       let config = URLSessionConguration.ephemeral
       config.allowCellularAccess = false
       config.requestCachePolicy = .reloadIgnoringLocalCacheData
       config.urlCache = nil
       config.httpMaximumConnectionsPerHost = 2
       let queue = OperationQueue() // 线程
       let session = URLSession(configuration: config, delegate: self, delegateQueue: queue)
       return session
    }()
    
    @IBAction func doHttp(_ sender: Any) {
        self.iv.image = nil
        let s = "https://www.nasa.gov/sites/default/files/styles/1600x1200_autoletterbox/public/pia17474_1.jpg"
        let url = URL(string: s)!
        let request = URLRequest(url: url)
        let task = session.dataTask(with: request)
        self.data[task.taskIdentifier] = Data() // 将数据清空
        task.resume()
    }
    
    // 数据是分块接收的
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
    	// 将接收到的数据添加到对应的任务中数据中
        self.data[dataTask.taskIdentifier]?.append(data)
        print("""
        	任务 \(dataTask.taskIdentifier) 接收到 \
        	\(data.count) 字节数据； \
        	总数据 \(self.data[dataTask.taskIdentifier]!.count)
        """)
    }
    
    // 任务完成 或者 任务出错
    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
    	// 将接收的最后数据保存下来
        let d = self.data[task.taskIdentifier]!
        // 然后将对应的任务数据置空
        self.data[task.taskIdentifier] = nil
        
        if error == nil {
            DispatchQueue.main.async {
                self.iv.image = UIImage(data: d)
            }
        }
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        // 页面消失时
        // 允许现有任务完成，然后释放委托，不再使用 避免内存泄漏
        self.session.finishTasksAndInvalidate()
    }
    
    deinit {
        print("view destroy")
    }
}
```







