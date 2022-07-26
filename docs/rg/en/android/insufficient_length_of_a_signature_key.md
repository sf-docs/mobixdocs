# Insufficient length of a signature key

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

The length of the key used for signing an APK file is insufficient.

[Official NIST recommendations](https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/) (PDF, pages 64 and 67) qualify 1024-bit RSA keys as insecure (starting from 2013). This doesn't mean that 1024-bit RSA has been compromised, this is more of a preventive measure in order to be a step ahead of attackers. The main risk in using of a weak key is that a malicious person could break it to forge APK signatures. Thereafter, the malicious APK signed with your key could be installed as an update for your application. Depending on how a key is used in an application, there are other possible attacks involving a compromised key.

Also, some app stores, such as [Huawei](https://developer.huawei.com/consumer/en/appgallery), do not recommend signing applications with a key length shorter than 2048 bits. Moreover, they don't allow uploading such applications.

## Recommendations

To remediate this vulnerability, you need to sign the application using modern algorithms, such as **SHA256withRSA** or **SHA512withRSA**, using a key with a minimum length of 2048 bits (the recommended length is 4096 bits). Note that early versions of Android may not support algorithms higher than SHA1. This [article](https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/) highlights the above problem and provides guidance through the process of signature changing.

**Example: Generation of key using SHA512withRSA**

    keytool -genkey -v -keystore test.keystore -alias testkey -keyalg RSA -keysize 4096 -sigalg SHA512withRSA -dname "cn=Test,ou=Test,c=CA" -validity 10000

**Example: Signature with a generated key**

    jarsigner -verbose -sigalg SHA512withRSA -digestalg SHA512 -keystore test.keystore test.apk testkey

## Links

1. [https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/](https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/)

2. [https://developer.android.com/studio/publish/app-signing](https://developer.android.com/studio/publish/app-signing)

3. [https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md)

4. [https://sites.google.com/site/itstheshappening/](https://sites.google.com/site/itstheshappening/)

5. [https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/](https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/)