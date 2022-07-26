# Insecure networking configuration

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

The application uses insecurely tuned networking configuration. With the release of Android 6.0 Marshmallow, Google introduced the manifest attribute: `android: usesCleartextTraffic`, to protect against accidental use of the HTTP protocol. Android 7.0 Nougat expanded this attribute by introducing an Android networking security configuration feature that allows developers to more clearly define connection settings. The network communication configuration is an XML file that configures the network security settings for the Android application. This setting is specified by a special attribute in ***AndroidManifest.xml*** — `android:networkSecurityConfig`.

**Connection example:**

    <?xml version="1.0" encoding="utf-8"?>
    <manifest ... >
    <application android:networkSecurityConfig="@xml/network_security_config"
    ... >
    ...
    </application>
    </manifest>

## Recommendations

The Network Security Config feature allows you to configure network communication settings in a declarative configuration file without changing the application code. These settings can be configured for specific domains and for a specific application. The key opportunities provided by this approach:

* **Custom trust anchors:** Configure the certification authorities (CA) that the application will trust during network communication. For example, trusting certain self-signed certificates or restricting the set of public CAs that the application trusts (those added to the trusted ones by the user or those CAs that the system trusts).
* **Debug-only overrides:** Configuring connections specifically for the debug version of the application. This allows you to distinguish between development and production environments.
* **Cleartext traffic:** Settings to allow or deny connections via HTTP.
* **Certificate pinning:** Configuration for SSL Pinning implementation.

In the configuration file, you can apply settings for the application in the `base-config` block, only for specific domains and subdomains in the `domain-config` block and separately for the debug build of the application in the `debug-overrides` block.

Let's consider examples of settings that are available for customization:

**Custom trust anchors: Configuring trust to certification authorities**

Android’s network security configuration gives developers several options for choosing which CAs to trust. By default in Android 7+ (Nougat, Oreo and Pie), the application will only trust certificates that are marked as system:

    <network-security-config>
    <base-config>
        <trust-anchors>
        <certificates src="system"/>
        </trust-anchors>
    </base-config>
    </network-security-config>

In Android 6 (Marshmallow) and lower, the application also trusts the certificates installed by the user. With this setting it is possible to perform a MiTM (man in the middle) attack if the user installs his certificate:

    <network-security-config>
    <base-config>
        <trust-anchors>
        <certificates src="system"/>
        <certificates src="user"/>
        </trust-anchors>
    </base-config>
    </network-security-config>

It is also possible to enable trust in certificates located in the application's assets:

    <network-security-config>
    <base-config>
        <trust-anchors>
        <certificates src="@raw/my_custom_ca"/>
        </trust-anchors>
    </base-config>
    </network-security-config>

The most secure option is the first one, where the application trusts only the system certificates.

**Configuring HTTP communication**

By splitting into separate configurations for the entire application, individual domains, and the debug version, it is possible to control the permissions for using HTTP at each level. Let's take a closer look: in the first example, HTTP data transfer is forbidden at the application level (this is the most secure option):

    <network-security-config>
    <base-config cleartextTrafficPermitted="false" />
    ...
    </network-security-config>

In some exceptional cases it is necessary to connect via HTTP to certain domains. Exactly for this case there is an opportunity to specify separately what is allowed for different domains:

    <network-security-config>
    <domain-config cleartextTrafficPermitted="true">
    <domain includeSubdomains="true">insecure.example.com</domain>
    <domain includeSubdomains="true">insecure.cdn.example2.com</domain>
    </domain-config>
    <base-config cleartextTrafficPermitted="false" />
    </network-security-config>

This setting will only allow HTTPS access for all network connections except for the two that are allowed to communicate over HTTP (**insecure.example.com** and **insecure.cdn.example2.com**), including their subdomains. The **includeSubdomains** setting is responsible for enabling subdomains.

**Certificate Pinning settings**

Network Security Config allows you to easily connect the Certificate Pinning mechanism to the application. However, it is worth to take into account certain nuances. Let's look at a configuration that at first glance looks like a properly configured one and see how it can be slightly improved:

    <network-security-config>
    <domain-config>
    <domain includeSubdomains="true">example.com</domain>
    <pin-set>
    <pin digest="SHA-256">7HIpactkIAq2Y49orFOOQKurWxmmSFZhBCoQYcRhJ3Y=</pin>
    </pin-set>
    </domain-config>
    </network-security-config>

This example has two small disadvantages:

1. The certificate fingerprint (pin-set) does not have an expiration date.
2. There is no backup certificate.

If your certificate is about to expire and no expiration date is specified in the settings, the application will stop connecting to the server and generate an error. However, if an expiration date has been set and it expires, the application will switch to the trusted certificate authorities installed in the system. Instead of getting a non-functional application, you will get no SSL Pinning for some time until you update the certificate in the application.

To avoid this, if you know the certificate to be used on your server after the current one expires, you can specify it immediately in the "backup certificates" settings.

Here is an example of the most correct use of the Certificate Pinning functionality:

    <network-security-config>
    <domain-config>
    <domain includeSubdomains="true">example.com</domain>
    <pin-set expiration="2021-01-01">
    <pin digest="SHA-256">7HIpactkIAq2Y49orFOOQKurWxmmSFZhBCoQYcRhJ3Y=</pin>
    <!-- backup pin -->
    <pin digest="SHA-256">fwza0LRMXouZHRC8Ei+4PyuldPDcf3UKgO/04cDM1oE=</pin>
    </pin-set>
    </domain-config>
    </network-security-config>

Despite all the conveniences of using the Network Security Config, some checks should be performed internally in the application code. For example, you still need to determine if your application performs hostname validation, because Network Security Config does not protect against this type of issue.

!!! note "Note" Also, before implementation, make sure that third-party libraries support Network Security Config. Otherwise, these security measures may cause problems in your application. Besides, Network Security Config is not supported by lower-level network connections, such as web sockets.

## Links

1. [https://developer.android.com/guide/topics/manifest/application-element#usesCleartextTraffic](https://developer.android.com/guide/topics/manifest/application-element#usesCleartextTraffic)

2. [https://developer.android.com/training/articles/security-config](https://developer.android.com/training/articles/security-config)

3. [https://www.nowsecure.com/blog/2018/08/15/a-security-analysts-guide-to-network-security-configuration-in-android-p/](https://www.nowsecure.com/blog/2018/08/15/a-security-analysts-guide-to-network-security-configuration-in-android-p/)