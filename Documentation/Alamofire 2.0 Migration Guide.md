# Alamofire 2.0 Migration Guide
Alamofire 2.0迁移指南

Alamofire 2.0 is the latest major release of Alamofire, an HTTP networking library for iOS, Mac OS X and watchOS written in Swift. As a major release, following Semantic Versioning conventions, 2.0 introduces several API-breaking changes that one should be aware of.
Alamofire2.0是Alamofire的最新主要版本，这是一个用Swift编写的适用于iOS、Mac OS X和watchOS的HTTP网络库。作为一个主要版本，遵循语义版本控制约定，2.0引入了一些应该注意的API破坏性更改。


This guide is provided in order to ease the transition of existing applications using Alamofire 1.x to the latest APIs, as well as explain the design and structure of new and changed functionality.
提供本指南是为了简化使用Alamofire 1.x的现有应用程序向最新API的过渡，并解释新功能和更改功能的设计和结构。

## New Requirements
新要求

Alamofire 2.0 officially supports iOS 8+, Mac OS X 10.9+, Xcode 7 and Swift 2.0. If you'd like to use Alamofire in a project targeting iOS 7 and Swift 1.x, use the latest tagged 1.x release.
Alamofire 2.0正式支持iOS 8+、Mac OS X 10.9+、Xcode 7和Swift 2.0。如果你想在针对iOS 7和Swift 1.x的项目中使用Alamofire，请使用最新的1.x版本。

---

## Breaking API Changes

### Swift 2.0

The biggest change between Alamofire 1.x and Alamofire 2.0 is Swift 2.0. Swift 2 brought many new features to take advantage of such as error handling, protocol extensions and availability checking. Other new features such as `guard` and `defer` do not affect the public APIs, but allowed us to create much cleaner implementations of the same logic. All of the source files, test logic and example code has been updated to reflect the latest Swift 2.0 paradigms.
Alamofire1.x和Alamofire2.0之间最大的变化是Swift 2.0。Swift 2带来了许多新功能，如错误处理、协议扩展和可用性检查。其他新功能，如`guard`和`defer`，不会影响公共API，但允许我们创建相同逻辑的更干净的实现。所有的源文件、测试逻辑和示例代码都已更新，以反映最新的Swift 2.0范例。

> It is not possible to use Alamofire 2.0 without Swift 2.0.
如果没有Swift 2.0，就不可能使用Alamofire 2.0。

### Response Serializers
响应序列化程序

The most significant logic change made to Alamofire 2.0 is its new response serialization system leveraging `Result` types. Previously in Alamofire 1.x, each response serializer used the same completion handler signature:
对Alamofire 2.0进行的最重要的逻辑更改是其利用`Result`类型的新响应序列化系统。以前在Alamofire1.x中，每个响应序列化程序都使用相同的完成处理程序签名：

```swift
public func response(completionHandler: (NSURLRequest, NSHTTPURLResponse?, AnyObject?, NSError?) -> Void) -> Self {
    return response(serializer: Request.responseDataSerializer(), completionHandler: completionHandler)
}
```

Alamofire 2.0 has redesigned the entire response serialization process to make it much easier to access the original server data without serialization, or serialize the response into a non-optional `Result` type defining whether the `Request` was successful.
Alamofire 2.0重新设计了整个响应序列化过程，使访问原始服务器数据变得更容易，而无需序列化，或者将响应序列化为非可选的`Result`类型，定义`Request`是否成功。

#### No Response Serialization
无响应序列化

The first `response` serializer is non-generic and does not process the server data in any way. It merely forwards on the accumulated information from the `NSURLSessionDelegate` callbacks.
第一个`response`序列化程序是非通用的，不会以任何方式处理服务器数据。它只是转发来自`NSURLSessionDelegate`回调的累积信息。

```swift
public func response(
	queue queue: dispatch_queue_t? = nil,
	completionHandler: (NSURLRequest?, NSHTTPURLResponse?, NSData?, ErrorType?) -> Void)
	-> Self
{
	delegate.queue.addOperationWithBlock {
		dispatch_async(queue ?? dispatch_get_main_queue()) {
			completionHandler(self.request, self.response, self.delegate.data, self.delegate.error)
		}
	}

	return self
}
```

