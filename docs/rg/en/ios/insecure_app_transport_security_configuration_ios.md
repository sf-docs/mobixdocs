# Insecure App Transport Security configuration

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
        <td>Detection method:<strong> SAST, NETWORK</strong></td>
      </tr>
    </tbody>
</table>
## Description

The application uses an insecurely configured App Transport Security networking configuration. With the release of iOS 9.0, Apple introduced App Transport Security (ATS) as a mandatory setting for all applications. ATS configuration is a separate section inside the main Info.plist file of the application. The networking parameters are configured here. The ATS configuration has the following structure:

    NSAppTransportSecurity : Dictionary {
        NSAllowsArbitraryLoads : Boolean
        NSAllowsArbitraryLoadsForMedia : Boolean
        NSAllowsArbitraryLoadsInWebContent : Boolean
        NSAllowsLocalNetworking : Boolean
        NSExceptionDomains : Dictionary {
            : Dictionary {
                NSIncludesSubdomains : Boolean
                NSExceptionAllowsInsecureHTTPLoads : Boolean
                NSExceptionMinimumTLSVersion : String
                NSExceptionRequiresForwardSecrecy : Boolean   
                NSRequiresCertificateTransparency : Boolean
            }
        }
    }

An incorrect networking implementation makes the Man In The Middle attacks easier and decreases the security of your application.

## Recommendations

Below is a description of each of the parameters and the risks that can result from disabling each of the parameters.

In this case, it is recommended to fully enable ATS at the application level and without exceptions for domains.

Recommended configuration:

    <key>NSAppTransportSecurity</key>
    <dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <key>NSAllowsArbitraryLoadsForMedia</key>
    <false/>
    <key>NSAllowsArbitraryLoadsInWebContent</key>
    <false/>
    </dict>

Apple originally planned to require all apps on the App Store to support ATS by January 2017. Then Apple extended the deadline, but has yet to announce a new date.

### NSAllowArbitraryLoads

The NSAllowArbitraryLoads key defines the state of ATS in general - whether it is on or off for the application. NSAllowArbitraryLoads is set to NO by default. Setting the key to YES will completely disable ATS. This means that ATS will not prevent your application from communicating with any HTTP domains and no security checks will be applied. It is strongly not recommended to disable ATS, especially for the entire application.

If for some reason it is necessary to disable ATS, it is recommended to check additionally:

* Ciphers used for application networking (and that they are secure);
* The protocols used to send and receive data (and that they are secure);
* Whether there is a vulnerability in the application that allows switching to an earlier version of the encryption protocol;
* Whether the application checks the certificates used for TLS connections;

### NSAllowsLoadsForMedia

This exception applies to multimedia content protected by digital rights management (DRM) or encryption. NSAllowsLoadsForMedia is set to NO by default. If the NSAllowsLoadsForMedia key is set to YES, ATS is disabled for content sent using the AVFoundation framework. This usually happens with applications that have the ability to handle video or audio content.

If for some reason it is necessary to disable ATS, it is recommended to check additionally:

* The multimedia data sent by the application contains no confidential content and is protected by DRM or encryption;

It is better to implement these protection measures even if the content is transmitted over HTTPS, because it is very easy to intercept content transmitted over HTTP or other insecure protocols.

### NSAllowsArbitraryLoadsInWebContent

The NSAllowsArbitraryLoadsInWebContent key determines if it is possible to connect using insecure protocols from **WebView** components.

NSAllowsArbitraryLoadsInWebContent is set to NO by default. If the key is set to YES, ATS will be disabled for **WebView**.

Incorrect use of **WebView** can lead to various vulnerabilities in the application, so it is extremely important to secure these WebViews. For example, **WebView** can be vulnerable to a number of common web vulnerabilities, such as SQL injection, cross-site request forgery (CSRF), and cross-site scripting attacks (XSS). For more information about security when using **WebView**, see additional guidelines.

### NSAllowsLocalNetworking

The NSAllowsLocalNetworking key defines the ATS operation in the local network. NSAllowsLocalNetworking is set to NO by default. Typically, applications that connect to a local device to provide an Internet of Things (IoT) network use this exception. If you disable ATS, make sure that no sensitive data is transmitted on the local network and that a secure TLS connection is used.

### NSExceptionDomains

