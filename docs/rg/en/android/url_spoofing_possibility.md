# Possibility of URL spoofing

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

The vulnerability allows an attacker to control a URL in an instance of the `java.net.URL` class.  Depending on how a URL is used, this vulnerability can be exploited in different attack vectors.

Vulnerability is present if an instance of the `java.net.URL` class is created based on a string obtained from an untrusted source.

For example, a vulnerable application can use this code:

    if("https".equals(uri.getScheme()) && "vuln.app.pkg".equals(uri.getHost())) {
    String path = uri.getPath();
    if("/login".equals(path)) {
        String urlStr = uri.getQueryParameter("url");
        if(urlStr != null) {
        try {
            URL url = new URL(urlStr);
            /* .. */
        } catch (MalformedURLException e) {
            e.printStackTrace();
        }
        }
        finish();
    }
    }

## Recommendations

1. Perform URL validation:
   
   1. Allow access only to company resources, i.e., maintain a "white list" of URLs and check the created URL instance against it.
   
   2. Allow access only to particular origin, i.e. check scheme and domain of the URL.

2. The generated URL should also be checked before use.
   
        List<String> whiteHosts = Arrays.asList("stackoverflow.com",  "stackexchange.com", "google.com");
        public boolean isValid(String url) {
        String host = Uri.parse(url).getHost();
        if(whiteHosts.contains(host) {
            return true;
        }
        return false;
        }

## Links

1. [Input Validation - OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)

2. [UrlQuerySanitizer  \|  Android Developers](https://developer.android.com/reference/android/net/UrlQuerySanitizer)