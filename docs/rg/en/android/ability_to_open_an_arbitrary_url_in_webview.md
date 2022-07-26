# Possibility to open an arbitrary URL in WebView

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_srednij.png"/></td>
        <td>Severity:<strong> MEDIUM</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> IAST</strong></td>
      </tr>
    </tbody>
</table>
## Description

Vulnerability allows to open arbitrary URLs in an application **WebView**. Depending on **WebView** settings, this vulnerability can be exploited in different attack vectors.

The vulnerability is present in applications that use data from an untrusted source to form a URL, which is then used in calls to the `WebView.loadUrl` method.

For example, a vulnerable application can use this code:

    WebView webView = findViewById(R.id.webview);
    setupWebView(webView);
    webView.loadUrl(getIntent().getStringExtra("url"));

Or this one:

    String url = uri.getQueryParameter("url");
    if(url != null) {
    webView.loadUrl(url);
    }

## Recommendations

1. Do not use dynamically generated URLs for **WebView**.
   
        webView.loadUrl("https://url.to.your.contents/");

2. Perform URL validation:
   
   1. Allow access only to company resources, i.e. maintain a white-list of URLs and check the URL passed to the `loadUrl` method against it.
   
   2. Allow access only to particular origin, i.e. check scheme and domain of the URL.
      
           List<String> whiteHosts = Arrays.asList("stackoverflow.com",  "stackexchange.com", "google.com");
           public boolean isValid(String url) {
           String host = Uri.parse(url).getHost();
           if(whiteHosts.contains(host) {
               return true;
           }
           return false;
           }

3. Disable potentially dangerous settings for accessing application resources from **WebView**:

`WebSettings.setAllowContentAccess(false)`<br>

`WebSettings.setAllowFileAccess(false)`<br>

`WebSettings.setAllowFileAccessFromFileURLs(false)`<br>

`WebSettings.setAllowUniversalAccessFromFileURLs(false)`<br>

`WebSettings.setGeolocationEnabled(false)`<br>

## Links

1. [Android security checklist:](https://blog.oversecured.com/Android-security-checklist-webview/)[ ](https://blog.oversecured.com/Android-security-checklist-webview/)[WebView](https://blog.oversecured.com/Android-security-checklist-webview/)

2. [UrlQuerySanitizer  \|  Android Developers](https://developer.android.com/reference/android/net/UrlQuerySanitizer)