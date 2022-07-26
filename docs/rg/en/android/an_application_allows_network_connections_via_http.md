# An application allows network connections via HTTP

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
        <td>Detection method:<strong> SAST, MANIFEST</strong></td>
      </tr>
    </tbody>
</table>
## Description

The **AndroidManifest** settings have the `android:usesCleartextTraffic=true` attribute, allowing the application to communicate with any servers via the unsecured HTTP protocol. This setting depends on several parameters and its default value also depends on targetSDK specified in the application manifest:

1. If the `android:networkSecurityConfig` attribute is present in AndroidManifest, the `android:usesCleartextTraffic` value is ignored, since all network settings are defined inside the network configuration file.
2. If targetSDK =\< 27, the default attribute value is **true** (if not specified in the manifest).
3. If targetSDK >= 28, the default attribute value is **false** (if not specified in the manifest).

**An example of a vulnerable configuration:**

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.appsec.android.activity.privateactivity" >
    <application
    <!-- *** Включенние отладочного режима *** -->
    android:debuggable="true"
    android:icon="@drawable/ic_launcher"
    android:usesCleartextTraffic="true"
    android:label="@string/app_name" >
    <activity
    android:name=".PrivateActivity"
    android:label="@string/app_name"
    android:exported="false" />
    </application>
    </manifest>

## Recommendations

It is recommended to explicitly disable the ability to transfer data over an unsecured HTTP protocol by setting the `android:usesCleartextTraffic` attribute to `false`.

**An example of a secure configuration:**

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.appsec.android.activity.privateactivity" >
    <application
    <!-- *** Включенние отладочного режима *** -->
    android:debuggable="true"
    android:icon="@drawable/ic_launcher"
    android:usesCleartextTraffic="false"
    android:label="@string/app_name" >
    <activity
    android:name=".PrivateActivity"
    android:label="@string/app_name"
    android:exported="false" />
    </application>
    </manifest>

## Links

1. [https://developer.android.com/guide/topics/manifest/application-element#usesCleartextTraffic](https://developer.android.com/guide/topics/manifest/application-element#usesCleartextTraffic)

2. [https://imstudio.medium.com/android-8-cleartext-http-traffic-not-permitted-73c1c9e3b803](https://imstudio.medium.com/android-8-cleartext-http-traffic-not-permitted-73c1c9e3b803)