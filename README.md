OHHTTPStubs
===========

[![Version](http://cocoapod-badges.herokuapp.com/v/OHHTTPStubs/badge.png)](http://cocoadocs.org/docsets/OHHTTPStubs)
[![Platform](http://cocoapod-badges.herokuapp.com/p/OHHTTPStubs/badge.png)](http://cocoadocs.org/docsets/OHHTTPStubs)
[![Build Status](https://travis-ci.org/AliSoftware/OHHTTPStubs.png?branch=master)](https://travis-ci.org/AliSoftware/OHHTTPStubs)

`OHHTTPStubs` is a library designed to stub your network requests very easily. It can help you:

* test your apps with **fake network data** (stubbed from file) and **simulate slow networks**, to check your application behavior in bad network conditions
* write **Unit Tests** that use fake network data from your fixtures.

It works with `NSURLConnection`, new iOS7/OSX.9's `NSURLSession`, `AFNetworking` (both 1.x and 2.x), or any networking framework that use Cocoa's URL Loading System.

[![Donate](http://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=TRTU3UEWEHV92 "Donate")

----

# Documentation & Usage Examples

`OHHTTPStubs` headers are fully documented using Appledoc-like / Headerdoc-like comments in the header files. You can also [read the **online documentation** here](http://cocoadocs.org/docsets/OHHTTPStubs)
[![Version](http://cocoapod-badges.herokuapp.com/v/OHHTTPStubs/badge.png)](http://cocoadocs.org/docsets/OHHTTPStubs)


## Basic example

### In Objective-C

```objc
[OHHTTPStubs stubRequestsPassingTest:^BOOL(NSURLRequest *request) {
  return [request.URL.host isEqualToString:@"mywebservice.com"];
} withStubResponse:^OHHTTPStubsResponse*(NSURLRequest *request) {
  // Stub it with our "wsresponse.json" stub file (which is in same bundle as self)
  NSString* fixture = OHPathForFile(@"wsresponse.json", self.class);
  return [OHHTTPStubsResponse responseWithFileAtPath:fixture
            statusCode:200 headers:@{@"Content-Type":@"application/json"}];
}];
```

### In Swift

```swift
OHHTTPStubs.stubRequestsPassingTest({$0.URL!.host == "mywebservice.com"}) { _ in
  // Stub it with our "wsresponse.json" stub file (which is in same bundle as self)
  let fixture = OHPathForFile("wsresponse.json", self.dynamicType)
  return OHHTTPStubsResponse(fileAtPath: fixture!,
    statusCode: 200, headers: ["Content-Type":"application/json"])
}
```

## More examples & Help Topics
    
* For a lot more examples, see the dedicated "[Usage Examples](https://github.com/AliSoftware/OHHTTPStubs/wiki/Usage-Examples)" wiki page.
* The wiki also contain [some articles that can help you get started](https://github.com/AliSoftware/OHHTTPStubs/wiki) with (and troubleshoot if needed) `OHHTTPStubs`.

# Compatibility

`OHHTTPStubs` is compatible with **iOS 5.0+** and **OSX 10.7+**.

`OHHTTPStubs` also works with iOS7's and OSX 10.9's `NSURLSession` mechanism.

`OHHTTPStubs` is fully **Swift-compatible**. [Nullability annotations](https://developer.apple.com/swift/blog/?id=25) have been added to allow a cleaner API when used from Swift. —  If you use CocoaPods 0.36+, simply add `use_frameworks!` to your `Podfile` (see "Installing in your projects" below for more details).

# Installing in your projects

## Cocoapods

Simply add `pod 'OHHTTPStubs'` to your `Podfile` then run `pod install` and you are ready to use it.

If you want to use `OHHTTPStubs` from Swift:

* With CocoaPods 0.36+, simply add `use_frameworks!` to your Podfile to use dynamic frameworks. Then use `import OHHTTPStubs` in your Swift source.
* With CocoaPods 0.35, you have to have and Obj-C/Swift bridging file. As the import is done in briding-header, no import is needed in you Swift code.

## Carthage

Simply add `github "AliSoftware/OHHTTPStubs"` to your `Cartfile` then run `carthage update`.

# Special Considerations

## Using OHHTTPStubs in your Unit Tests

`OHHTTPStubs` is ideal to write Unit Tests that normally would perform network requests. But if you use it in your Unit Tests, don't forget to:

* remove any stubs you installed after each test — to avoid those stubs to still be installed when executing the next Test Case — by calling `[OHHTTPStubs removeAllStubs]` in your `tearDown` method. [see this wiki page for more info](https://github.com/AliSoftware/OHHTTPStubs/wiki/Remove-stubs-after-each-test)
* be sure to wait until the request has received its response before doing your assertions and letting the test case finish (like for any asynchronous test). [see this wiki page for more info](https://github.com/AliSoftware/OHHTTPStubs/wiki/OHHTTPStubs-and-asynchronous-tests)

## Automatic loading

Thanks to method swizzling, `OHHTTPStubs` is automatically loaded and installed both for:

* requests made using `NSURLConnection` or `[NSURLSession sharedSession]`;
* requests made using a `NSURLSession` created using a `[NSURLSessionConfiguration defaultSessionConfiguration]` or `[NSURLSessionConfiguration ephemeralSessionConfiguration]` configuration (using `[NSURLSession sessionWithConfiguration:…]`-like methods).

If you need to disable (and re-enable) `OHHTTPStubs` — globally or per `NSURLSession` — you can use `[OHHTTPStubs setEnabled:]` / `[OHHTTPStubs setEnabled:forSessionConfiguration:]`.

## Known limitations

* `OHHTTPStubs` **can't work on background sessions** (sessions created using `[NSURLSessionConfiguration backgroundSessionConfiguration]`) because background sessions don't allow the use of custom `NSURLProtocols` and are handled by the iOS Operating System itself.
* `OHHTTPStubs` don't simulate data upload. The `NSURLProtocolClient` `@protocol` does not provide a way to signal the delegate that data has been **sent** (only that some has been loaded), so any data in the `HTTPBody` or `HTTPBodyStream` of an `NSURLRequest`, or data provided to `-[NSURLSession uploadTaskWithRequest:fromData:];` will be ignored, and more importantly, the `-URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:` delegate method will never be called when you stub the request using `OHHTTPStubs`.

_As far as I know, there's nothing we can do about those two limitations. Please let me know if you know a solution that would make that possible anyway._


## Submitting to the AppStore

`OHHTTPStubs` **can be used** on apps subbmiting **on the AppStore**. It does not use any private API and nothing prevents you from shipping it.

But you generally only use stubs during the development phase and want to remove your stubs when submitting to the AppStore. So be careful to only include `OHHTTPStubs` when needed (only in your test targets, or only inside `#if DEBUG` portions, or by using [per-Build-Configuration pods](https://guides.cocoapods.org/syntax/podfile.html#pod)) to avoid forgetting to remove it when the time comes that you release for the AppStore and you want your requests to hit the net!



# License and Credits

This project and library has been created by Olivier Halligon (@aligatr on Twitter) and is under the MIT License.

It has been inspired by [this article from InfiniteLoop.dk](http://www.infinite-loop.dk/blog/2011/09/using-nsurlprotocol-for-injecting-test-data/).
I would also like to thank to @kcharwood for its contribution, and everyone who contributed to this project on GitHub.

If you want to support the development of this library, feel free to [![Donate](http://www.paypalobjects.com/en_US/i/btn/btn_donate_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=TRTU3UEWEHV92 "Donate"). Thanks to all contributors so far!
