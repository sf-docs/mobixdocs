# Enabled caching of network requests

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_vysokij.png"/></td>
        <td>Severity:<strong> HIGH</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> DAST, SENSITIVE INFO</strong></td>
      </tr>
    </tbody>
</table>
## Description

When using the standard library for networking in iOS, all network requests are cached in the file system of the device. These cache files may contain interesting information, including authentication requests containing all user credentials.

Despite the fact that this file is located in the internal directory of the application, it is not recommended to store session IDs and other sensitive data related to the authentication process in an open form.

## Recommendations

To disable caching of network requests, use one of the suggested methods, depending on the implementation:

1. Delete the shared cache at any time (for example, when you launch an application):
   
        URLCache.shared.removeAllCachedResponses()

2. Disable cache on a global level:
   
        let theURLCache = URLCache(memoryCapacity: 0, diskCapacity: 0, diskPath: nil)
        URLCache.shared = theURLCache

3. If you use a NSURLConnection object with a delegate, you can disable cache using the following method:
   
        func connection(_ connection: NSURLConnection, willCacheResponse cachedResponse: CachedURLResponse) -> CachedURLResponse?
            {
                return nil
            }

4. Create a URL request that does not use cache:
   
        var request = NSMutableURLRequest(url: theUrl, cachePolicy: .reloadIgnoringLocalCacheData, timeoutInterval: urlTimeoutTime)

5. Also, the NSURLRequest object has an attribute cachePolicy, which defines the work with cache:
   
   1. UseProtocolCachePolicy — default value, caching depends on HTTP headers.
   2. ReloadIgnoringLocalCacheData — cache is not used.

6. Finally, one of the easiest options is to simply clear the network query database or delete this file when opening/closing the application.

## Links

1. [https://codewithchris.com/preventing-nsurlconnection-cache-issues/](https://codewithchris.com/preventing-nsurlconnection-cache-issues/)
2. [https://developer.apple.com/forums/thread/45794](https://developer.apple.com/forums/thread/45794)
3. [https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/URLLoadingSystem/Concepts/CachePolicies.html)