Another important note of this change is the return type of `data` is now an `NSData` type. You no longer need to cast the `data` parameter from an `AnyObject?` to an `NSData?`.
这一变化的另一个重要注意事项是`data`的返回类型现在是`NSData`类型。您不再需要从`AnyObject？`强制转换`data`参数转换为`NSData?`。

#### Generic Response Serializers
通用响应序列化程序

The second, more powerful response serializer leverages generics along with a `Result` type to eliminate the case of the dreaded double optional.
第二个更强大的响应序列化程序利用泛型和`Result`类型来消除可怕的双可选的情况。

```swift
public func response<T: ResponseSerializer, V where T.SerializedObject == V>(
    queue queue: dispatch_queue_t? = nil,
    responseSerializer: T,
    completionHandler: (NSURLRequest?, NSHTTPURLResponse?, Result<V>) -> Void)
    -> Self
{
    delegate.queue.addOperationWithBlock {
        let result: Result<T.SerializedObject> = {
            if let error = self.delegate.error {
                return .Failure(self.delegate.data, error)
            } else {
                return responseSerializer.serializeResponse(self.request, self.response, self.delegate.data)
            }
        }()

        dispatch_async(queue ?? dispatch_get_main_queue()) {
            completionHandler(self.request, self.response, result)
        }
    }

    return self
}
```

##### Response Data
响应数据

```swift
Alamofire.request(.GET, "http://httpbin.org/get")
         .responseData { _, _, result in
             print("Success: \(result.isSuccess)")
             print("Response: \(result)")
         }
```

##### Response String
响应字符串

```swift
Alamofire.request(.GET, "http://httpbin.org/get")
         .responseString { _, _, result in
             print("Success: \(result.isSuccess)")
             print("Response String: \(result.value)")
         }
```

##### Response JSON
响应JSON

```swift
Alamofire.request(.GET, "http://httpbin.org/get")
         .responseJSON { _, _, result in
             print(result)
             debugPrint(result)
         }
```

#### Result Types
结果类型

The `Result` enumeration was added to handle the case of the double optional return type. Previously, the return value and error were both optionals. Checking if one was `nil` did not ensure the other was also not `nil`. This case has been blogged about many times and can be solved by a `Result` type. Alamofire 2.0 brings a `Result` type to the response serializers to make it much easier to handle success and failure cases.
添加`Result`枚举是为了处理双可选返回类型的情况。以前，返回值和错误都是可选的。检查其中一个是否为`nil`并不能确保另一个也不是`nil`。这个案例已经在博客上被写了很多次，可以通过`Result`类型来解决。Alamofire 2.0为响应序列化程序引入了`Result`类型，使其更容易处理成功和失败的情况。

```swift
public enum Result<Value> {
    case Success(Value)
    case Failure(NSData?, ErrorType)
}
```

There are also many other convenience computed properties to make accessing the data inside easy. The `Result` type also conforms to the `CustomStringConvertible` and `CustomDebugStringConvertible` protocols to make it easier to debug.
还有许多其他方便的计算属性，可以轻松访问内部数据。`Result`类型还符合`CustomStringConvertible`和`CustomDebugStringConvertibility`协议，以便于调试。

#### Error Types
错误类型

While Alamofire still only generates `NSError` objects, all `Result` types have been converted to store `ErrorType` objects to allow custom response serializer implementations to use any `ErrorType` they wish. This also includes the `ValidationResult` and `MultipartFormDataEncodingResult` types as well.
虽然Alamofire仍然只生成`NSError`对象，但所有`Result`类型都已转换为存储`ErrorType`对象，以允许自定义响应序列化程序实现使用他们希望使用的任何`ErrorType`。这也包括`ValidationResult`和`MultipartFormDataEncodingResult`类型。

### URLRequestConvertible
URL可转换请求

