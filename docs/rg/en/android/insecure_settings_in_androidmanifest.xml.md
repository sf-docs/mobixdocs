# Insecure settings in AndroidManifest.xml

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
        <td>Detection method:<strong> SAST, APK</strong></td>
      </tr>
    </tbody>
</table>
## Description

An Android application that has been built with an enabled [debug mode](https://developer.android.com/guide/topics/manifest/application-element#debug) (flag `android:debuggable = True` in `AndroidManifest.xml`) can provide a malicious person with access to confidential information, with possibility to control the application's run and to execute code in the application's context.

**Example of vulnerable code (AndroidManifest.xml)**

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.appsec.android.activity.privateactivity" >
            <application
                    <!-- *** Включенние отладочного режима *** -->
                android:debuggable="true"
                android:icon="@drawable/ic_launcher"
                android:label="@string/app_name" >
                <activity
                        android:name=".PrivateActivity"
                        android:label="@string/app_name"
                        android:exported="false" />
            </application>
    </manifest>

## Recommendations

While building a release version of an application, ensure that the debug mode is disabled. You can disable the debug mode by deleting the `android:debuggable` attribute from an **\<application>** tag in the manifest file, or by setting the `android:debuggable` value for the `false` attribute in the manifest.

**Example of secure code**

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.appsec.android.activity.privateactivity" >
    <application
    <!-- *** Выключенние отладочного режима *** -->
    android:debuggable="false"
    android:icon="@drawable/ic_launcher"
    android:label="@string/app_name" >
    <activity
    android:name=".PrivateActivity"
    android:label="@string/app_name"
    android:exported="false" />
    </application>
    </manifest>

It is also possible to set up the debug mode for different builds using specific configurations in the `build.gradle` file:

    android {
        defaultConfig {
            ...
            ...
        }
        buildTypes {
            release {
                // *** Отключение отладочного режима для релизной сборки приложения *** //
                debuggable false
                minifyEnabled true
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            }
            debug {
                applicationIdSuffix ".debug"
                // *** Включение отладочного режима для целей разработки *** //
                debuggable true
            }

## Links

1. [https://developer.android.com/studio/build/build-variants#build-types](https://developer.android.com/studio/build/build-variants#build-types)

2. [https://developer.android.com/studio/publish/preparing#publishing-configure](https://developer.android.com/studio/publish/preparing#publishing-configure)

3. [https://resources.infosecinstitute.com/android-hacking-security-part-6-exploiting-debuggable-android-applications/](https://resources.infosecinstitute.com/android-hacking-security-part-6-exploiting-debuggable-android-applications/)

4. [https://securitygrind.com/how-to-exploit-a-debuggable-android-application/](https://securitygrind.com/how-to-exploit-a-debuggable-android-application/)

5. [https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md)