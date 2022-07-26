# Possibility to create a backup copy of the application

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
        <td>Detection method:<strong> DAST, APK</strong></td>
      </tr>
    </tbody>
</table>
## Description

An Android application built with the [backup option enabled](https://developer.android.com/guide/topics/manifest/application-element#allowbackup) (the `android:allowBackup = True` flag in ***AndroidManifest.xml***) can allow an attacker with physical access to the device to create an application backup with all the data stored in the internal directory of the application. In addition to the access to the information, it is possible to change the information contained in the files and restore the settings from the modified backup. This type of exploitation, in some cases, can lead to the compromise of user data.

Please note that this option is enabled by default.

**An example of vulnerable code (AndroidManifest.xml file):**

    <?xml version=»1.0″ encoding=»utf-8″?>
    <manifest xmlns:android=»http://schemas.android.com/apk/res/android»
    package=»com.appsec.android.activity.privateactivity» >
    <application
    <!— *** Включенние возможности создания резервной копии *** —>
    android:allowBackup=»true»
    android:icon=»@drawable/ic_launcher»
    android:label=»@string/app_name» >
    <activity
    android:name=».PrivateActivity»
    android:label=»@string/app_name»
    android:exported=»false» />
    </application>
    </manifest>

**An example of a command that creates a backup of an application:**

    adb backup -f com.appsec.android

Next, using scripts or the [android-backup-extractor](https://github.com/nelenkov/android-backup-extractor) utility, you need to convert from the Android Backup format to a regular archive and access the data. Or, you can use an alternative command:

    dd if=mybackup.ab bs=24 skip=1| openssl zlib -d > mybackup.tar
    # Или воспользоваться abe
    abe unpack mybackup.ab mybackup.tar

If necessary, you can change the contents of the files, and use the same [android-backup-extractor](https://github.com/nelenkov/android-backup-extractor) utility to create an Android Backup archive and restore it to the device:

    abe pack mybackup.tar mybackup.adb restore mybackup.ab

## Recommendations

When building the release version of the application, make sure that the backup option is disabled. You can disable this option by setting the `android:allowBackup` attribute to `false` in the manifest file.

**An example of a secure code:**

    <?xml version=»1.0″ encoding=»utf-8″?>
    <manifest xmlns:android=»http://schemas.android.com/apk/res/android»
    package=»com.appsec.android.activity.privateactivity» >
    <application
    <!— *** Выключенние возможности создания резервной копии *** —>
    android:allowBackup=»false»
    android:icon=»@drawable/ic_launcher»
    android:label=»@string/app_name» >
    <activity
    android:name=».PrivateActivity»
    android:label=»@string/app_name»
    android:exported=»false» />
    </application>
    </manifest>

## Links

1. [https://developer.android.com/guide/topics/manifest/application-element#allowbackup](https://developer.android.com/guide/topics/manifest/application-element#allowbackup)

2. [https://github.com/nelenkov/android-backup-extractor](https://github.com/nelenkov/android-backup-extractor)

3. [https://securitygrind.com/exploiting-android-backup/](https://securitygrind.com/exploiting-android-backup/)

4. [https://github.com/OWASP/owasp-stg/blob/master/Document/0x05d-Testing-Data-Storage.md#local](https://github.com/OWASP/owasp-stg/blob/master/Document/0x05d-Testing-Data-Storage.md#local)

5. [https://resources.infosecinstitute.com/topic/android-hacking-security-part-15-hacking-android-apps-using-backup-techniques/](https://resources.infosecinstitute.com/topic/android-hacking-security-part-15-hacking-android-apps-using-backup-techniques/)