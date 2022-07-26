# Insecure Signature Algorithm

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

Algorithm used to sign an APK file is considered insecure.

Signature algorithms (**MD2withRSA**, **MD5withRSA**, **SHA1withRSA**, **SHA1withDSA**, **SHA1withECDSA**) based on hash functions **MD2**, **MD5**, **SHA1** and other outdated encryption algorithms contain known vulnerabilities and are prone to collisions. The main risk in using of a weak key is that a malicious person could break it to forge APK signatures. Thereafter, the malicious APK signed with your key could be installed as an update for your application. Depending on how a key is used in an application, there are other possible attacks involving a compromised key.

Also, some application stores (for example, by [Huawei](https://developer.huawei.com/consumer/en/appgallery)) don't recommend signing an application using such algorithms. Moreover, they don't allow uploading such applications.

## Recommendations

To repair this vulnerability, you need to sign an application using up-to-date algorithms, such as SHA256withRSA or SHA512withRSA. Note, that earlier Android versions may not support algorithms beyond SHA1. This [article](https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/) highlights the above problem and provides guidance through the process of signature changing.

**Example: Generation of key using SHA512withRSA**

    keytool -genkey -v -keystore test.keystore -alias testkey -keyalg RSA -keysize 4096 -sigalg SHA512withRSA -dname "cn=Test,ou=Test,c=CA" -validity 10000

**Example: Signature with a generated key**

    jarsigner -verbose -sigalg SHA512withRSA -digestalg SHA512 -keystore test.keystore test.apk testkey

## Links

1. [https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/](https://guardianproject.info/2015/12/29/how-to-migrate-your-android-apps-signing-key/)

2. [https://developer.android.com/studio/publish/app-signing](https://developer.android.com/studio/publish/app-signing)

3. [https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md)

4. [https://sites.google.com/site/itstheshappening/](https://sites.google.com/site/itstheshappening/)