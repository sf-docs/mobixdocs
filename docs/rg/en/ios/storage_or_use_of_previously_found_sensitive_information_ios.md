# Storage or use of previously found sensitive information

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
        <td>Detection method:<strong> DAST, API</strong></td>
      </tr>
    </tbody>
</table>
## Description

An application stores or uses sensitive information in its operation.

During its operation, an application often handles sensitive information such as passwords, various tokens, encryption keys, etc. During the analysis of the application Mobix detects such information according to the search rules and additionally checks if the found sensitive information is stored unchanged or is used by the application in other functions or is [“embedded“ in the source code of application](./storing_sensitive_information_in_the_application_source_code_ios.md).

## Recommendations

If you need to use sensitive information in the application, make sure that it is stored correctly and does not escape to public places, such as system logs.

If it is necessary to store such information, it is recommended to use encryption. To ensure privacy, iOS is equipped with many cryptographic features and methods that allow iOS applications to securely encrypt and decrypt (to ensure privacy), as well as perform message authentication (MAC) and digital signatures (to verify integrity).

To choose an encryption method and key type suitable for the given conditions, you can use the following scheme:

<figure markdown>
![](../../img/image8.png)
</figure>
### Encryption/decryption using Keychain

As an example, let's consider encryption/decryption using Keychain. This mechanism allows you to generate and use keys generated in the iOS hardware key storage. This approach is the most secure in terms of key storage because the private key never appears in memory, which minimizes the risk of leakage or compromise.