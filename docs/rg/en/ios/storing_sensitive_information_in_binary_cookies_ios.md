# Storing sensitive information in Binary Cookies

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_info.png"/></td>
        <td>Severity:<strong> INFO</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> DAST, ФАЙЛЫ ПРИЛОЖЕНИЯ</strong></td>
      </tr>
    </tbody>
</table>
## Description

Application does not disable storage of Cookies when working with **WebView**.

When using **WebView**, you need to remember that Cookies values received when visiting a web site are saved locally in a file with a special format `Cookies.binarycookies`. This applies to both UIWebView and WKWebView. These Cookies values can also contain sensitive data, such as session IDs or tokens for accessing a service.

## Recommendations

For proper use, it is necessary to limit storage of Cookies to the time of their use. Usually when you close **WebView** this data is no longer needed. Better yet, disable Cookies altogether.

Below is an example with storage disabled:

    import UIKit
    import WebKit
    class ViewController: UIViewController, WKUIDelegate {
        var webView: WKWebView!
        override func loadView()
            let webConfiguration = WKWebViewConfiguration()
            webView = WKWebView(frame: .zero, configuration: webConfiguration)
            webView.uiDelegate = self
            view = webView
        }
        override func viewDidLoad() {
            super.viewDidLoad()
            let myURL = URL(string: "http://example.com")
            var myRequest = URLRequest(url: myURL!)
            myRequest.httpShouldHandleCookies = false
            webView.load(myRequest)
        }
    }

To delete Cookie values (e.g. when logging out of an application or closing/opening it):

    let storage = HTTPCookieStorage.shared
            if let cookies = storage.cookies{
                for cookie in cookies {
                    storage.deleteCookie(cookie)
                }
        }

## Links

1. [https://book.hacktricks.xyz/mobile-apps-pentesting/ios-pentesting#cookies](https://book.hacktricks.xyz/mobile-apps-pentesting/ios-pentesting#cookies)
2. [https://developer.apple.com/forums/thread/81874](https://developer.apple.com/forums/thread/81874)