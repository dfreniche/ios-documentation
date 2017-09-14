theme: Plain jane, 3
autoscale: true
footer: (c) Diego Freniche / @dfreniche / www.freniche.com

# Core Networking

---


## Networking Libraries options

- Foundation Layer		
- Core Foundation Layer
	
---

## Foundation Layer
 	
- Protocol streams: NSStream
- URL Loading: NSURLConnection / NSURLSession and NSURLRequest
- Service discovery: NSNetService

---

## Core Foundation Layer
- Protocol streams: CFStream
- URL Loading: CFHTTPMessage
- Service discovery: CFNetService
 
---

## Which one?

- __Always the most high level one__: Foundation Layer
- Exceptions:
	- writing server code
	- special needs
	
---
	
## Peer to peer networking (games)

- Game Kit framework
- support for peer-to-peer communication
	- globally (over the Internet)
	- locally (using a Bluetooth personal area network or a Wi-Fi LAN)

---

## Peer-to-peer networking (for other apps)

- Multipeer Connectivity framework 
	- support for peer-to-peer communication
		- infrastructure Wi-Fi
		- peer-to-peer Wi-Fi
		- Bluetooth.

---

## Connect to a web server

_The preferred way to send and receive short pieces of information is over a standard protocol such as HTTP or HTTPS_


---

## App Transport Security [^1]

