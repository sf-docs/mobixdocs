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

During its operation, an application often handles sensitive information such as passwords, various tokens, encryption keys, etc. During the analysis of the application Mobix detects such information according to the search rules and additionally checks if the found sensitive information is stored unchanged or is used by the application in other functions or is [“embedded“ in the source code of application](./storing_sensitive_information_in_the_application_source_code.md).

## Recommendations

If you need to use sensitive information in the application, make sure that it is stored correctly and does not escape to public places, such as system logs (logcat) or application files on the SD-card.

If it is necessary to store such information, it is recommended to use encryption. To ensure privacy, Android is equipped with many cryptographic features and methods that allow Android applications to securely encrypt and decrypt (to ensure privacy), as well as perform message authentication (MAC) and digital signatures (to verify integrity).

In order to select an encryption method and key type suitable for the given conditions, you can use the following scheme:

<figure markdown>
![](../../img/hranenie-ili-ispolzovanie-ranee-najdennoj-chuvstvitelnoj-informacii.png)
</figure>
**Encryption/decryption using Android KeyStore**

As an example, let's consider encryption/decryption using Android KeyStore. This mechanism allows you to generate and use keys generated in the Android hardware key store. This approach is the most secure in terms of key storage because the private key never appears in memory, which minimizes the risk of leakage or compromise.

**Creating new keys**

Before you start the encryption process, you must set the alias that will be used to encrypt/decrypt the data. This can be any string. Alias is the entry name to refer to the generated key in the Android KeyStore.

