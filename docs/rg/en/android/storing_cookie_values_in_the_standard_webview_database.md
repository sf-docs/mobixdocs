# Storing Cookie values in the standard WebView database

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

If you use **WebView** with default settings (or improperly configured), all Cookie values handled by **WebView** will be stored in the application directory in a special Cookie.db database. The values in this database are not encrypted in any way and are openly available.

Despite the fact that this file is located in the internal directory of the application, it is not recommended to store session IDs and other sensitive data related to the authentication process in an open form. There are several ways to get this file, from application backup, to exploiting other vulnerabilities allowing to read any files in the application directory.

## Recommendations

To prevent storing Cookie values, you should configure **WebView** correctly when you create it. In case the values have been saved previously, they must be deleted.

It is important to keep in mind that this operation is performed differently on different versions of the Android SDK. Here is an example of code that implements correct Cookie clearing for all Android versions:

    webView.clearCache(true);
        webView.clearHistory();
        WebSettings webSettings = webView.getSettings();
        webSettings.setSaveFormData(false);
        
        // Not needed for API level 18 or greater (deprecated)
        webSettings.setSavePassword(false); 
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP_MR1) {
            CookieManager.getInstance().removeAllCookies(null);
            CookieManager.getInstance().flush();
        } else {
            CookieSyncManager cookieSyncMngr = CookieSyncManager.createInstance(this);
            cookieSyncMngr.startSync();
            CookieManager cookieManager = CookieManager.getInstance();
            cookieManager.removeAllCookie();
            cookieManager.removeSessionCookie();
            cookieSyncMngr.stopSync();
            cookieSyncMngr.sync();
        }

## Links

1. [https://developer.android.com/reference/android/webkit/CookieManager#removeAllCookies(android.webkit.ValueCallback\<java.lang.Boolean>)](https://developer.android.com/reference/android/webkit/CookieManager#removeAllCookies(android.webkit.ValueCallback<java.lang.Boolean>))

2. [https://www.owasp.org/index.php/Mobile\_Top\_10\_2016-M2-Insecure\_Data\_Storage](https://www.owasp.org/index.php/Mobile_Top_10_2016-M2-Insecure_Data_Storage)