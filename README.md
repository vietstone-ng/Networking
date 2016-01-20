![Networking](https://raw.githubusercontent.com/3lvis/Networking/master/Images/cover-v3.png)

[![Version](https://img.shields.io/cocoapods/v/Networking.svg?style=flat)](https://cocoapods.org/pods/Networking)
[![License](https://img.shields.io/cocoapods/l/Networking.svg?style=flat)](https://cocoapods.org/pods/Networking)
[![Platform](https://img.shields.io/cocoapods/p/Networking.svg?style=flat)](https://cocoapods.org/pods/Networking)

- Super friendly API
- Singleton free
- Optimized for unit testing
- Minimal implementation
- Easy stubbing
- Runs synchronously in automatic testing enviroments
- Image download and caching
- Free

## Table of Contents

* [Configuration types](#configuration-types)
* [Authentication](#authentication)
    * [HTTP basic](#http-basic)
    * [Bearer token](#bearer-token)
* [Making a request](#making-a-request)
* [Content Types](#content-types)
* [Stubbing](#stubbing)
* [Cancelling](#cancelling)
* [Image download](#image-download)
* [Error logging](#error-logging)
* [Network Activity Indicator](#network-activity-indicator)
* [Installation](#installation)
* [Author](#author)
* [License](#license)
* [Attribution](#attribution)

## Configuration types

**Networking** is basically a wrapper of NSURLSession, the great thing about this is that we can leverage to the great configuration types supported by NSURLSession, such as the default one, ephemeral and background.

When initializing your instance of **Networking** you can provide a `NetworkingConfigurationType`.

 - `Default`: If you don't provide any option this one is used. This configuration type manages upload and download tasks using the default options.
 - `Ephemeral`: A configuration type that uses no persistent storage for caches, cookies, or credentials.
 It's optimized for transferring data to and from your app’s memory.
 - `Background`: A configuration type that allows HTTP and HTTPS uploads or downloads to be performed in the background.
 It causes upload and download tasks to be performed by the system in a separate process.

## Authentication

### HTTP basic

To authenticate using [basic authentication](http://www.w3.org/Protocols/HTTP/1.0/spec.html#BasicAA) with a username **"aladdin"** and password **"opensesame"**, you would need to set the following header field: 

`Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`, which contains the string `aladin:opensesame` in Base64 format. 

Luckily, **Networking** provides a simpler way to do this:

```swift
let networking = Networking(baseURL: "http://httpbin.org")
networking.authenticate("aladdin", password: "opensesame")
networking.GET("/basic-auth/aladdin/opensesame", completion: { JSON, error in
    // Successfully logged in! Now do something with the JSON
})
```

### Bearer token

To authenticate using a [bearer token](https://tools.ietf.org/html/rfc6750) **"AAAFFAAAA3DAAAAAA"**, you would need to set the following header field:  `Authorization: Bearer AAAFFAAAA3DAAAAAA`. 

Luckily, **Networking** provides a simpler way to do this:


```swift
let networking = Networking(baseURL: "http://sample.org")
networking.authenticate("AAAFFAAAA3DAAAAAA")
networking.GET("/users", completion: { JSON, error in
    // Do something...
})
```

## Making a request
  
Making a request is as simple as just calling `GET`, `POST`, `PUT`, or `DELETE`. The only difference between this calls is that `GET` and `DELETE` don't require parameters while the other ones do.  

**GET example**:

```swift
let networking = Networking(baseURL: "https://api-news.layervault.com/api/v2")
networking.GET("/stories", completion: { JSON, error in
  if let JSON = JSON {
    // Stories JSON: https://api-news.layervault.com/api/v2/stories
  }
})
```

**POST example**:

```swift
let networking = Networking(baseURL: "http://httpbin.org")
networking.POST("/post", params: ["username":"jameson", "password":"password"]) { JSON, error in
    /*
    JSON Pretty Print:
    {
        "json" : {
            "username" : "jameson",
            "password" : "password"
        },
        "url" : "http://httpbin.org/post",
        "data" : "{"password":"password","username":"jameson"}",
        "headers" : {
            "Accept" : "application/json",
            "Content-Type" : "application/json",
            "Host" : "httpbin.org",
            "Content-Length" : "44",
            "Accept-Language" : "en-us"
        }
    }
    */
}
```

## Content Types

**Networking** by default uses `application/json` as the `Content-Type` and `Accept` headers. You can use other content types by proving the `ContentType` attribute. Internally **Networking** will format your parameters so they use [`Percent-encoding`](https://en.wikipedia.org/wiki/Percent-encoding#The_application.2Fx-www-form-urlencoded_type).

```swift
let networking = Networking(baseURL: "http://httpbin.org")
networking.POST("/post", contentType: .FormURLEncoded, parameters: ["custname":"jameson"]) { JSON, error in
   // Successfull post using `application/x-www-form-urlencoded` as `Content-Type`
}
```

## Stubbing

Stubbing a request means that after calling this method on a specific path, any call to this resource, will return what you registered as a response. 

**Stubbing with successfull response**:

```swift
let networking = Networking(baseURL: "https://api-news.layervault.com/api/v2")
networking.stubGET("/stories", response: [["id" : 47333, "title" : "Site Design: Aquest"]])
networking.GET("/stories", completion: { JSON, error in
    // JSON containing stories
})
```

**Stubbing with contents of a file**:

If your file is not located in the main bundle you have to specify using the bundle parameters, otherwise `NSBundle.mainBundle()` will be used.

```swift
let networking = Networking(baseURL: baseURL)
networking.stubGET("/entries", fileName: "entries.json")
networking.GET("/entries", completion: { JSON, error in
    // JSON with the contents of entries.json
})
```

**Stubbing with status code**:

If you do not provide a status code for this stub, the default returned one will be 200 (SUCCESS), but if you do provide a status code that is not 2XX, then **Networking** will return an NSError containing the status code and a proper error description.

```swift
let networking = Networking(baseURL: "https://api-news.layervault.com/api/v2")
networking.stubGET("/stories", response: nil, statusCode: 500)
networking.GET("/stories", completion: { JSON, error in
    // error with status code 500
})
```

## Cancelling

Cancelling any request for a specific path is really simple. Beware that cancelling a request will cause the request to return with an error with status code -999.

```swift
let networking = Networking(baseURL: "http://httpbin.org")
networking.GET("/get", completion: { JSON, error in
    // Cancelling a GET request returns an error with code -999 which means cancelled request
})

networking.cancelGET("/get")
```

## Image download

```swift
let networking = Networking(baseURL: "http://httpbin.org")
networking.downloadImage("/image/png") { image, error in
   // Do something with the downloaded image
}
```

**Stubbing**:

```swift
let networking = Networking(baseURL: baseURL)
let pigImage = UIImage(named: "pig.png")!
networking.stubImageDownload("/image/png", image: pigImage)
networking.downloadImage("/image/png") { image, error in
   // Here you'll get the stubbed pig.png image
}
```

**Cancelling**:

```swift
let networking = Networking(baseURL: baseURL)
networking.downloadImage("/image/png") { image, error in
    // Cancelling a image download returns an error with code -999 which means cancelled request
}

networking.cancelImageDownload("/image/png")
```

**Caching**:

**Networking** stores the download image in the Caches folder. It also uses NSCache internally so it doesn't have to download the same image again and again.

If you want to remove the downloaded image so it downloads again. You can do it like this:

```swift
let networking = Networking(baseURL: "http://httpbin.org")
let destinationURL = networking.destinationURL("/image/png")
if let path = destinationURL.path where NSFileManager().fileExistsAtPath(path) {
   try! NSFileManager().removeItemAtPath(path)
}
```

## Error logging

Any error catched by **Networking** will be printed in your console. This is really convenient since you want to know why your networking call failed anyway.

For example a cancelled request will print this:

```
========== Networking Error ==========

Cancelled request: GET https://ddxxx.com/38bea9c8b75bfed1326f90c48675fce87dd04ae6/thumb/small

================= ~ ==================
```

A 404 request will print something like this:

```
========== Networking Error ==========
 
Error 3840: Error Domain=NSCocoaErrorDomain Code=3840 "JSON text did not start with array or object and option to allow fragments not set." UserInfo={NSDebugDescription=JSON text did not start with array or object and option to allow fragments not set.}
 
Request: <NSMutableURLRequest: 0x7fede8d17220> { URL: http://httpbin.org/invalidpath }
 
Data: <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<title>404 Not Found</title>
<h1>Not Found</h1>
<p>The requested URL was not found on the server.  If you entered the URL manually please check your spelling and try again.</p>

 
Response status code: 404
 
Path: http://httpbin.org/invalidpath
 
Response: <NSHTTPURLResponse: 0x7fede8d0c4e0> { URL: http://httpbin.org/invalidpath } { status code: 404, headers {
    "Access-Control-Allow-Credentials" = true;
    "Access-Control-Allow-Origin" = "*";
    Connection = "keep-alive";
    "Content-Length" = 233;
    "Content-Type" = "text/html";
    Date = "Tue, 17 Nov 2015 09:59:42 GMT";
    Server = nginx;
} }
 
================= ~ ==================
```

## Network Activity Indicator

**Networking** leverages on [NetworkActivityIndicator](https://github.com/3lvis/NetworkActivityIndicator) to balance how the network activity indicator is displayed.

> A network activity indicator appears in the status bar and shows that network activity is occurring.
>The network activity indicator:
>
> - Spins in the status bar while network activity proceeds and disappears when network activity stops
> - Doesn’t allow user interaction
>
> Display the network activity indicator to provide feedback when your app accesses the network for more than a couple of seconds. If the operation finishes sooner than that, you don’t have to show the network activity indicator, because the indicator is likely to disappear before users notice its presence.
>
>— [iOS Human Interface Guidelines](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/Controls.html)

<p align="center">
  <img src="https://raw.githubusercontent.com/3lvis/NetworkActivityIndicator/master/GIF/sample.gif"/>
</p>

## Installation

**Networking** is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
use_frameworks!

pod 'Networking'
```

## Author

Elvis Nuñez, [@3lvis](https://twitter.com/3lvis)


## License

**Networking** is available under the MIT license. See the [LICENSE file](https://github.com/3lvis/Networking/blob/master/LICENSE.md) for more info.


## Attribution

The logo typeface comes thanks to [Sanid Jusić](https://dribbble.com/shots/1049674-Free-Colorfull-Triangle-Typeface)
