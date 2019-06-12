使用 **`URLSession`** 对请求进行封装，下面是基本的功能：

```swift
enum APIError {
    case invalidURL
    case requestFailed
}

struct APIRequest {
    typealias APIClientCompletion = (HTTPResponse?, Data?, APIError?) -> Void
    
    private let session = URLSession.shared
    private let baseURL = URL(string: "https://jsonplaceholder.typpicode.code")
    
    func request(method: String, path: String, _ completion: @escaping APIClientCompletion) {
    	// 注意这里是appendingPathComponent 不是 appendPathComponent 方法
        guard let url = baseURL?.appendingPathComponent(path) else {
            completion(nil, nil, .invalidURL)
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = method
        
        let task = session.dataTask(with: url) { (data, response, error) in 
        	guard let httpResponse = response as? HTTPURLResponse else {
                completion(nil, nil, .requestFailed)
                return
        	}
        	
        	completion(httpResponse, data, nil)
        }
        task.resume()
    }
}
```

示例：

```swift
APIClient().request(method: "get", path: "todos/1") { (_, data, _) in
	if let data = data, let result = String(data: data, encoding: .utf8) {
        print(result)
	}
}
// 打印结果
{
    "userId": 1,
    "id": 1,
    "title": "delectus aut autem",
    "completed": false
}
```



------------------------

上面的封装对简单的网络请求可以满足，但是有些请求带有查询参数（比如： **`/todos/?hello=world`**）, 或者存在请求体，另外，swift是强类型，**`method`** 请求方式使用 **`String`** 类型很容易出错，下面对其进行改进：

```swift
enum APIError {
    case invalidURL
    case requestFailed
}

// 请求方式
enum HTTPMethod: String {
    case get = "GET"
    case post = "POST"
    case delete = "DELETE"
    case put = "PUT"
    case head = "HEAD"
    case options = "OPTIONS"
    case trace = "TRACE"
    case connect = "CONNECT"
}

// 请求头
struct HTTPHeader {
    let field: String
    let value: String
}

// 请求所需要的信息
class APIRequest {
    let method: HTTPMethod
    let path: String
    var queryItems: [URLQueryItem]? // 查询的参数
    var headers: [HTTPHeader]?
    var body: Data?
    
    init(method: HTTPMethod, path: String) {
        self.method = method
        self.path = path
    }
}

struct APIClient {
    typealias APIClientCompletion: (HTTPURLResponse?, Data?, APIError?) -> Void
    
    private let session = URLSession.shared
    private let baseURL = URL(string: "https://jsonplaceholder.typpicode.code")!
    
    // 这里的第一个参数变为 APIRequest
    func request(_ request: APIRequest, _ completion: @escaping APIClientCompletion) {
        var urlComponents = URLComponents()
        urlComponents.scheme = baseURL.scheme
        urlComponents.host = baseURL.host
        urlComponents.path = baseURL.path
        urlComponents.queryItems = request.queryItems
        
        guard let url = urlComponents.url?.appendingPathComponent(request.path) else {
            completion(nil, nil, .invalidURL)
            return
        }
        
        var urlRequest = URLRequest(url: url)
        urlRequest.httpMethod = request.method
        urlRequest.httpBody = request.body
        
        request.headers?.forEach {
            urlRequest.addValue($0.value, forHTTPHeaderField: $0.field)
        }
        
        let task = session.dataTask(with: url) { (data, response, error) in
        	guard let httpResponse = response as? HTTPResponse else {
                completion(nil, nil, .requestFailed)
                return
        	}
        	
        	completion(httpResponse, data, nil)
        }
        task.resume()
    }
}
```

使用：

```swift
let request = APIRequest(method: .post, path: "posts")
request.queryItems = [URLQueryItem(name: "hello", value: "world")]
request.headers = [HTTPHeader(field: "Content-Type", value: "application/json")]
request.body = Data()

APIClient().request(request) { (_, data, _) in
	if let data = data, let result = String(data: data, encoding: .utf8) {
        print(result)
	}
}

// 打印
[
    {
        "userId": 1,
        "id": 1,
        "title": "skdlkseo dkdk",
        "body": "djsdksjd"
    },
    {
        "userId": 2,
        "id": 2,
        "title": "esdsokseo dkdk",
        "body": "ddsds3e"
    },
    //....
]
```