If you use the NSExceptionDomains key, you can configure ATS exceptions for specific domains. Remember that ATS subsections inside NSExceptionDomains replace other primary keys. For example, if an application loads media from a specific domain and both the top-level NSAllowsLoadsForMedia exception and the NSExceptionDomains configuration are used for that specific domain, then the NSExceptionDomains parameters take precedence. That is, they replace the top-level NSAllowsLoadsForMedia for that particular domain.

Second point: if a domain is specified in exceptions without any configuration, that domain will get full ATS protection even if NSAllowsArbitraryLoads is set to YES. That is, developer can disable ATS globally, but enable it for specific domains by specifying them in the NSExceptionDomains key. Disabling ATS for the entire application and enabling it only for certain domains is a configuration error. If you want to use insecure protocols for known domains, you have to add them to exceptions, and enable ATS globally for the entire application.

If you specify domain exceptions, ATS ignores any global configuration keys for that domain, such as NSAllowsArbitraryLoads. This works even if you leave the dictionary for the domain empty and rely entirely on the default values of its keys.

### NSIncludesSubdomains

This key defines whether the ATS policy will be applied to all subdomains.

NSIncludesSubdomains is set to NO by default. If the key is set to YES, any ATS configuration enabled for a particular domain will be used for all subdomains. If a domain is set, but no additional keys other than NSIncludesSubdomains are configured, that domain and its subdomains will use ATS.

### NSExceptionAllowsInsecureHTTPLoads

The NSExceptionAllowsInsecureHTTPLoad key defines whether insecure HTTP traffic can be transmitted to the specified domain.

NSExceptionAllowsInsecureHTTPLoad is set to NO by default. If the key is set to YES, the application will be allowed to send HTTP traffic to this domain.

### NSExceptionMinimumTLSVersion

This key allows you to decrease the minimum allowable version of TLS. By default, valid versions are TLS 1.2 and higher.

### NSExceptionRequiresForwardSecrecy

This key defines use of Forward Secrecy for a specific domain.

NSExceptionAllowsInsecureHTTPLoad is set to YES by default. If the key is set to NO, Forward Secrecy will be disabled for a particular domain.

### NSRequiresCertificateTransparency

This key defines use of Certificate Transparency for a specific domain.

NSExceptionAllowsInsecureHTTPLoad is set to NO by default. If the key is set to YES, the domain certificate will require a Certificate Transparency timestamp. Certificate Transparency is a Google project aimed at improving security of the SSL certificate issuance system. If your organization or the target domain supports Certificate Transparency, it is recommended that you enable this option. Certificate Transparency helps to identify fraudulent certificate authorities. It also helps prevent man-in-the-middle attacks by notifying the owner if the certificate has been compromised. If this key is enabled, Certificate Transparency related certificate checks will be performed before a connection is established.

### ATS is not a silver bullet

ATS is a client-side security measure that does not replace server-side security. Client-side security can be bypassed if an attacker has physical access to the device. While ATS protects iOS applications and their users by helping to prevent attacks on an earlier version of SSL and the use of weak ciphers, companies still need to protect the server side of the application. For example, by implementing HTTP Strict Transport Security (HSTS), disabling weak ciphers, etc. Client-side security reinforces server-side security and is just one layer of comprehensive mobile application security.

## Links

1. [https://developer.apple.com/documentation/security/preventing\_insecure\_network\_connections](https://developer.apple.com/documentation/security/preventing_insecure_network_connections)

2. [https://books.nowsecure.com/secure-mobile-development/en/webviews/](https://books.nowsecure.com/secure-mobile-development/en/webviews/)

3. [https://developer.apple.com/documentation/security/preventing\_insecure\_network\_connections](https://developer.apple.com/documentation/security/preventing_insecure_network_connections)

4. [https://www.certificate-transparency.org/](https://www.certificate-transparency.org/)

5. [https://developer.apple.com/documentation/avfoundation](https://developer.apple.com/documentation/avfoundation)

6. [https://developer.apple.com/news/?id=12212016b](https://developer.apple.com/news/?id=12212016b)

7. [https://www.owasp.org/index.php/Mobile\_Top\_10\_2016-M3-Insecure\_Communication](https://www.owasp.org/index.php/Mobile_Top_10_2016-M3-Insecure_Communication)

8. [https://cwe.mitre.org/data/definitions/319.html](https://cwe.mitre.org/data/definitions/319.html)