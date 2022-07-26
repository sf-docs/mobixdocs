# Insecure settings in AndroidManifest.xml. The android:requestLegacyExternalStorage flag

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_nizkij.png"/></td>
        <td>Severity:<strong> LOW</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> SAST, APK</strong></td>
      </tr>
    </tbody>
</table>
## Description

The Android application built with the `android:requestLegacyExternalStorage=true` attribute in ***AndroidManifest.xml*** provides access to directories and various types of media files stored in external storage. This flag is used in the old file access model, which is not supported in new versions of Android.

1. If the `android:requestLegacyExternalStorage=true` attribute is present in AndroidManifest and `targetSDK >=30`, this attribute is ignored by the system because, starting from Android 11, only [scoped storage](https://developer.android.com/about/versions/11/privacy/storage) is supported.
2. If `targetSDK = 29`, then the default attribute value is **false** (if not specified in the manifest).
3. If `targetSDK >= 28`, then the default attribute value is **true** (if not specified in the manifest).

**An example of vulnerable configuration (AndroidManifest.xml file):**

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.appsec.android.activity.privateactivity" >
    
        <application
            android:icon="@drawable/ic_launcher"
            android:requestLegacyExternalStorage="true"
            android:label="@string/app_name" >
            
            <activity
                android:name=".PrivateActivity"
                android:label="@string/app_name"
                android:exported="false" />
                
        </application>
    </manifest>

## Recommendations

It is recommended not to set the `android:requestLegacyExternalStorage` attribute and only use scoped storage to guarantee better protection of applications and user data on external storage.

## Links

1. [https://developer.android.com/training/data-storage/use-cases](https://developer.android.com/training/data-storage/use-cases)

2. [https://commonsware.com/blog/2019/06/07/death-external-storage-end-saga.html](https://commonsware.com/blog/2019/06/07/death-external-storage-end-saga.html)

3. [https://medium.com/mindful-engineering/scoped-storage-in-android-d52460630d6a](https://medium.com/mindful-engineering/scoped-storage-in-android-d52460630d6a)

4. [https://blog.mindorks.com/understanding-the-scoped-storage-in-android](https://blog.mindorks.com/understanding-the-scoped-storage-in-android)