First, you need to get an instance of [Android KeyGenerator](https://developer.android.com/reference/javax/crypto/KeyGenerator).

    final KeyGenerator keyGenerator = KeyGenerator
            .getInstance(KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");

This example uses the AES algorithm and the keys will be stored in AndroidKeyStore.

Next you need to create a [KeyGenParameterSpec](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html), using [KeyGenParameterSpec.Builder](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder.html) to pass to the initialization method [KeyGenerators](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder.html).

**What is KeyGenParameterSpec?**

KeyGenParameterSpec is some properties of the keys that will be generated. For example, you can specify the validity period of a key, its purpose, and various other parameters.

    final KeyGenerator keyGenerator = KeyGenerator
            .getInstance(KeyProperties.KEY_ALGORITHM_AES, ANDROID_KEY_STORE);
    final KeyGenParameterSpec keyGenParameterSpec = new KeyGenParameterSpec.Builder(alias,
            KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
            .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .build();

In the example above, the first argument is the alias, that will be used to refer to this key. Then the purpose of this key is specified - to encrypt and decrypt data. The `setBlockModes` specifies a mode for using this key. Since we use the "**AES** / **GCM** / **NoPadding**" conversion algorithm, it is necessary to specify `BLOCK_MODE_GCM`. And the last parameter specifies the padding mode (`ENCRYPTION_PADDING_NONE`).

**Data encryption**

The presets are now complete. The following example can be used to encrypt data:

    final KeyGenerator keyGenerator = KeyGenerator
            .getInstance(KeyProperties.KEY_ALGORITHM_AES, ANDROID_KEY_STORE);
    final KeyGenParameterSpec keyGenParameterSpec = new KeyGenParameterSpec.Builder(alias,
            KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
            .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .build();
    keyGenerator.init(keyGenParameterSpec);
    final SecretKey secretKey = keyGenerator.generateKey();
    final Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);

First, the keyGenerator is initialized using [keyGenParameterSpec](http://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html). After that - the [SecretKey](https://developer.android.com/reference/javax/crypto/SecretKey.html) generation itself.

Now a secret key is available, you can use it to initialize the [Cipher](https://developer.android.com/reference/javax/crypto/Cipher.html) object, which is actually responsible for the encryption.

    final KeyGenerator keyGenerator = KeyGenerator
            .getInstance(KeyProperties.KEY_ALGORITHM_AES, ANDROID_KEY_STORE);
    final KeyGenParameterSpec keyGenParameterSpec = new KeyGenParameterSpec.Builder(alias,
            KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
            .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .build();
    keyGenerator.init(keyGenParameterSpec);
    final SecretKey secretKey = keyGenerator.generateKey();
    final Cipher cipher = Cipher.getInstance(TRANSFORMATION);
    cipher.init(Cipher.ENCRYPT_MODE, secretKey);
    iv = cipher.getIV();
    encryption = cipher.doFinal(textToEncrypt.getBytes("UTF-8"));

Then a reference to the initialization vector (IV) is used. This vector is also required for decryption. Using `doFinal (textToEncrypt)` we finish the encryption operation. The `doFinal` method returns an array of bytes, which is the encrypted text.

**Data decryption**

**Launching KeyStore**

Before we can start decrypting data, we need a `KeyStore` instance.

    keyStore = KeyStore.getInstance("AndroidKeyStore");
    keyStore.load(null);

`KeyStore` is used to get a private key using the `alias` that was previously used to encrypt the data.

You need `SecretKeyEntry` from the key store to get the `secretKey`.

    keyStore = KeyStore.getInstance("AndroidKeyStore");
    keyStore.load(null);
    final KeyStore.SecretKeyEntry secretKeyEntry = (KeyStore.SecretKeyEntry) keyStore
            .getEntry(alias, null);
    final SecretKey secretKey = secretKeyEntry.getSecretKey();

Use `GCMParameterSpec` with `Cipher` to initialize the decryption process (the `encryptionIv` parameter is the initialization vector that was used for encryption).

    keyStore = KeyStore.getInstance("AndroidKeyStore");
    keyStore.load(null);
    final KeyStore.SecretKeyEntry secretKeyEntry = (KeyStore.SecretKeyEntry) keyStore
            .getEntry(alias, null);
    final SecretKey secretKey = secretKeyEntry.getSecretKey();
    final Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    final GCMParameterSpec spec = new GCMParameterSpec(128, encryptionIv);
    cipher.init(Cipher.DECRYPT_MODE, secretKey, spec);

And, as we did before, to get the decrypted data:

    final byte[] decodedData = cipher.doFinal(encryptedData);

To get an unencrypted string representation:

    final String unencryptedString = new String(decodedData, "UTF-8");

[Complete example source code](https://gist.github.com/JosiasSena/3bf4ca59777f7dedcaf41a495d96d984).

## Links

1. [https://medium.com/@josiassena/using-the-android-keystore-system-to-store-sensitive-information-3a56175a454bhttps://mobile-security.gitbook.io/mobile-security-testing-guide/android-testing-guide/0x05d-testing-data-storage](https://medium.com/@josiassena/using-the-android-keystore-system-to-store-sensitive-information-3a56175a454bhttps://mobile-security.gitbook.io/mobile-security-testing-guide/android-testing-guide/0x05d-testing-data-storage)

2. [https://gist.github.com/JosiasSena/3bf4ca59777f7dedcaf41a495d96d984https://cwe.mitre.org/data/definitions/200.html](https://gist.github.com/JosiasSena/3bf4ca59777f7dedcaf41a495d96d984https://cwe.mitre.org/data/definitions/200.html)

3. [https://developer.android.com/reference/javax/crypto/KeyGenerator.html](https://developer.android.com/reference/javax/crypto/KeyGenerator.html)

4. [https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html)

5. [https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder.html](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.Builder.html)

6. [https://developer.android.com/reference/android/security/keystore/KeyProperties.html](https://developer.android.com/reference/android/security/keystore/KeyProperties.html)

7. [https://developer.android.com/reference/javax/crypto/SecretKey.html](https://developer.android.com/reference/javax/crypto/SecretKey.html)

8. [https://developer.android.com/reference/javax/crypto/Cipher.html](https://developer.android.com/reference/javax/crypto/Cipher.html)

9. [https://developer.android.com/reference/javax/crypto/spec/GCMParameterSpec.html](https://developer.android.com/reference/javax/crypto/spec/GCMParameterSpec.html)