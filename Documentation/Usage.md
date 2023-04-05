* [Introduction](#introduction)（）
简介
    - [Aside: The `AF` Namespace](#aside-the-af-namespace-and-reference)
    AF命名空间及引用
* [Making Requests](#making-requests)
发起请求
  + [HTTP Methods](#http-methods)
  HTTP方法
  + [Request Parameters and Parameter Encoders](#request-parameters-and-parameter-encoders)
  请求和参数编码器
    - [`URLEncodedFormParameterEncoder`](#urlencodedformparameterencoder)
    URL参数编码器
      * [GET Request With URL-Encoded Parameters](#get-request-with-url-encoded-parameters)
      带有URL编码参数的GET请求
      * [POST Request With URL-Encoded Parameters](#post-request-with-url-encoded-parameters)
      带有URL编码参数的POST请求
      * [Configuring the Sorting of Encoded Parameters](#configuring-the-sorting-of-encoded-parameters)
      配置编码参数排序
      * [Configuring the Encoding of `Array` Parameters](#configuring-the-encoding-of-array-parameters)
      配置Array参数的编码方式
      * [Configuring the Encoding of `Bool` Parameters](#configuring-the-encoding-of-bool-parameters)
      配置Bool参数的编码方式
      * [Configuring the Encoding of `Data` Parameters](#configuring-the-encoding-of-data-parameters)
      配置Data参数的编码方式
      * [Configuring the Encoding of `Date` Parameters](#configuring-the-encoding-of-date-parameters)
      配置Date参数的编码方式
      * [Configuring the Encoding of Coding Keys](#configuring-the-encoding-of-coding-keys)
      配置Codingkeys的编码方式
      * [Configuring the Encoding of Spaces](#configuring-the-encoding-of-spaces)
      配置空格的编码方式
    - [`JSONParameterEncoder`](#jsonparameterencoder)
    JSON参数编码器
      * [POST Request with JSON-Encoded Parameters](#post-request-with-json-encoded-parameters)
      带有JSON编码参数的POST请求
      * [Configuring a Custom `JSONEncoder`](#configuring-a-custom-jsonencoder)
      配置自定义JSON编码器
      * [Manual Parameter Encoding of a `URLRequest`](#manual-parameter-encoding-of-a-urlrequest)
      URLRequest的手动参数编码
  + [HTTP Headers](#http-headers)
  HTTP请求头
  + [Response Validation](#response-validation)
  响应验证
    - [Automatic Validation](#automatic-validation)
    自动验证
    - [Manual Validation](#manual-validation)
    手动验证
  + [Response Handling](#response-handling)
  响应处理
    - [Response Handler](#response-handler)
    响应处理处理器
    - [Response Data Handler](#response-data-handler)
    响应Data处理器
    - [Response String Handler](#response-string-handler)
    响应String处理器
    - [Response `Decodable` Handler](#response-decodable-handler)
    响应Decodable协议处理器
    - [Chained Response Handlers](#chained-response-handlers)
    响应链式处理器
    - [Response Handler Queue](#response-handler-queue)
    响应处理器队列
  + [Response Caching](#response-caching)
  响应缓存
  + [Authentication](#authentication)
  身份验证
    - [HTTP Basic Authentication](#http-basic-authentication)
    HTTP基本身份验证
    - [Authentication with `URLCredential`](#authentication-with-urlcredential)
    使用URLCredential进行身份验证
    - [Manual Authentication](#manual-authentication)
    手动身份验证
  + [Downloading Data to a File](#downloading-data-to-a-file)
  将数据下载到文件
    - [Download File Destination](#download-file-destination)
    下载文件位置
    - [Download Progress](#download-progress)
    下载进度
    - [Canceling and Resuming a Download](#canceling-and-resuming-a-download)
    取消并继续下载
  + [Uploading Data to a Server](#uploading-data-to-a-server)
  将数据上传到服务器
    - [Uploading Data](#uploading-data)
    上传数据
    - [Uploading a File](#uploading-a-file)
    上传文件
    - [Uploading Multipart Form Data](#uploading-multipart-form-data)
    上传多部分表单数据
    - [Upload Progress](#upload-progress)
    上传进度
  + [Streaming Data from a Server](#streaming-data-from-a-server)
  从服务器流式传输数据
    - [Streaming `Data`](#streaming-data)
    流式处理Data
    - [Streaming `String`s](#streaming-strings)
    流式处理String
    - [Streaming `Decodable` Values](#streaming-decodable-values)
    流式处理Decodable协议的值
    - [Producing an `InputStream`](#producing-an-inputstream)
    生成InputStream
  + [Statistical Metrics](#statistical-metrics)
  统计指标
    - [`URLSessionTaskMetrics`](#urlsessiontaskmetrics)
  + [cURL Command Output](#curl-command-output)
  cURL命令输出

# Using Alamofire
使用Alamofire
## Introduction
引言
Alamofire provides an elegant and composable interface to HTTP network requests. It does not implement its own HTTP networking functionality. Instead it builds on top of Apple's [URL Loading System](https://developer.apple.com/documentation/foundation/url_loading_system/) provided by the Foundation framework. At the core of the system is [`URLSession`](https://developer.apple.com/documentation/foundation/urlsession) and the [`URLSessionTask`](https://developer.apple.com/documentation/foundation/urlsessiontask) subclasses. Alamofire wraps these APIs, and many others, in an easier to use interface and provides a variety of functionality necessary for modern application development using HTTP networking. However, it's important to know where many of Alamofire's core behaviors come from, so familiarity with the URL Loading System is important. Ultimately, the networking features of Alamofire are limited by the capabilities of that system, and the behaviors and best practices should always be remembered and observed.
Alamofire为HTTP网络请求提供了一个优雅且可组合的接口。它没有实现自己的HTTP网络功能，相反，它建立在苹果的[URL加载系统]之上(https://developer.apple.com/documentation/foundation/url_loading_system/)，由Foundation框架提供。系统的核心是[`URLSession`](https://developer.apple.com/documentation/foundation/urlsession)和[`URLSessionTask`](https://developer.apple.com/documentation/foundation/urlsessiontask)子类。Alamofire将这些API和许多其他API封装在一个更易于使用的界面中，并提供了使用HTTP网络进行现代应用程序开发的各种功能。然而，了解Alamofire的许多核心行为来自哪里很重要，因此熟悉URL加载系统很重要。最终，Alamofire的网络功能受到该系统功能的限制，应该始终记住和遵守其行为和最佳实践。

Additionally, networking in Alamofire (and the URL Loading System in general) is done _asynchronously_. Asynchronous programming may be a source of frustration to programmers unfamiliar with the concept, but there are [very good reasons](https://developer.apple.com/library/ios/qa/qa1693/_index.html) for doing it this way.
此外，Alamofire（以及通常的URL加载系统）中的网络是异步完成。异步编程可能让不熟悉这个概念的程序员感到沮丧，但有[非常好的理由](https://developer.apple.com/library/ios/qa/qa1693/_index.html)来这么做。

#### Aside: The `AF` Namespace and Reference
AF命名空间和引用
Previous versions of Alamofire's documentation used examples like `Alamofire.request()`. This API, while it appeared to require the `Alamofire` prefix, in fact worked fine without it. The `request` method and other functions were available globally in any file with `import Alamofire`. Starting in Alamofire 5, this functionality has been removed and instead the `AF` global is a reference to `Session.default`. This allows Alamofire to offer the same convenience functionality while not having to pollute the global namespace every time Alamofire is used and not having to duplicate the `Session` API globally. Similarly, types extended by Alamofire will use an `af` property extension to separate the functionality Alamofire adds from other extensions.
Alamofire文档的早期版本使用了诸如`Alamofire.request()`之类的示例。虽然此API似乎需要`Alamofire`前缀，但是事实上没有它也能正常工作。`request`方法和其他函数在任何带有`import Alamofire`的文件中都可以全局使用。从Alamofire5开始，该功能已被删除，取而代之的是`AF`全局引用`Session.default`。这使得Alamofire能够提供相同的便利功能，而不必每次使用Alamofire时污染全局命名空间，也不必全局复制`Session`API。类似地，Alamofire扩展的类型将使用`af`属性扩展来将Alamofire添加的功能与其他扩展分开。

## Making Requests
创建请求
Alamofire provides a variety of convenience methods for making HTTP requests. At the simplest, just provide a `String` that can be converted into a `URL`:
Alamofire提供了各种方便的方法来进行HTTP请求。最简单的是，只需提供一个可以转换为`URL`的`String`：

```swift
AF.request("https://httpbin.org/get").response { response in
    debugPrint(response)
}
```

> All examples require `import Alamofire` somewhere in the source file.
所有示例都要求在源文件中的某个位置使用`import Alamofire`。
> For examples of use with Swift's `async`-`await` syntax, see our [Advanced Usage](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#using-alamofire-with-swift-concurrency) documentation.
有关Swift的`async`-`await`语法，请参阅我们的[高级用法](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#using-alamofire-with-swift-concurrency)文档。
This is actually one form of the two top-level APIs on Alamofire's `Session` type for making requests. Its full definition looks like this:
这实际上是Alamofire的`Session`类型上用于发出请求的两个顶级API的一种形式。它的完整定义如下：
```swift
open func request<Parameters: Encodable>(_ convertible: URLConvertible,
                                         method: HTTPMethod = .get,
                                         parameters: Parameters? = nil,
                                         encoder: ParameterEncoder = URLEncodedFormParameterEncoder.default,
                                         headers: HTTPHeaders? = nil,
                                         interceptor: RequestInterceptor? = nil) -> DataRequest
```
This method creates a `DataRequest` while allowing the composition of requests from individual components, such as the `method` and `headers`, while also allowing per-request `RequestInterceptor`s and `Encodable` parameters.
此方法创建`DataRequest`，同时允许组合来自单个组件的请求，如`method`和`headers`，同时还允许每个请求的`RequestInterceptor`和`Encodable`参数。

> There are additional methods that allow you to make requests using `Parameters` dictionaries and `ParameterEncoding` types. This API is no longer recommended and will eventually be deprecated and removed from Alamofire.
还有其他方法允许您使用`Parameters`字典和`ParameterEncoding`类型进行请求。不再推荐使用此API，最终将被启用并从Alamofire删除。

The second version of this API is much simpler:
此API的第二个版本简单得多：

```swift
open func request(_ urlRequest: URLRequestConvertible, 
                  interceptor: RequestInterceptor? = nil) -> DataRequest
```

This method creates a `DataRequest` for any type conforming to Alamofire's `URLRequestConvertible` protocol. All of the different parameters from the previous version are encapsulated in that value, which can give rise to very powerful abstractions. This is discussed in our [Advanced Usage](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md) documentation.
该方法为符合Alamofire的`URLRequestConvertible`协议的任何类型创建`DataRequest`。以前版本中的所有不同参数都封装在该值中，这可以产生非常强大的抽象。这在我们的[高级用法](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md)文档中进行了讨论。

### HTTP Methods
HTTP方法

The `HTTPMethod` type lists the HTTP methods defined in [RFC 7231 §4.3](https://tools.ietf.org/html/rfc7231#section-4.3):
`HTTPMethod`类型列出了[RFC 7231]中定义的HTTP方法(https://tools.ietf.org/html/rfc7231#section-4.3)：

```swift
public struct HTTPMethod: RawRepresentable, Equatable, Hashable {
    public static let connect = HTTPMethod(rawValue: "CONNECT")
    public static let delete = HTTPMethod(rawValue: "DELETE")
    public static let get = HTTPMethod(rawValue: "GET")
    public static let head = HTTPMethod(rawValue: "HEAD")
    public static let options = HTTPMethod(rawValue: "OPTIONS")
    public static let patch = HTTPMethod(rawValue: "PATCH")
    public static let post = HTTPMethod(rawValue: "POST")
    public static let put = HTTPMethod(rawValue: "PUT")
    public static let query = HTTPMethod(rawValue: "QUERY")
    public static let trace = HTTPMethod(rawValue: "TRACE")

    public let rawValue: String

    public init(rawValue: String) {
        self.rawValue = rawValue
    }
}
```

These values can be passed as the `method` argument to the `AF.request` API:
这些值可以作为`method`参数传递给`AF.request`API：

```swift
AF.request("https://httpbin.org/get")
AF.request("https://httpbin.org/post", method: .post)
AF.request("https://httpbin.org/put", method: .put)
AF.request("https://httpbin.org/delete", method: .delete)
```

It's important to remember that the different HTTP methods may have different semantics and require different parameter encodings depending on what the server expects. For instance, passing body data in a `GET` request is not supported by `URLSession` or Alamofire and will return an error.
重要的是要记住，不同的HTTP方法可能具有不同的语义，并且根据服务器的期望需要不同的参数编码。例如，`URLSession`或Alamofire不支持在`GET`请求中传递正文数据，并且会返回错误。

Alamofire also offers an extension on `URLRequest` to bridge the `httpMethod` property that returns a `String` to an `HTTPMethod` value:
Alamofire还提供了对`URLRequest`的扩展，将返回`String`的`httpMethod`属性桥接为返回`HTTPMethod`的值

```swift
extension URLRequest {
    /// Returns the `httpMethod` as Alamofire's `HTTPMethod` type.
    public var method: HTTPMethod? {
        get { httpMethod.flatMap(HTTPMethod.init) }
        set { httpMethod = newValue?.rawValue }
    }
}
```

If you need to use an HTTP method that Alamofire's `HTTPMethod` type doesn't support, you can extend the type to add your custom values:
如果你需要使用Alamofire的`HTTPMethod`不支持的HTTP方法，你可以扩展该类型以添加自定义值：

```swift
extension HTTPMethod {
    static let custom = HTTPMethod(rawValue: "CUSTOM")
}

AF.request("https://httpbin.org/headers", method: .custom)
```

### Setting Other `URLRequest` Properties
设置其他`URLRequest`属性
Alamofire's request creation methods offer the most common parameters for customization but sometimes those just aren't enough. The `URLRequest`s created from the passed values can be modified by using a `RequestModifier` closure when creating requests. For example, to set the `URLRequest`'s `timeoutInterval` to 5 seconds, modify the request in the closure.
Alamofire的请求创建方法提供了最常见的自定义参数，但有时这些参数还不够。根据传递的值创建的`URLRequest`可以在创建请求时使用`RequestModifier`闭包进行修改。例如，要将`URLRequest`的`timeoutInterval`设置为5秒，可以修改闭包中的请求。

```swift
AF.request("https://httpbin.org/get", requestModifier: { $0.timeoutInterval = 5 }).response(...)
```

`RequestModifier`s also work with trailing closure syntax.
`RequestModifier`也适用于尾随闭包语法。

```swift
AF.request("https://httpbin.org/get") { urlRequest in
    urlRequest.timeoutInterval = 5
    urlRequest.allowsConstrainedNetworkAccess = false
}
.response(...)
```

`RequestModifier`s only apply to request created using methods taking a `URL` and other individual components, not to values created directly from `URLRequestConvertible` values, as those values should be able to set all parameters themselves. Additionally, adoption of `URLRequestConvertible` is recommended once *most* requests start needing to be modified during creation. You can read more in our [Advanced Usage documentation](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#making-requests).
`RequestModifier`仅适用于使用采用`URL`和其他单独组件的方法创建的请求，而不适用于直接从“URLRequestConvertible”值创建的值，因为这些值应该能够自己设置所有参数。此外，一旦*大多数*请求在创建过程中开始需要修改，建议采用“URLRequestConvertible”。您可以在我们的[高级用法文档]中阅读更多信息(https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#making-请求）。


### Request Parameters and Parameter Encoders
请求参数和参数编码器

Alamofire supports passing any `Encodable` type as the parameters of a request. These parameters are then passed through a type conforming to the `ParameterEncoder` protocol and added to the `URLRequest` which is then sent over the network. Alamofire includes two `ParameterEncoder` conforming types: `JSONParameterEncoder` and `URLEncodedFormParameterEncoder`. These types cover the most common encodings used by modern services (XML encoding is left as an exercise for the reader).
Alamofire支持传递任何`Encodable`类型作为请求的参数。然后，这些参数通过符合`ParameterEncoder`协议的类型传递，并添加到`URLRequest`中，然后通过网络发送。Alamofire包括两种符合`ParameterEncoder`的类型：`JSONParameterEncoder`和`URLEncodedFormParameterEncoder`。这些类型涵盖了现代服务使用的最常见的编码（XML编码留给读者练习）。

```swift
struct Login: Encodable {
    let email: String
    let password: String
}

let login = Login(email: "test@test.test", password: "testPassword")

AF.request("https://httpbin.org/post",
           method: .post,
           parameters: login,
           encoder: JSONParameterEncoder.default).response { response in
    debugPrint(response)
}
```

#### `URLEncodedFormParameterEncoder`

The `URLEncodedFormParameterEncoder` encodes values into a url-encoded string to be set as or appended to any existing URL query string or set as the HTTP body of the request. Controlling where the encoded string is set can be done by setting the `destination` of the encoding. The `URLEncodedFormParameterEncoder.Destination` enumeration has three cases:
`URLEncodedFormParameterEncoder`将值编码为url编码的字符串，该字符串将被设置为或附加到任何现有的url查询字符串，或被设置为请求的HTTP主体。可以通过设置编码的`destination`来控制编码字符串的设置位置。`URLEncodedFormParameterEncoder.Destination `枚举有三种情况：

- `.methodDependent` - Applies the encoded query string result to existing query string for `.get`, `.head` and `.delete` requests and sets it as the HTTP body for requests with any other HTTP method.
`.methodDependent` - 将编码的查询字符串结果应用于`.get`、`.head`和`.delete`请求的现有查询字符串，或将其设置为使用任何其他HTTP方法的请求的HTTP正文。
- `.queryString` - Sets or appends the encoded string to the query of the request's `URL`.
`.queryString` - 将编码字符串设置或附加到请求的`URL`的查询中。
- `.httpBody` - Sets the encoded string as the HTTP body of the `URLRequest`.
`.httpBody` - 将编码字符串设置为`URLRequest `的HTTP正文。

The `Content-Type` HTTP header of an encoded request with HTTP body is set to `application/x-www-form-urlencoded; charset=utf-8`, if `Content-Type` is not already set.
具有HTTP主体的编码请求的`Content-Type`HTTP头如果尚未设置，则会被设置为`application/x-www-form-urlencoded;charset=utf-8`。

Internally, `URLEncodedFormParameterEncoder` uses `URLEncodedFormEncoder` to perform the actual encoding from an `Encodable` type to a URL encoded form `String`. This encoder can be used to customize the encoding for various types, including `Array` using the `ArrayEncoding`, `Bool` using the `BoolEncoding`, `Data` using the `DataEncoding`, `Date` using the `DateEncoding`, coding keys using the `KeyEncoding`, and spaces using the `SpaceEncoding`.
在内部，`URLEncodedFormParameterEncoder`使用`URLEncoedFormEncoder'执行从`Encodable`类型到URL编码形式`String`的实际编码。该编码器可用于自定义各种类型的编码，包括使用`ArrayEncoding`的`Array`、使用`BoolEncoding`的`Bool`、使用`DataEncoding`的`Data`、使用`DateEncoding`的`Date`、使用`KeyEncoding`的编码键以及使用`SpaceEncoding`的空格。

##### GET Request With URL-Encoded Parameters
带有URL编码参数的GET请求

```swift
let parameters = ["foo": "bar"]

// All three of these calls are equivalent
AF.request("https://httpbin.org/get", parameters: parameters) // encoding defaults to `URLEncoding.default`
AF.request("https://httpbin.org/get", parameters: parameters, encoder: URLEncodedFormParameterEncoder.default)
AF.request("https://httpbin.org/get", parameters: parameters, encoder: URLEncodedFormParameterEncoder(destination: .methodDependent))

// https://httpbin.org/get?foo=bar
```

##### POST Request With URL-Encoded Parameters
带有URL编码参数的POST请求

```swift
let parameters: [String: [String]] = [
    "foo": ["bar"],
    "baz": ["a", "b"],
    "qux": ["x", "y", "z"]
]

// All three of these calls are equivalent
AF.request("https://httpbin.org/post", method: .post, parameters: parameters)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: URLEncodedFormParameterEncoder.default)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: URLEncodedFormParameterEncoder(destination: .httpBody))

// HTTP body: "qux[]=x&qux[]=y&qux[]=z&baz[]=a&baz[]=b&foo[]=bar"
```

#### Configuring the Sorting of Encoded Values
配置编码值的排序

Since Swift 4.2, the hashing algorithm used by Swift's `Dictionary` type produces a random internal ordering at runtime which differs between app launches. This can cause encoded parameters to change order, which may have an impact on caching and other behaviors. By default `URLEncodedFormEncoder` will sort its encoded key-value pairs. While this produces constant output for all `Encodable` types, it may not match the actual encoding order implemented by the type. You can set `alphabetizeKeyValuePairs` to `false` to return to implementation order, though that will also have the randomized `Dictionary` order as well.
自Swift 4.2以来，Swift的`Dictionary`类型使用的哈希算法在运行时产生随机的内部排序，这在每次应用程序启动时都有所不同。这可能会导致编码参数改变顺序，这可能会对缓存和其他行为产生影响。默认情况下，`URLEncodedFormEncoder`将对其编码的键值对进行排序。虽然这会为所有`Encodable`类型产生恒定的输出，但它可能与该类型实现的实际编码顺序不匹配。您可以将`alphabetizeKeyValuePairs`设置为`false`以返回到实现顺序，尽管这也将具有随机化的`Dictionary`顺序。

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `alphabetizeKeyValuePairs` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder`的初始值设定项中指定所需的`alphabetizeKeyValuePairs`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(alphabetizeKeyValuePairs: false))
```

##### Configuring the Encoding of `Array` Parameters
配置`Array`参数的编码

Since there is no published specification for how to encode collection types, by default Alamofire follows the convention of appending `[]` to the key for array values (`foo[]=1&foo[]=2`), and appending the key surrounded by square brackets for nested dictionary values (`foo[bar]=baz`).
由于没有发布关于如何对集合类型进行编码的规范，默认情况下，Alamofire遵循的惯例是将`[]`附加到数组值的键上（`foo[]=1&foo[]=2`），并将嵌套字典值的键附加到由方括号包围的键（`foo[bar]=baz`）。

The `URLEncodedFormEncoder.ArrayEncoding` enumeration provides the following methods for encoding `Array` parameters:
`URLEncodedFormEncoder.ArrayEncoding`枚举提供了以下用于编码`Array`参数的方法：

- `.brackets` - An empty set of square brackets is appended to the key for every value. This is the default case.
`.括号` - 每个值的键后面都会附加一组空的方括号。这是默认情况。
- `.noBrackets` - No brackets are appended. The key is encoded as is.
`.No括号` - 没有附加括号。密钥按原样进行编码。

By default, Alamofire uses the `.brackets` encoding, where `foo = [1, 2]` is encoded as `foo[]=1&foo[]=2`.
默认情况下，Alamofire使用`.brackets`编码，其中`foo=[1，2]`编码为`foo[]=1&foo[]=2`。

Using the `.noBrackets` encoding will encode `foo = [1, 2]` as `foo=1&foo=2`.
使用`.noBrackets`编码将`foo=[1，2]`编码为`foo=1&foo=2`。

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `ArrayEncoding` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder`的初始值设定项中指定所需的`ArrayEncoding`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(arrayEncoding: .noBrackets))
```

##### Configuring the Encoding of `Bool` Parameters
配置`Bool`参数的编码

The `URLEncodedFormEncoder.BoolEncoding` enumeration provides the following methods for encoding `Bool` parameters:
`URLEncodedFormEncoder.BoolEncoding`枚举提供了以下用于编码`Bool`参数的方法：

- `.numeric` - Encode `true` as `1` and `false` as `0`. This is the default case.
`.numeric` - 将`true`编码为`1`，将`false`编码为`0`。这是默认情况。
- `.literal` - Encode `true` and `false` as string literals.
`.literal`-将`true`和`false`编码为字符串文字。

By default, Alamofire uses the `.numeric` encoding.
默认情况下，Alamofire使用`.numeric`编码。

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `BoolEncoding` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder`的初始值设定项中指定所需的`BoolEncoding`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(boolEncoding: .numeric))
```

##### Configuring the Encoding of `Data` Parameters
配置`Data`参数的编码

`DataEncoding` includes the following methods for encoding `Data` parameters:
`DataEncoding`包括以下用于对`Data`参数进行编码的方法：

- `.deferredToData` - Uses `Data`'s native `Encodable` support.
`.deferredToData`-使用`Data`的本机`Encodable`支持。
- `.base64` - Encodes `Data` as a Base 64 encoded `String`. This is the default case.
`.base64`-将`Data`编码为base64编码的`String`。这是默认情况。
- `.custom((Data) -> throws -> String)` - Encodes `Data` using the given closure.
`.custom((Data) -> throws -> String)`-使用给定的闭包对`Data'进行编码。

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `DataEncoding` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder`的初始值设定项中指定所需的`DataEncoding`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(dataEncoding: .base64))
```

##### Configuring the Encoding of `Date` Parameters
配置`Date`参数的编码

Given the sheer number of ways to encode a `Date` into a `String`, `DateEncoding` includes the following methods for encoding `Date` parameters:

- `.deferredToDate` - Uses `Date`'s native `Encodable` support. This is the default case.
`.deferredToDate` - 使用`Date`的本机`Encodable`支持。这是默认情况。
- `.secondsSince1970` - Encodes `Date`s as seconds since midnight UTC on January 1, 1970. 
`.secondsSinc1970 ` - 将`Date`编码为自1970年1月1日UTC午夜以来的秒。
- `.millisecondsSince1970` - Encodes `Date`s as milliseconds since midnight UTC on January 1, 1970.
`.millisecondsSinc1970 ` - 将`Date`编码为自1970年1月1日UTC午夜以来的毫秒。
- `.iso8601` - Encodes `Date`s according to the ISO 8601 and RFC3339 standards.
`.iso8601` - 根据ISO 8601和RFC3339标准对`日期`进行编码。
- `.formatted(DateFormatter)` - Encodes `Date`s using the given `DateFormatter`.
`.formated（DateFormatter）` - 使用给定的`DateFormatter`对`Date`进行编码。
- `.custom((Date) throws -> String)` - Encodes `Date`s using the given closure.
`.custom（（Date）throws->String）`-使用给定的闭包对`Date'进行编码。

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `DateEncoding` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder`的初始值设定项中指定所需的`DateEncoding`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(dateEncoding: .iso8601))
```

##### Configuring the Encoding of Coding Keys
配置编码键的编码

Due to the variety of parameter key styles, `KeyEncoding` provides the following methods to customize key encoding from keys in `lowerCamelCase`:
由于参数键样式的多样性，`KeyEncoding`提供了以下方法来从`lowerCamelCase`中的键自定义键编码：

- `.useDefaultKeys` - Uses the keys specified by each type. This is the default case.
`.useDefaultKeys`-使用每种类型指定的键。这是默认情况。
- `.convertToSnakeCase` - Converts keys to snake case: `oneTwoThree` becomes `one_two_three`.
`.convertToSnakeCase`-将密钥转换为蛇型：`oneTwoThree`变为`one_two_three`。
- `.convertToKebabCase` - Converts keys to kebab case: `oneTwoThree` becomes `one-two-three`.
`.convertToKebabCase`-将键转换为kebab型：`oneTwoThree`变为`one-two-three`。
- `.capitalized` - Capitalizes the first letter only, a.k.a `UpperCamelCase`: `oneTwoThree` becomes `OneTwoThree`.
`.大写`-仅将第一个字母大写，也称为`UpperCamelCase`：`oneTwoThree`变为`OneTwoThree。
- `.uppercased` - Uppercases all letters: `oneTwoThree` becomes `ONETWOTHREE`.
`.大写`-大写所有字母：`oneTwoThree`变为`ONETOWTHREE`。
- `.lowercased` - Lowercases all letters: `oneTwoThree` becomes `onetwothree`.
`.小写`-所有字母的小写：`oneTwoThree`变为`onetwothree`。
- `.custom((String) -> String)` - Encodes keys using the given closure.
`.custom（（String）->String）`-使用给定的闭包对键进行编码。

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `KeyEncoding` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder`的初始值设定项中指定所需的`KeyEncoding`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(keyEncoding: .convertToSnakeCase))
```

#### Configuring the Encoding of Object Key Paths
配置对象键路径的编码

Nest object key paths are typically encoded using brackets (e.g. `parent[child][grandchild]`). Alamofire provides the `KeyPathEncoding` to customize that behavior.
嵌套对象键路径通常使用括号进行编码（例如`parent[child][grandschild]`）。Alamofire提供了`KeyPathEncoding`来自定义该行为。


- `.brackets` - Wraps each sub-key in the key path in brackets. e.g `parent[child][grandchild]`.
`.括号`-将密钥路径中的每个子密钥括在括号中。例如`parent[child][grandschild]`。
- `.dots` - Separates each sub-key in the key path with dots. e.g. `parent.child.grandchild`.
`.dots`-用点分隔键路径中的每个子键。例如`parent.child.sgrandschild`。

Additionally, you can create your own encoding by creating an instance with a custom encoding closure. For example, `KeyPathEncoding { "-\($0)" }` will separate each sub-key path with hyphens. e.g. `parent-child-grandchild`.
此外，您可以通过创建一个带有自定义编码闭包的实例来创建自己的编码。例如，`KeyPathEncoding｛“-\（$0）”｝`将用连字符分隔每个子键路径。例如`parent-child-grandchild`。

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `KeyPathEncoding` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder`的初始值设定项中指定所需的`KeyPathEncoding`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(keyPathEncoding: .brackets))
```

##### Configuring the Encoding of Spaces
配置空格编码

Older form encoders used `+` to encode spaces and some servers still expect this encoding instead of the modern percent encoding, so Alamofire includes the following methods for encoding spaces:
旧形式的编码器使用`+`来编码空格，一些服务器仍然期望这种编码而不是现代的百分号编码，因此Alamofire包括以下用于编码空格的方法：

- `.percentEscaped` - Encodes space characters by applying standard percent escaping. `" "` is encoded as  `"%20"`. This is the default case.
`.percentEscaped` - 通过应用标准的百分比转义对空格字符进行编码`" "`被编码为`"%20"`。这是默认情况。

- `.plusReplaced` - Encodes space characters by replacing them with `+`. `" "` is encoded as `"+"`.
`.plusReplaced` - 通过将空格字符替换为`+`对其进行编码`" "`被编码为`"+"`。

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `SpaceEncoding` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder'的初始值设定项中指定所需的`SpaceEncoding`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(spaceEncoding: .plusReplaced))
```

##### Configuring the Encoding of Optionals
配置可选的编码

There is no standard for encoding `Optional` values as part of form data. Nonetheless, Alamofire provides `NilEncoding` with the following methods for encoding optionals:
没有将`Optional`值作为表单数据的一部分进行编码的标准。尽管如此，Alamofire为`NilEncoding`提供了以下编码选项的方法：

- `.dropKey` - Encodes `nil` values by dropping them from the output entirely. This matches other Swift encoders. e.g. `otherValue=2`.
`.dropKey` - 通过从输出中完全删除`nil`值来对其进行编码。这与其他Swift编码器相匹配。例如`otherValue=2`。
- `.dropValue` - Encodes `nil` values by dropping the value from the output. e.g. `nilValue=&otherValue=2`.
`.dropValue` - 通过从输出中删除值来对`nil`值进行编码。例如`nilValue=&otherValue=2 `。
- `.null` - Encodes `nil` values as the string `null`. e.g. `nilValue=null&otherValue=2`.
`.null` - 将`nil`值编码为字符串`null`。例如`nilValue=null&otherValue=2`。


Additionally, custom encodings can be created by specifying an encoding closure that provides the `nil` replacement value.
此外，可以通过指定提供`nil`替换值的编码闭包来创建自定义编码。

```swift
extension URLEncodedFormEncoder.NilEncoding {
  static let customEncoding = NilEncoding { "customNilValue" }
}
```

You can create your own `URLEncodedFormParameterEncoder` and specify the desired `NilEncoding` in the initializer of the passed `URLEncodedFormEncoder`:
您可以创建自己的`URLEncodedFormParameterEncoder`，并在传递的`URLEncodedFormEncoder`的初始值设定项中指定所需的`NilEncoding`：

```swift
let encoder = URLEncodedFormParameterEncoder(encoder: URLEncodedFormEncoder(nilEncoding: .dropKey))
```

#### `JSONParameterEncoder`

`JSONParameterEncoder` encodes `Encodable` values using Swift's `JSONEncoder` and sets the result as the `httpBody` of the `URLRequest`. The `Content-Type` HTTP header field of an encoded request is set to `application/json` if not already set.
`JSONParameterEncoder`使用Swift的`JSONEncoder`对`Encodable`值进行编码，并将结果设置为`URLRequest`的`httpBody`。编码请求的`Content-Type`HTTP头字段如果尚未设置，则设置为`application/json`。

##### POST Request with JSON-Encoded Parameters

```swift
let parameters: [String: [String]] = [
    "foo": ["bar"],
    "baz": ["a", "b"],
    "qux": ["x", "y", "z"]
]

AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.default)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.prettyPrinted)
AF.request("https://httpbin.org/post", method: .post, parameters: parameters, encoder: JSONParameterEncoder.sortedKeys)

// HTTP body: {"baz":["a","b"],"foo":["bar"],"qux":["x","y","z"]}
```

##### Configuring a Custom `JSONEncoder`

You can customize the behavior of `JSONParameterEncoder` by passing it a `JSONEncoder` instance configured to your needs:
您可以自定义`JSONParameterEncoder`的行为，方法是向其传递一个根据您的需要配置的`JSONEncoder`实例：

```swift
let encoder = JSONEncoder()
encoder.dateEncoding = .iso8601
encoder.keyEncodingStrategy = .convertToSnakeCase
let parameterEncoder = JSONParameterEncoder(encoder: encoder)
```

##### Manual Parameter Encoding of a `URLRequest`
`URLRequest`的手动参数编码

The `ParameterEncoder` APIs can also be used outside of Alamofire by encoding parameters directly in `URLRequest`s.
`ParameterEncoder`API也可以在Alamofire之外使用，直接在`URLRequest`中对参数进行编码。

```swift
let url = URL(string: "https://httpbin.org/get")!
var urlRequest = URLRequest(url: url)

let parameters = ["foo": "bar"]
let encodedURLRequest = try URLEncodedFormParameterEncoder.default.encode(parameters, 
                                                                          into: urlRequest)
```

### HTTP Headers
HTTP请求头
Alamofire includes its own `HTTPHeaders` type, an order-preserving and case-insensitive representation of HTTP header name / value pairs. The `HTTPHeader` types encapsulate a single name / value pair and provides a variety of static values for common headers.
Alamofire包括自己的`HTTPHeaders`类型，这是一种保留顺序且不区分大小写的HTTP标头名称/值对表示。`HTTPHeader`类型封装了单个名称/值对，并为公共请求头提供了各种静态值。

Adding custom `HTTPHeaders` to a `Request` is as simple as passing a value to one of the `request` methods:
将自定义`HTTPHeaders`添加到`Request`中，就像将值传递给`Request`方法之一一样简单：

```swift
let headers: HTTPHeaders = [
    "Authorization": "Basic VXNlcm5hbWU6UGFzc3dvcmQ=",
    "Accept": "application/json"
]

AF.request("https://httpbin.org/headers", headers: headers).responseDecodable(of: DecodableType.self) { response in
    debugPrint(response)
}
```

`HTTPHeaders` can also be constructed from an array of `HTTPHeader` values:
`HTTPHeaders`也可以由`HTTPHeader`值的数组构造：

```swift
let headers: HTTPHeaders = [
    .authorization(username: "Username", password: "Password"),
    .accept("application/json")
]

AF.request("https://httpbin.org/headers", headers: headers).responseDecodable(of: DecodableType.self) { response in
    debugPrint(response)
}
```

> For HTTP headers that do not change, it is recommended to set them on the `URLSessionConfiguration` so they are automatically applied to any `URLSessionTask` created by the underlying `URLSession`. For more information, see the [Session Configurations](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#creating-a-session-with-a-urlsessionconfiguration) section.
对于不更改的HTTP请求头，建议在`URLSessionConfiguration`上设置它们，以便它们自动应用于由基础`URLSession`创建的任何`URLSessionTask`。有关更多信息，请参阅[会话配置](https://github.com/Alamofire/Alamofire/blob/master/Documentation/AdvancedUsage.md#creating-具有url会话配置的会话）部分。

The default Alamofire `Session` provides a default set of headers for every `Request`. These include:
默认的Alamofire`Session`为每个`Request`提供一组默认的头。其中包括：

- `Accept-Encoding`, which defaults to `br;q=1.0, gzip;q=0.8, deflate;q=0.6`, per [RFC 7230 §4.2.3](https://tools.ietf.org/html/rfc7230#section-4.2.3).
`Accept-Encoding`，默认为`br;q=1.0，gzip;q=0.8，defalte;q=0.6`，根据[RFC 7230§4.2.3](https://tools.ietf.org/html/rfc7230#section-4.2.3).
- `Accept-Language`, which defaults to up to the top 6 preferred languages on the system, formatted like `en;q=1.0`, per [RFC 7231 §5.3.5](https://tools.ietf.org/html/rfc7231#section-5.3.5).
`Accept-Language`，默认为系统中前6种首选语言，格式类似于`en;q=1.0`，根据[RFC 7231§5.3.5](https://tools.ietf.org/html/rfc7231#section-5.3.5).
- `User-Agent`, which contains versioning information about the current app. For example: `iOS Example/1.0 (com.alamofire.iOS-Example; build:1; iOS 13.0.0) Alamofire/5.0.0`, per [RFC 7231 §5.5.3](https://tools.ietf.org/html/rfc7231#section-5.5.3).
`User-Agent`，其中包含有关当前应用程序的版本控制信息。例如：`iOS example/1.0（com.alamofire.iOS-example; build:1; iOS 13.0.0）alamofire/5.0`，根据[RFC 7231§5.5.3](https://tools.ietf.org/html/rfc7231#section-5.5.3).



If you need to customize these headers, a custom `URLSessionConfiguration` should be created, the `headers` property updated, and the configuration applied to a new `Session` instance. Use `URLSessionConfiguration.af.default` to customize your configuration while keeping Alamofire's default headers.
如果需要自定义这些请求头，则应创建自定义的`URLSessionConfiguration`，更新`headers`属性，并将配置应用于新的`Session`实例。使用`URLSessionConfiguration.af.default`自定义您的配置，同时保留Alamofire的默认请求头。

### Response Validation
响应验证

By default, Alamofire treats any completed request to be successful, regardless of the content of the response. Calling `validate()` before a response handler causes an error to be generated if the response had an unacceptable status code or MIME type.
默认情况下，无论响应的内容如何，Alamofire都会将任何已完成的请求视为成功。如果响应具有不可接受的状态代码或MIME类型，则在响应处理程序之前调用`validate()`会导致生成错误。

#### Automatic Validation
自动验证

The `validate()` API automatically validates that status codes are within the `200..<300` range, and that the `Content-Type` header of the response matches the `Accept` header of the request, if one is provided.
`validate()`API会自动验证状态代码是否在`200..<300`范围内，以及响应的`Content Type`标头是否与请求的`Accept`标头匹配（如果提供）。

```swift
AF.request("https://httpbin.org/get").validate().responseData { response in
    debugPrint(response)
}
```

#### Manual Validation
手动验证

```swift
AF.request("https://httpbin.org/get")
    .validate(statusCode: 200..<300)
    .validate(contentType: ["application/json"])
    .responseData { response in
        switch response.result {
        case .success:
            print("Validation Successful")
        case let .failure(error):
            print(error)
        }
    }
```

### Response Handling
响应处理

Alamofire's `DataRequest` and `DownloadRequest` both have a corresponding response type: `DataResponse<Success, Failure: Error>` and `DownloadResponse<Success, Failure: Error>`. Both of these are composed of two generics: the serialized type and the error type. By default, all response values will produce the `AFError` error type (i.e. `DataResponse<Success, AFError>`). Alamofire uses the simpler `AFDataResponse<Success>` and `AFDownloadResponse<Success>`, in its public API, which always have `AFError` error types. `UploadRequest`, a subclass of `DataRequest`, uses the same `DataResponse` type.
Alamofire的`DataRequest`和`DownloadRequest`都有相应的响应类型：`DataResponse＜Success，Failure:Error＞`和`DownloadResponse＜Success，Failure:Error＞`。这两者都由两个泛型组成：序列化类型和错误类型。默认情况下，所有响应值都将产生`AFError`错误类型（即`DataResponse＜Success，AFError＞`）。Alamofire在其公共API中使用更简单的`AFDataResponse＜Success＞`和`AFDownloadResponse＜Success＞`，它们总是有`AFError`错误类型。`UploadRequest`是`DataRequest`的子类，使用相同的`DataResponse`类型。

Handling the `DataResponse` of a `DataRequest` or `UploadRequest` made in Alamofire involves chaining a response handler like `responseDecodable` onto the `DataRequest`:
处理Alamofire中`DataRequest`或`UploadRequest`生成的`DataResponse`涉及将`responseDecodable`之类的响应处理程序链接到`DataRequest`上：

```swift
AF.request("https://httpbin.org/get").responseDecodable(of: DecodableType.self) { response in
    debugPrint(response)
}
```

In the above example, the `responseDecodable` handler is added to the `DataRequest` to be executed once the `DataRequest` is complete. The closure passed to the handler receives the `DataResponse<DecodableType, AFError>` value produced by the `DecodableResponseSerializer` from the `URLRequest`, `HTTPURLResponse`, `Data`, and `Error` produced by the request.
在上面的例子中，一旦`DataRequest`完成，就将`responseDecodable`处理程序添加到要执行的`DataRequest`中。传递给处理程序的闭包接收`DecodeableResponseSerializer`中产生的`DataResponse＜DecodeableType，AFError＞`值，`DecodeableResponseSerializer`是由`URLRequest`、`HTTPURResponse`、`Data`和`Error`产生的，它们则是由请求产生的。


Rather than blocking execution to wait for a response from the server, this closure is added as a [callback](https://en.wikipedia.org/wiki/Callback_%28computer_programming%29) to handle the response once it's received. The result of a request is only available inside the scope of a response closure. Any execution contingent on the response or data received from the server must be done within a response closure.
这个闭包不是阻塞执行以等待服务器的响应，而是作为[回调](https://en.wikipedia.org/wiki/Callback_%28computer_programming%29)添加的，以在收到响应后进行处理。请求的结果仅在响应闭包的范围内可用。任何取决于从服务器接收到的响应或数据的执行都必须在响应闭包内完成。

> Networking in Alamofire is done _asynchronously_. Asynchronous programming may be a source of frustration to programmers unfamiliar with the concept, but there are [very good reasons](https://developer.apple.com/library/ios/qa/qa1693/_index.html) for doing it this way.
Alamofire的网络是异步完成的。异步编程可能会让不熟悉这个概念的程序员感到沮丧，但有[非常好的理由](https://developer.apple.com/library/ios/qa/qa1693/_index.html)去这样做。

Alamofire contains five different data response handlers by default, including:
默认情况下，Alamofire包含五种不同的数据响应处理程序，包括：

```swift
// Response Handler - Unserialized Response
func response(queue: DispatchQueue = .main, 
              completionHandler: @escaping (AFDataResponse<Data?>) -> Void) -> Self

// Response Serializer Handler - Serialize using the passed Serializer
func response<Serializer: DataResponseSerializerProtocol>(queue: DispatchQueue = .main,
                                                          responseSerializer: Serializer,
                                                          completionHandler: @escaping (AFDataResponse<Serializer.SerializedObject>) -> Void) -> Self

// Response Data Handler - Serialized into Data
func responseData(queue: DispatchQueue = .main,
                  dataPreprocessor: DataPreprocessor = DataResponseSerializer.defaultDataPreprocessor,
                  emptyResponseCodes: Set<Int> = DataResponseSerializer.defaultEmptyResponseCodes,
                  emptyRequestMethods: Set<HTTPMethod> = DataResponseSerializer.defaultEmptyRequestMethods,
                  completionHandler: @escaping (AFDataResponse<Data>) -> Void) -> Self

// Response String Handler - Serialized into String
func responseString(queue: DispatchQueue = .main,
                    dataPreprocessor: DataPreprocessor = StringResponseSerializer.defaultDataPreprocessor,
                    encoding: String.Encoding? = nil,
                    emptyResponseCodes: Set<Int> = StringResponseSerializer.defaultEmptyResponseCodes,
                    emptyRequestMethods: Set<HTTPMethod> = StringResponseSerializer.defaultEmptyRequestMethods,
                    completionHandler: @escaping (AFDataResponse<String>) -> Void) -> Self

// Response Decodable Handler - Serialized into Decodable Type
func responseDecodable<T: Decodable>(of type: T.Type = T.self,
                                     queue: DispatchQueue = .main,
                                     dataPreprocessor: DataPreprocessor = DecodableResponseSerializer<T>.defaultDataPreprocessor,
                                     decoder: DataDecoder = JSONDecoder(),
                                     emptyResponseCodes: Set<Int> = DecodableResponseSerializer<T>.defaultEmptyResponseCodes,
                                     emptyRequestMethods: Set<HTTPMethod> = DecodableResponseSerializer<T>.defaultEmptyRequestMethods,
                                     completionHandler: @escaping (AFDataResponse<T>) -> Void) -> Self
```

None of the response handlers perform any validation of the `HTTPURLResponse` it gets back from the server.
没有任何响应处理程序对它从服务器返回的`HTTPURLResponse`执行任何验证。

> For example, response status codes in the `400..<500` and `500..<600` ranges do NOT automatically trigger an `Error`. Alamofire uses [Response Validation](#response-validation) method chaining to achieve this.
例如，`400..<500`和`500..<600`范围内的响应状态代码不会自动触发`Error`。Alamofire使用[响应验证]（#响应验证）方法链来实现这一点。

#### Response Handler
响应处理程序

The `response` handler does NOT evaluate any of the response data. It merely forwards on all information directly from the `URLSessionDelegate`. It is the Alamofire equivalent of using `cURL` to execute a `Request`.
`response`处理程序不计算任何响应数据。它只是直接转发来自`URLSessionDelegate`的所有信息。它相当于使用`cURL`来执行`Request`。

```swift
AF.request("https://httpbin.org/get").response { response in
    debugPrint("Response: \(response)")
}
```

> We strongly encourage you to leverage the other response serializers taking advantage of `Response` and `Result` types.
我们强烈建议您利用`Response`和`Result`类型的其他响应序列化程序。

#### Response Data Handler
响应数据处理程序

The `responseData` handler uses a `DataResponseSerializer` to extract and validate the `Data` returned by the server. If no errors occur and `Data` is returned, the response `Result` will be a `.success` and the `value` will be the `Data` returned from the server.
`responseData`处理程序使用`DataResponseSerializer`来提取和验证服务器返回的`Data`。如果没有出现错误并且返回了`Data`，则响应`Result`将为`.success`，`value`将为从服务器返回的`Data`。

```swift
AF.request("https://httpbin.org/get").responseData { response in
    debugPrint("Response: \(response)")
}
```

#### Response String Handler
响应字符串处理程序

The `responseString` handler uses a `StringResponseSerializer` to convert the `Data` returned by the server into a `String` with the specified encoding. If no errors occur and the server data is successfully serialized into a `String`, the response `Result` will be a `.success` and the `value` will be of type `String`.
`responseString`处理程序使用`StringResponseSerializer`将服务器返回的`Data`转换为具有指定编码的`String`。如果没有出现错误，并且服务器数据成功地序列化为`String`，则响应`Result`将为`.success`，`value`将为类型`String`。

```swift
AF.request("https://httpbin.org/get").responseString { response in
    debugPrint("Response: \(response)")
}
```

> If no encoding is specified, Alamofire will use the text encoding specified in the `HTTPURLResponse` from the server. If the text encoding cannot be determined by the server response, it defaults to `.isoLatin1`.
如果未指定编码，Alamofire将使用服务器`HTTPURLRResponse`中指定的文本编码。如果服务器响应无法确定文本编码，则默认为`.isoLatin1`。

#### Response `Decodable` Handler
响应`Decodable`处理程序

The `responseDecodable` handler uses a `DecodableResponseSerializer` to convert the `Data` returned by the server into the passed `Decodable` type using the specified `DataDecoder` (a protocol abstraction for `Decoder`s which can decode from `Data`). If no errors occur and the server data is successfully decoded into a `Decodable` type, the response `Result` will be a `.success` and the `value` will be of the passed type.
`responseDecodable`处理程序使用`DecodeableResponseSerializer`将服务器返回的`Data`转换为使用指定的`DataDecoder`（可以从`Data`进行解码的`Decoder`s的协议抽象）传递的`decode able`类型。如果没有出现错误，并且服务器数据被成功解码为`Decodable`类型，则响应`Result`将为`.success`，`value`将为传递类型。


```swift
struct DecodableType: Decodable { let url: String }

AF.request("https://httpbin.org/get").responseDecodable(of: DecodableType.self) { response in
    debugPrint("Response: \(response)")
}
```

#### Chained Response Handlers
链式响应处理程序

Response handlers can also be chained:
响应处理程序也可以被链接：

```swift
Alamofire.request("https://httpbin.org/get")
    .responseString { response in
        print("Response String: \(response.value)")
    }
    .responseDecodable(of: DecodableType.self) { response in
        print("Response DecodableType: \(response.value)")
    }
```

> It is important to note that using multiple response handlers on the same `Request` requires the server data to be serialized multiple times, once for each response handler. Using multiple response handlers on the same `Request` should generally be avoided as best practice, especially in production environments. They should only be used for debugging or in circumstances where there is no better option.
需要注意的是，在同一个`Request`上使用多个响应处理程序需要多次序列化服务器数据，每个响应处理程序一次。通常应避免在同一`请求`上使用多个响应处理程序作为最佳实践，尤其是在生产环境中。它们只能用于调试或在没有更好选择的情况下使用。


#### Response Handler Queue
响应处理程序队列

Closures passed to response handlers are executed on the `.main` queue by default, but a specific `DispatchQueue` can be passed on which to execute the closure. Actual serialization work (conversion of `Data` to some other type) is always executed in the background on either the `rootQueue` or the `serializationQueue`, if one was provided, of the `Session` issuing the request.
默认情况下，传递给响应处理程序的闭包在`.main`队列上执行，但可以传递特定的`DispatchQueue`来执行闭包。实际的序列化工作（将`Data`转换为其他类型）总是在发出请求的`Session`的`rootQueue`或`serializationQueue`（如果提供）的后台执行。

```swift
let utilityQueue = DispatchQueue.global(qos: .utility)

AF.request("https://httpbin.org/get").responseDecodable(of: DecodableType.self, queue: utilityQueue) { response in
    print("This closure is executed on utilityQueue.")
    debugPrint(response)
}
```

### Response Caching
响应缓存

Response caching is handled on the system framework level by [`URLCache`](https://developer.apple.com/reference/foundation/urlcache). It provides a composite in-memory and on-disk cache and lets you manipulate the sizes of both the in-memory and on-disk portions.
响应缓存由[`URLCache`]在系统框架级别上处理(https://developer.apple.com/reference/foundation/urlcache). 它提供了内存和磁盘缓存的组合，并允许您操作内存和磁盘部分的大小。

> By default, Alamofire leverages the `URLCache.shared` instance. In order to customize the `URLCache` instance used, see the [Session Configuration](AdvancedUsage.md#session-manager) section.
默认情况下，Alamofire使用`URLCache.shared`实例。要自定义使用的`URLCache`实例，请参阅[Session Configuration]（AdvancedUsage.md#会话管理器）部分。

### Authentication
身份验证

Authentication is handled on the system framework level by [`URLCredential`](https://developer.apple.com/reference/foundation/nsurlcredential) and [`URLAuthenticationChallenge`](https://developer.apple.com/reference/foundation/urlauthenticationchallenge).
身份验证由[`URLCredential`]在系统框架级别处理(https://developer.apple.com/reference/foundation/nsurlcredential)和[`URLAuthenticationChallenge`](https://developer.apple.com/reference/foundation/urlauthenticationchallenge).


> These authentication APIs are for servers which prompt for authorization, not general use with APIs which require an `Authenticate`  or equivalent header.
这些身份验证API用于提示授权的服务器，而不是与需要`Authenticate`或等效请求头的API一起通用。

**Supported Authentication Schemes**
支持的身份验证方案

- [HTTP Basic](https://en.wikipedia.org/wiki/Basic_access_authentication)
- [HTTP Digest](https://en.wikipedia.org/wiki/Digest_access_authentication)
- [Kerberos](https://en.wikipedia.org/wiki/Kerberos_%28protocol%29)
- [NTLM](https://en.wikipedia.org/wiki/NT_LAN_Manager)

#### HTTP Basic Authentication
HTTP基本身份验证

The `authenticate` method on a `Request` will automatically provide a `URLCredential` when challenged with a `URLAuthenticationChallenge` when appropriate:
`Request`上的`authenticate`方法将在适当时用`URLAuthenticationChallenge`进行质询时自动提供`URLCredential：

```swift
let user = "user"
let password = "password"

AF.request("https://httpbin.org/basic-auth/\(user)/\(password)")
    .authenticate(username: user, password: password)
    .responseDecodable(of: DecodableType.self) { response in
        debugPrint(response)
    }
```

#### Authentication with `URLCredential`
使用`URLCredential进行身份验证`

```swift
let user = "user"
let password = "password"

let credential = URLCredential(user: user, password: password, persistence: .forSession)

AF.request("https://httpbin.org/basic-auth/\(user)/\(password)")
    .authenticate(with: credential)
    .responseDecodable(of: DecodableType.self) { response in
        debugPrint(response)
    }
```

> It is important to note that when using a `URLCredential` for authentication, the underlying `URLSession` will actually end up making two requests if a challenge is issued by the server. The first request will not include the credential which "may" trigger a challenge from the server. The challenge is then received by Alamofire, the credential is appended and the request is retried by the underlying `URLSession`.
需要注意的是，当使用`URLCredential`进行身份验证时，如果服务器发出质询，底层的`URLSession`实际上会发出两个请求。第一个请求将不包括"可能"触发来自服务器的质询的凭据。然后，Alamofire接收质询，附加凭证，并通过底层的`URLSession`重试请求。

#### Manual Authentication
手动身份验证

If you are communicating with an API that always requires an `Authenticate` or similar header without prompting, it can be added manually:
如果您使用的API总是需要`Authenticate`或类似的请求头而没有提示，则可以手动添加：


```swift
let user = "user"
let password = "password"

let headers: HTTPHeaders = [.authorization(username: user, password: password)]

AF.request("https://httpbin.org/basic-auth/user/password", headers: headers)
    .responseDecodable(of: DecodableType.self) { response in
        debugPrint(response)
    }
```

However, headers that must be part of all requests are often better handled as part of a custom [`URLSessionConfiguration`](AdvancedUsage.md#session-manager), or by using a [`RequestAdapter`](AdvancedUsage.md#request-adapter).
然而，必须是所有请求的一部分的头通常作为自定义[`URLSessionConfiguration`]（AdvancedUsage.md#会话管理器）的一部分，或通过使用[`RequestAdapter`]（Advanced Usage.md#request adapter）来更好地处理。

### Downloading Data to a File
将数据下载到文件

In addition to fetching data into memory, Alamofire also provides the `Session.download`, `DownloadRequest`, and `DownloadResponse<Success, Failure: Error>` APIs to facilitate downloading to disk. While downloading into memory works great for small payloads like most JSON API responses, fetching larger assets like images and videos should be downloaded to disk to avoid memory issues with your application.
除了将数据提取到内存中，Alamofire还提供`Session.download`、`DownloadRequest`和`DownloadResponse<Success, Failure: Error>`API，以方便下载到磁盘。虽然下载到内存中对于像大多数JSON API响应这样的小有效负载非常有效，但获取图像和视频等较大资产应该下载到磁盘，以避免应用程序出现内存问题。

```swift
AF.download("https://httpbin.org/image/png").responseURL { response in
    // Read file from provided file URL.
}
```

In addition to having the same response handlers that `DataRequest` does, `DownloadRequest` also includes `responseURL`. Unlike the other response handlers, this handler just returns the `URL` containing the location of the downloaded data and does not read the `Data` from disk.
除了具有与`DataRequest`相同的响应处理程序外，`DownloadRequest`还包括`responseURL`。与其他响应处理程序不同，此处理程序只返回包含下载数据位置的`URL`，而不从磁盘读取`数据`。

Other response handlers, like `responseDecodable`, involve reading the response `Data` from disk. This may involve reading large amounts of data into memory, so it's important to keep that in mind when using those handlers for downloads.
其他响应处理程序，如`responseDecodable`，涉及从磁盘读取响应`Data`。这可能涉及到将大量数据读取到内存中，因此在使用这些处理程序进行下载时，务必牢记这一点。

#### Download File Destination
下载文件目标

All downloaded data is initially stored in the system temporary directory. It will eventually be deleted by the system at some point in the future, so if it's something that needs to live longer, it's important to move the file somewhere else.
所有下载的数据最初都存储在系统临时目录中。它最终会在未来的某个时候被系统删除，所以如果它需要更长的寿命，那么将文件移到其他地方是很重要的。

You can provide a `Destination` closure to move the file from the temporary directory to a final destination. Before the temporary file is actually moved to the `destinationURL`, the `Options` specified in the closure will be executed. The two currently supported `Options` are:
您可以提供一个`Destination`闭包，将文件从临时目录移动到最终目的地。在临时文件实际移动到`destinationURL`之前，将执行闭包中指定的`Options`。目前支持的两个`Options`是：

- `.createIntermediateDirectories` - Creates intermediate directories for the destination URL if specified.
`.createIntermediateDirectories` - 如果指定，则为目标URL创建中间目录。
- `.removePreviousFile` - Removes a previous file from the destination URL if specified.
`.removePreviousFile` - 如果指定，则从目标URL中删除先前的文件。

```swift
let destination: DownloadRequest.Destination = { _, _ in
    let documentsURL = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    let fileURL = documentsURL.appendingPathComponent("image.png")

    return (fileURL, [.removePreviousFile, .createIntermediateDirectories])
}

AF.download("https://httpbin.org/image/png", to: destination).response { response in
    debugPrint(response)

    if response.error == nil, let imagePath = response.fileURL?.path {
        let image = UIImage(contentsOfFile: imagePath)
    }
}
```

You can also use the suggested download destination API:
您也可以使用建议的下载目的地API：

```swift
let destination = DownloadRequest.suggestedDownloadDestination(for: .documentDirectory)

AF.download("https://httpbin.org/image/png", to: destination)
```

#### Download Progress
下载进度

Many times it can be helpful to report download progress to the user. Any `DownloadRequest` can report download progress using the `downloadProgress` API.
很多时候，向用户报告下载进度是很有帮助的。任何`DownloadRequest`都可以使用`downloadProgress`API报告下载进度。


```swift
AF.download("https://httpbin.org/image/png")
    .downloadProgress { progress in
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseData { response in
        if let data = response.value {
            let image = UIImage(data: data)
        }
    }
```

> The progress reporting APIs for `URLSession`, and therefore Alamofire, only work if the server properly returns a `Content-Length` header that can be used to calculate the progress. Without that header, progress will stay at `0.0` until the download completes, at which point the progress will jump to `1.0`.
只有当服务器正确返回可用于计算进度的`Content-Length`请求头时，`URLSession`的进度报告API以及Alamofire才能工作。如果没有该请求头，进度将保持在`0.0`，直到下载完成，此时进度将跳到`1.0`。


The `downloadProgress` API can also take a `queue` parameter which defines which `DispatchQueue` the download progress closure should be called on.
`downloadProgress`API还可以采用`queue`参数，该参数定义应在哪个`DispatchQueue`上调用下载进度闭包。

```swift
let progressQueue = DispatchQueue(label: "com.alamofire.progressQueue", qos: .utility)

AF.download("https://httpbin.org/image/png")
    .downloadProgress(queue: progressQueue) { progress in
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseData { response in
        if let data = response.value {
            let image = UIImage(data: data)
        }
    }
```

#### Canceling and Resuming a Download
取消并继续下载

In addition to the `cancel()` API that all `Request` classes have, `DownloadRequest`s can also produce resume data, which can be used to later resume a download. There are two forms of this API: `cancel(producingResumeData: Bool)`, which allows control over whether resume data is produced, but only makes it available on the `DownloadResponse`; and `cancel(byProducingResumeData: (_ resumeData: Data?) -> Void)`, which performs the same actions but makes the resume data available in the completion handler.
除了所有`Request`类都具有的`cancel()`API外，`DownloadRequest`还可以生成恢复数据，这些数据可以用于以后恢复下载。此API有两种形式：`cancel(producingResumeData: Bool)`，它允许控制是否生成恢复数据，但仅在`DownloadResponse`上可用;`cancel（byProducingResumeData:（_ resumeData:Data？）->Void）`，会执行相同的操作，但在完成处理程序中提供恢复数据。


If a `DownloadRequest` is canceled or interrupted, the underlying `URLSessionDownloadTask` *may* generate resume data. If this happens, the resume data can be re-used to restart the `DownloadRequest` where it left off. 
如果`DownloadRequest`被取消或中断，那么基础的`URLSessionDownloadTask`*可能会*生成恢复数据。如果发生这种情况，可以重新使用恢复数据来重新启动它停止的`DownloadRequest`。

> **IMPORTANT:** On some versions of all Apple platforms (iOS 10 - 10.2, macOS 10.12 - 10.12.2, tvOS 10 - 10.1, watchOS 3 - 3.1.1), `resumeData` is broken on background `URLSessionConfiguration`s. There's an underlying bug in the `resumeData` generation logic where the data is written incorrectly and will always fail to resume the download. For more information about the bug and possible workarounds, please see this [Stack Overflow post](https://stackoverflow.com/a/39347461/1342462).
**重要提示：**在所有苹果平台的某些版本（iOS 10-10.2、macOS 10.12-10.12.2、tvOS 10-10.1、watchOS 3-3.1.1）上，`resumeData`在后台`URLSessionConfiguration`上断开。`resumeData`生成逻辑中存在一个潜在的错误，即数据写入错误，并且总是无法恢复下载。有关该错误和可能的解决方法的更多信息，请参阅此[Stack Overflow post](https://stackoverflow.com/a/39347461/1342462).

```swift
var resumeData: Data!

let download = AF.download("https://httpbin.org/image/png").responseData { response in
    if let data = response.value {
        let image = UIImage(data: data)
    }
}

// download.cancel(producingResumeData: true) // Makes resumeData available in response only.
download.cancel { data in
    resumeData = data
}

AF.download(resumingWith: resumeData).responseData { response in
    if let data = response.value {
        let image = UIImage(data: data)
    }
}
```

### Uploading Data to a Server
将数据上传到服务器

When sending relatively small amounts of data to a server using JSON or URL encoded parameters, the `request()` APIs are usually sufficient. If you need to send much larger amounts of data from `Data` in memory, a file `URL`, or an `InputStream`, then the `upload()` APIs are what you want to use.
当使用JSON或URL编码的参数向服务器发送相对少量的数据时，`request()`API通常就足够了。如果您需要从内存中的`data`、文件`URL`或`InputStream`发送大量数据，那么您需要使用`upload（）API。

#### Uploading Data
上传数据

```swift
let data = Data("data".utf8)

AF.upload(data, to: "https://httpbin.org/post").responseDecodable(of: DecodableType.self) { response in
    debugPrint(response)
}
```

#### Uploading a File
上传文件

```swift
let fileURL = Bundle.main.url(forResource: "video", withExtension: "mov")

AF.upload(fileURL, to: "https://httpbin.org/post").responseDecodable(of: DecodableType.self) { response in
    debugPrint(response)
}
```

#### Uploading Multipart Form Data
上传多部分表单数据

```swift
AF.upload(multipartFormData: { multipartFormData in
    multipartFormData.append(Data("one".utf8), withName: "one")
    multipartFormData.append(Data("two".utf8), withName: "two")
}, to: "https://httpbin.org/post")
    .responseDecodable(of: DecodableType.self) { response in
        debugPrint(response)
    }
```

#### Upload Progress
上传进度

While your user is waiting for their upload to complete, sometimes it can be handy to show the progress of the upload to the user. Any `UploadRequest` can report both upload progress of the upload and download progress of the response data download using the `uploadProgress` and `downloadProgress` APIs.
当用户正在等待上传完成时，有时向用户显示上传进度会很方便。任何`UploadRequest`都可以使用`uploadProgress`和`downloadProgress`API报告上传的上传进度和响应数据下载的下载进度。

```swift
let fileURL = Bundle.main.url(forResource: "video", withExtension: "mov")

AF.upload(fileURL, to: "https://httpbin.org/post")
    .uploadProgress { progress in
        print("Upload Progress: \(progress.fractionCompleted)")
    }
    .downloadProgress { progress in
        print("Download Progress: \(progress.fractionCompleted)")
    }
    .responseDecodable(of: DecodableType.self) { response in
        debugPrint(response)
    }
```

### Streaming Data from a Server
从服务器流式传输数据

Large downloads or long lasting server connections which receive data over time may be better served by streaming rather than accumulating `Data` as it arrives. Alamofire offers the `DataStreamRequest` type and associated APIs to handle this usage. Although it offers much of the same API as other `Request`s, there are several key differences. Most notably, `DataStreamRequest` never accumulates `Data` in memory or saves it to disk. Instead, added `responseStream` closures are repeatedly called as `Data` arrives. The same closures are called again when the connection has completed or received an error.
随着时间的推移接收数据的大型下载或长期服务器连接可以通过流式传输得到更好的服务，而不是在数据到达时积累`Data`。Alamofire提供了`DataStreamRequest`类型和相关的API来处理这种用法。尽管它提供了与其他`Request`相同的API，但有几个关键区别。最值得注意的是，`DataStreamRequest`从不在内存中累积`Data`或将其保存到磁盘。相反，添加的`responseStream`闭包在`Data到达时被重复调用。当连接完成或收到错误时，会再次调用相同的闭包。

Every `Handler` closure captures a `Stream` value, which contains both the `Event` being processed as well as a `CancellationToken`, which can be used to cancel the request.
每个`Handler`闭包都捕获一个`Stream`值，该值既包含正在处理的`Event`，也包含一个可用于取消请求的`CancellationToken`。

```swift
public struct Stream<Success, Failure: Error> {
    /// Latest `Event` from the stream.
    public let event: Event<Success, Failure>
    /// Token used to cancel the stream.
    public let token: CancellationToken
    /// Cancel the ongoing stream by canceling the underlying `DataStreamRequest`.
    public func cancel() {
        token.cancel()
    }
}
```

An `Event` is an `enum` representing two possible stream states.
`Event`是表示两种可能的流状态的`enum`。

```swift
public enum Event<Success, Failure: Error> {
    /// Output produced every time the instance receives additional `Data`. The associated value contains the
    /// `Result` of processing the incoming `Data`.
    case stream(Result<Success, Failure>)
    /// Output produced when the instance has completed, whether due to stream end, cancellation, or an error.
    /// Associated `Completion` value contains the final state.
    case complete(Completion)
}
```

When complete, the `Completion` value will contain the state of the `DataStreamRequest` when the stream ended.
完成后，`Completion`值将包含流结束时`DataStreamRequest`的状态。

```swift
public struct Completion {
    /// Last `URLRequest` issued by the instance.
    public let request: URLRequest?
    /// Last `HTTPURLResponse` received by the instance.
    public let response: HTTPURLResponse?
    /// Last `URLSessionTaskMetrics` produced for the instance.
    public let metrics: URLSessionTaskMetrics?
    /// `AFError` produced for the instance, if any.
    public let error: AFError?
}
```

#### Streaming `Data`
流式处理`Data`

Streaming `Data` from a server can be accomplished like other Alamofire requests, but with a `Handler` closure added.
从服务器流式传输`Data`可以像其他Alamofire请求一样完成，但需要添加`Handler`闭包。

```swift
func responseStream(on queue: DispatchQueue = .main, stream: @escaping Handler<Data, Never>) -> Self
```

The provided `queue` is where the `Handler` closure will be called.
所提供的`queue`是调用`Handler`闭包的地方。

```swift
AF.streamRequest(...).responseStream { stream in
    switch stream.event {
    case let .stream(result):
        switch result {
        case let .success(data):
            print(data)
        }
    case let .complete(completion):
        print(completion)
    }
}
```

> Handling the `.failure` case of the `Result` in the example above is unnecessary, as receiving `Data` can never fail.
处理上例中`Result`的`.failure`情况是不必要的，因为接收`Data`永远不会失败。

#### Streaming `String`s
流式处理`String`

Like `Data` streaming, `String`s can be streamed by adding a `Handler`.
与`Data`流一样，`String`可以通过添加`Handler`进行流式传输。

```swift
func responseStreamString(on queue: DispatchQueue = .main,
                          stream: @escaping StreamHandler<String, Never>) -> Self
```

`String` values are decoded as `UTF8` and the decoding cannot fail.
`String`值被解码为`UTF8`，并且解码不会失败。

```swift
AF.streamRequest(...).responseStreamString { stream in
    switch stream.event {
    case let .stream(result):
        switch result {
        case let .success(string):
            print(string)
        }
    case let .complete(completion):
        print(completion)
    }
}
```

#### Streaming `Decodable` Values
流式处理`Decodable`值

Incoming stream `Data` values can be turned into any `Decodable` value using `responseStreamDecodable`.
传入流`Data`值可以使用`responseStreamDecodable`转换为任何`Decodable`值。

```swift
func responseStreamDecodable<T: Decodable>(of type: T.Type = T.self,
                                           on queue: DispatchQueue = .main,
                                           using decoder: DataDecoder = JSONDecoder(),
                                           preprocessor: DataPreprocessor = PassthroughPreprocessor(),
                                           stream: @escaping Handler<T, AFError>) -> Self
```

Decoding failures do not end the stream, but instead produce an `AFError` in the `Result` of the `Output`.
解码失败不会结束流，而是在`Output`的`Result`中产生`AFError`。


```swift
AF.streamRequest(...).responseStreamDecodable(of: SomeType.self) { stream in
    switch stream.event {
    case let .stream(result):
        switch result {
        case let .success(value):
            print(value)
        case let .failure(error):
            print(error)
        }
    case let .complete(completion):
        print(completion)
    }
}
```

#### Producing an `InputStream`
生成`InputStream`

In addition to handling incoming `Data` using `StreamHandler` closures, `DataStreamRequest` can produce an `InputStream` value which can be used to read bytes as they arrive.
除了使用`StreamHandler`闭包处理传入的`Data`外，`DataStreamRequest`还可以生成一个`InputStream`值，该值可用于在字节到达时读取字节。

```swift
func asInputStream(bufferSize: Int = 1024) -> InputStream
```

`InputStream`s produced in this manner must have `open()` called before reading can start, or be passed to an API that opens the stream automatically. Once returned from this method, it's the caller's responsibility to keep the `InputStream` value alive and to call `close()` after reading is complete.
以这种方式生成的`InputStream`必须先调用`open()`，然后才能开始读取，或者传递给自动打开流的API。从该方法返回后，调用者有责任保持`InputStream`值的有效性，并在读取完成后调用`close()`。

```swift
let inputStream = AF.streamRequest(...)
    .responseStream { output in
        ...
    }
    .asInputStream()
```

#### Cancellation
取消

`DataStreamRequest`s can be cancelled in four ways. First, like all other Alamofire `Request`s, `DataStreamRequest` can have `cancel()` called, canceling the underlying task and completing the stream.
可以通过四种方式取消`DataStreamRequest`。首先，像所有其他Alamofire`Request`一样，`DataStreamRequest`可以调用`cancel()`，从而取消底层任务并完成流。

```swift
let request = AF.streamRequest(...).responseStream(...)
...
request.cancel()
```

Second, `DataStreamRequest`s can be cancelled automatically when their `DataStreamSerializer` encounters and error. This behavior is disabled by default and can be enabled by passing the `automaticallyCancelOnStreamError` parameter when creating the request.
其次，当`DataStreamSerializer`遇到错误时，可以自动取消`DataStreamRequest`。此行为在默认情况下被禁用，并且可以通过在创建请求时传递`automaticallyCancelOnStreamError参数来启用。

```swift
AF.streamRequest(..., automaticallyCancelOnStreamError: true).responseStream(...)
```

Third, `DataStreamRequest`s will be cancelled if an error is thrown out of the `Handler` closure. This error is then stored on the request and is available in the `Completion` value.
第三，如果从`Handler`闭包中抛出错误，`DataStreamRequest`将被取消。该错误随后存储在请求中，并在`Completion`值中可用。

```swift
AF.streamRequest(...).responseStream { stream in
    // Process stream.
    throw SomeError() // Cancels request.
}
```

Finally, `DataStreamRequest`s can be cancelled by using the `Stream` value's `cancel()` method. 
最后，可以使用`Stream`值的`cancel()`方法取消`DataStreamRequest`。

```swift
AF.streamRequest(...).responseStream { stream in 
    // Decide to cancel request.
    stream.cancel()
}
```

### Statistical Metrics
统计指标

#### `URLSessionTaskMetrics`

Alamofire gathers `URLSessionTaskMetrics` for every `Request`. `URLSessionTaskMetrics` encapsulate some fantastic statistical information about the underlying network connection and request and response timing.
Alamofire为每个`Request`收集`URLSessionTaskMetrics`。`URLSessionTaskMetrics`封装了一些关于底层网络连接以及请求和响应时间的奇妙统计信息。


```swift
AF.request("https://httpbin.org/get").responseDecodable(of: DecodableType.self) { response in
    print(response.metrics)
}
```

> Due to `FB7624529`, collection of `URLSessionTaskMetrics` on watchOS is currently disabled.
由于`FB7624529`，watchOS上的`URLSessionTaskMetrics`集合当前被禁用。

### cURL Command Output
cURL命令输出

Debugging platform issues can be frustrating. Thankfully, Alamofire's `Request` type can produce the equivalent cURL command for easy debugging. Due to the asynchronous nature of Alamofire's `Request` creation, this API has both synchronous and asynchronous versions. To get the cURL command as soon as possible, you can chain the `cURLDescription` onto a request:
调试平台问题可能会令人沮丧。值得庆幸的是，Alamofire的`Request`类型可以生成等效的cURL命令，以便于调试。由于Alamofire的`Request`创建具有异步性质，因此此API既有同步版本，也有异步版本。要尽快获得cURL命令，可以将`cURLDescription`链接到请求上：

```swift
AF.request("https://httpbin.org/get")
    .cURLDescription { description in
        print(description)
    }
    .responseDecodable(of: DecodableType.self) { response in
        debugPrint(response.metrics)
    }
```

This should produce:
这应该产生：

```bash
$ curl -v \
-X GET \
-H "Accept-Language: en;q=1.0" \
-H "Accept-Encoding: br;q=1.0, gzip;q=0.9, deflate;q=0.8" \
-H "User-Agent: Demo/1.0 (com.demo.Demo; build:1; iOS 15.0.0) Alamofire/1.0" \
"https://httpbin.org/get"
```