------------------------

上面的例子中都是使用 **`String(data:encoding:)`** 将json数据转换为字符串，实际项目中，需要将json数据进行编码和解码，可以使用 **`JSONEncoder & JSONDecoder`** 对数据进行处理



```swift
enum HTTPMethod: String {
    case get = "GET"
    case put = "PUT"
    case post = "POST"
    case delete = "DELETE"
    case head = "HEAD"
    case options = "OPTIONS"
    case trace = "TRACE"
    case connect = "CONNECT"
}

struct HTTPHeader {
    let field: String
    let value: String
}

class APIRequest {
    let method: HTTPMethod
    let path: String
    var queryItems: [URLQueryItem]?
    var headers: [HTTPHeader]?
    var body: Data?

    init(method: HTTPMethod, path: String) {
        self.method = method
        self.path = path
    }

	// 对请求体进行编码
    init<Body: Encodable>(method: HTTPMethod, path: String, body: Body) throws {
        self.method = method
        self.path = path
        self.body = try JSONEncoder().encode(body)
    }
}

// 请求响应结果
struct APIResponse<Body> {
    let statusCode: Int
    let body: Body
}

extension APIResponse where Body == Data? {
    func decode<BodyType: Decodable>(to type: BodyType.Type) throws -> APIResponse<BodyType> {
        guard let data = body else {
            throw APIError.decodingFailure
        }
        let decodedJSON = try JSONDecoder().decode(BodyType.self, from: data)
        return APIResponse<BodyType>(statusCode: self.statusCode,
                                     body: decodedJSON)
    }
}

enum APIError: Error {
    case invalidURL
    case requestFailed
    case decodingFailure // 新增 编码错误
}

// 请求结果的2种枚举
enum APIResult<Body> {
    case success(APIResponse<Body>)
    case failure(APIError)
}

struct APIClient {

    typealias APIClientCompletion = (APIResult<Data?>) -> Void

    private let session = URLSession.shared
    private let baseURL = URL(string: "https://jsonplaceholder.typicode.com")!

    func perform(_ request: APIRequest, _ completion: @escaping APIClientCompletion) {

        var urlComponents = URLComponents()
        urlComponents.scheme = baseURL.scheme
        urlComponents.host = baseURL.host
        urlComponents.path = baseURL.path
        urlComponents.queryItems = request.queryItems

        guard let url = urlComponents.url?.appendingPathComponent(request.path) else {
            completion(.failure(.invalidURL)); return
        }

        var urlRequest = URLRequest(url: url)
        urlRequest.httpMethod = request.method.rawValue
        urlRequest.httpBody = request.body

        request.headers?.forEach { urlRequest.addValue($0.value, forHTTPHeaderField: $0.field) }

        let task = session.dataTask(with: url) { (data, response, error) in
            guard let httpResponse = response as? HTTPURLResponse else {
                completion(.failure(.requestFailed)); return
            }
            completion(.success(APIResponse<Data?>(statusCode: httpResponse.statusCode, body: data)))
        }
        task.resume()
    }
}
```

示例：

```swift
struct Post: Decodable {
    let userId: Int
    let id: Int
    let title: String
    let body: String
}

let request = APIRequest(method: .get, path: "posts")

APIClient().perform(request) { (result) in
    switch result {
    case .success(let response):
        if let response = try? response.decode(to: [Post].self) {
            let posts = response.body
            print("Received posts: \(posts.first?.title ?? "")")
        } else {
            print("Failed to decode response")
        }
    case .failure:
        print("Error perform network request")
    }
}
```



上面的示例基本上够用，但是还是存在一些需要改进的地方：

- **`baseURL`** 可能需要更改
- **`method`** 可以设置一个默认值，比如 **`get`**
- 拦截器功能
- 对需要加入token的请求做特殊处理





文章来源

- [It's time to break up with your networking library for URLSession](https://tim.engineering/break-up-third-party-networking-urlsession/)



2019年05月30日11:53:57





