In order to make it easier to deal with non-common scenarios, the `URLRequestConvertible` protocol now returns an `NSMutableURLRequest`. Alamofire 2.0 makes it much easier to customize the URL request after is has been encoded. This should only affect a small amount of users.
为了更容易处理非常见情况，`URLRequestConvertible`协议现在返回`NSMutableURLRequest。Alamofire 2.0使得在编码后自定义URL请求变得更加容易。这应该只影响一小部分用户。

```swift
public protocol URLRequestConvertible {
    var URLRequest: NSMutableURLRequest { get }
}
```

### Multipart Form Data
多部分表单数据

Encoding `MultipartFormData` previous returned an `EncodingResult` to encapsulate any possible errors that occurred during encoding. Alamofire 2.0 uses the new Swift 2.0 error handling instead making it easier to use. This change is mostly encapsulated internally and should only affect a very small subset of users.
编码`MultipartFormData`之前返回了一个`EncodingResult`，以封装编码过程中可能发生的任何错误。Alamofire 2.0使用了新的Swift 2.0错误处理，使其更易于使用。这一变化大多是内部封装的，应该只影响非常小的一部分用户。

---

## Updated ACLs and New Features

### Parameter Encoding
参数编码

#### ACL Updates
ACL更新

The `ParameterEncoding` enumeration implementation was previously hidden behind `internal` and `private` ACLs. Alamofire 2.0 opens up the `queryComponents` and `escape` methods to make it much easier to implement `.Custom` cases.
`ParameterEncoding`枚举实现以前隐藏在`internal`和`private`ACL后面。Alamofire 2.0打开了`queryComponents`和`escape`方法，使其更容易实现`.Custom`案例。


#### Encoding in the URL
URL中的编码

In the previous versions of Alamofire, `.URL` encoding would automatically append the query string to either the URL or HTTP body depending on which HTTP method was set in the `NSURLRequest`. While this satisfies the majority of common use cases, it made it quite difficult to append query string parameter to a URL for HTTP methods such as `PUT` and `POST`. In Alamofire 2.0, we've added a second URL encoding case, `.URLEncodedInURL`, that always appends the query string to the URL regardless of HTTP method.
在Alamofire的早期版本中，`.URL`编码会根据`NSURLRequest`中设置的HTTP方法，自动将查询字符串附加到URL或HTTP正文中。虽然这满足了大多数常见用例，但对于诸如`PUT`和`POST`之类的HTTP方法，将查询字符串参数附加到URL中是非常困难的。在Alamofire 2.0中，我们添加了第二种URL编码情况，`.URLEncodedInURL`，它总是将查询字符串附加到URL，而不考虑HTTP方法。

### Server Trust Policies
服务器信任策略

In Alamofire 1.x, the `ServerTrustPolicyManager` methods were internal making it impossible to implement any custom domain matching behavior. Alamofire 2.0 opens up the internals with a `public` ACL allowing more flexible server trust policy matching behavior (i.e. wildcarded domains) through subclassing.
在Alamofire 1.x中，`ServerTrustPolicyManager`方法是内部的，因此无法实现任何自定义的域匹配行为。Alamofire 2.0通过`public`ACL打开了内部，允许通过子类化实现更灵活的服务器信任策略匹配行为（即通配符域）。

```swift
class CustomServerTrustPolicyManager: ServerTrustPolicyManager {
    override func serverTrustPolicyForHost(host: String) -> ServerTrustPolicy? {
        var policy: ServerTrustPolicy?

        // Implement your custom domain matching behavior...

        return policy
    }
}
```

### Download Requests
下载请求

The global and `Manager` download APIs now support `parameters` and `encoding` parameters to better support dynamic payloads used in background sessions. Constructing a `download` request is now the same as constructing a `data` request with the addition of a `destination` parameter.
全局和`Manager`下载API现在支持`parameters`和`encoding`参数，以更好地支持后台会话中使用的动态有效载荷。构造`download`请求现在与构造添加了`destination`参数的`data`请求相同。

```swift
public func download(
    method: Method,
    _ URLString: URLStringConvertible,
    parameters: [String: AnyObject]? = nil,
    encoding: ParameterEncoding = .URL,
    headers: [String: String]? = nil,
    destination: Request.DownloadFileDestination)
    -> Request
{
    return Manager.sharedInstance.download(
        method,
        URLString,
        parameters: parameters,
        encoding: encoding,
        headers: headers,
        destination: destination
    )
}
```

### Stream Tasks
流式处理任务

Alamofire 2.0 adds support for creating `NSURLSessionStreamTask` tasks for iOS 9 and OS X 10.11. It also extends the `SessionDelegate` to support all the new `NSURLSessionStreamDelegate` APIs.
Alamofire 2.0增加了对为iOS 9和OS X 10.11创建`NSURLSessionStreamTask`任务的支持。它还扩展了`SessionDelegate`以支持所有新的`NSURLSessionStreamDelegate`API。
