---
layout: post
title: "Schroedinger's completion handler"
date: 2018-03-26
---

As Ole Begemann points out in a new [blog post](https://oleb.net/blog/2018/03/making-illegal-states-unrepresentable/), the completion handler of Apple's URLSession has three parameters, all of which are optional:

```swift
class URLSession {
    func dataTask(with url: URL,
        completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void)
        -> URLSessionDataTask
}
```

This presents us with a problem, as it is not inherently clear how to interpret certain cases. What does it mean if you receive data, but also an error? What if you don't receiver either?

## Error handling in Objective-C

URLSession isn't the only class that does this, it's all over Foundation and UIKit. The underlying implementation of these frameworks is still Objective-C, and was designed around the languages weird way of dealing with errors. Functions take an NSError pointer as an inout parameter, which can be checked after the function returns. By convention, these functions also return a boolean value, indicating whether the NSError pointer needs to be checked. From Apple's [documentation](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/ErrorHandling/ErrorHandling.html):

```objc
NSError *anyError;
BOOL success = [receivedData writeToURL:someLocalFileURL
                                options:0
                                  error:&anyError];
if (!success) {
    NSLog(@"Write failed with error: %@", anyError);
    // present error to user
}
```

## Possible solutions

To fix this, Ole [proposes a Result type](https://oleb.net/blog/2017/01/result-init-helper/). This is a really nice and Swift-y solution and i suspect at some point it will come to the standard library. I still wanted to show what i've been doing since before Swift even was a thing (so it's even compatible with Objective-C). For me, the easist solution often is to simply use separate success and failure handlers. For example, to request JSON data:

```swift
extension URLSession {
    func jsonTask<T: Decodable>(with request: URLRequest,
        successHandler: @escaping (T) -> Void,
        failureHandler: @escaping (Error) -> Void)
        -> URLSessionDataTask {
        return dataTask(with: request) { (data, response, error) in
            if let data = data,
               let object = try? JSONDecoder().decode(T.self, from: data) {
                successHandler(object)
                return
            }
            failureHandler(error
                ?? NSError(domain: "defaultErrorDomain", code: -1, userInfo: nil))
        }
    }
}
```

You could even go crazy and add more callbacks, if you want to other cases separately.