From [What's new in iOS 9](https://developer.apple.com/library/prerelease/ios/releasenotes/General/WhatsNewIniOS/Articles/iOS9.html#//apple_ref/doc/uid/TP40016198-SW1)

> App Transport Security (ATS) lets an app add a declaration to its Info.plist file that specifies the domains with which it needs secure communication.

- declare domains that doesn't support TLS
- bypass ATS altogether

[^1]: http://ste.vn/2015/06/10/configuring-app-transport-security-ios-9-osx-10-11/

---

## App Transport Security

Edit your project's plist as __raw__. Add this key:


```xml
<key>NSAppTransportSecurity</key>
<dict>
  <!--Include to allow all connections (DANGER)-->
  <key>NSAllowsArbitraryLoads</key>
      <true/>
</dict>
```


---

## NSURLSession vs NSURLConnection

NSURLConnection: OLD Foundation URL Loading System

- NSURLRequest
- NSURLResponse
- NSURLProtocol
- NSURLCache
- NSHTTPCookieStorage
- NSURLCredentialStorage
- ...


---

## NSURLSession

- iOS 7 / OS X 10.9
- NSURLSession class and related classes provide an API for downloading content via HTTP.
- can configure per-session cache, protocol, cookie, and credential policies, rather than sharing them across the app
- A NSURLSession is made using an NSURLSessionConfiguration with an optional delegate. After you create the session you then satisfy you networking needs by creating NSURLSessionTask’s.


---

## Tasks

- three types of tasks: __data__ tasks, __upload__ tasks, and __download__ tasks.
- data tasks for retrieving data to memory
- download tasks for downloading a file to disk
- upload tasks: uploading a file from disk and receiving the response as data in memory

---

## Code (Objective C)

```objectivec
// using NSURLSessionDataTask
NSURLSession *session = [NSURLSession sharedSession];
[[session dataTaskWithURL:[NSURL URLWithString:londonWeatherUrl] 
completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
		// handle response
}] resume]; // After you create the task, you must start it by calling its resume method.
	
// http://api.openweathermap.org/data/2.5/weather?q=El%20Saucejo,es
```  
  
---

## Code (Swift)

```swift
let session = URLSession.shared
let task = session.dataTask(with: url!) { (data: Data?, response: URLResponse?, error: Error?) in
	// handle response
}
task.resume() // After you create the task, you must start it by calling its resume method.
```
  


  
---  

## NSURLSession vs AFNetworking
  
- Is this better than AFNetworking?
- AFNetworking: external dependency, added via CocoaPods
- NSURLSession: comes with Cocoa. It's AFNetworking sherlock'ed

---

## NSURLSessionTask

![inline](http://www.objc.io/images/issue-5/NSURLSession.png)

---

## NSUrlSessionDataTask (Objective C)

```objectivec
NSURL *URL = [NSURL URLWithString:@"http://example.com"];
NSURLRequest *request = [NSURLRequest requestWithURL:URL];

NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDataTask *task = [session dataTaskWithRequest:request
									completionHandler:
^(NSData *data, NSURLResponse *response, NSError *error) {
	// ...
}];

[task resume];
```

---

## NSUrlSessionDataTask (Swift)

```swift
let urlString = "https://deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1"
let url = URL(string: urlString)

let session = URLSession.shared
let task = session.dataTask(with: url!) { (data: Data?, response: URLResponse?, error: Error?) in
	do {
		let j = try JSONSerialization.jsonObject(with: data!, options: JSONSerialization.ReadingOptions.allowFragments) as! Dictionary<String, Any>
		print(j["deck_id"])

	} catch {
		
	}
}
task.resume()

```

---

## NSURLSessionUploadTask

```objectivec
NSURL *URL = [NSURL URLWithString:@"http://example.com/upload"];
NSURLRequest *request = [NSURLRequest requestWithURL:URL];
NSData *data = ...;

NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionUploadTask *uploadTask = [session uploadTaskWithRequest:request
															fromData:data
													completionHandler:
	^(NSData *data, NSURLResponse *response, NSError *error) {
		// ...
	}];
	
[uploadTask resume];
```


---

## NSURLSessionDownloadTask

```objectivec
NSURL *URL = [NSURL URLWithString:@"http://example.com/file.zip"];
NSURLRequest *request = [NSURLRequest requestWithURL:URL];

NSURLSession *session = [NSURLSession sharedSession];
NSURLSessionDownloadTask *downloadTask = [session downloadTaskWithRequest:request
														completionHandler:
	^(NSURL *location, NSURLResponse *response, NSError *error) {
		NSString *documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
		NSURL *documentsDirectoryURL = [NSURL fileURLWithPath:documentsPath];
		return [documentsDirectoryURL URLByAppendingPathComponent:[[response URL] lastPathComponent]];
	}];
	
[downloadTask resume];
```

---

## Download image

```objectivec
NSURLSessionDownloadTask *getImageTask =
[session downloadTaskWithURL:[NSURL URLWithString:imageUrl]

	completionHandler:^(NSURL *location,
						NSURLResponse *response,
						NSError *error) {
				UIImage *downloadedImage =
			[UIImage imageWithData:
				[NSData dataWithContentsOfURL:location]];
		
		dispatch_async(dispatch_get_main_queue(), ^{
		// do stuff with image
		_imageWithBlock.image = downloadedImage;
		});
}];


[getImageTask resume];
```

 ---


## NSURLSessionConfiguration

- defaultSessionConfiguration – creates a configuration object that uses the global cache, cookie and credential storage objects. This is a configuration that causes your session to be the most like NSURLConnection.
- ephemeralSessionConfiguration – this configuration is for “private” sessions and has no persistent storage for cache, cookie, or credential storage objects.
- backgroundSessionConfiguration – this is the configuration to use when you want to make networking calls from remote push notifications or while the app is suspended. 

---

Once you create a NSURLSessionConfiguration, you can set various properties on it like this:

```objectivec
NSURLSessionConfiguration *sessionConfig =
[NSURLSessionConfiguration defaultSessionConfiguration];
 
// 1
sessionConfig.allowsCellularAccess = NO;
 
// 2
[sessionConfig setHTTPAdditionalHeaders:
          @{@"Accept": @"application/json"}];
 
// 3
sessionConfig.timeoutIntervalForRequest = 30.0;
sessionConfig.timeoutIntervalForResource = 60.0;
sessionConfig.HTTPMaximumConnectionsPerHost = 1;
```

Here you restrict network operations to wifi only.
This will set all requests to only accept JSON responses.
These properties will configure timeouts for resources or requests. Also you can restrict your app to only have one network connection to a host.
 
--- 

