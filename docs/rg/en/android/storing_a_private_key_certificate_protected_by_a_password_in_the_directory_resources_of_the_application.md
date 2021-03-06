# Storing a private key/certificate protected by a password in the directory/resources of the application

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
        <td>Detection method:<strong> DAST, SENSITIVE INFO</strong></td>
      </tr>
    </tbody>
</table>
## Description

Storing a password-protected private key anywhere on the device's file system can be a serious problem if the password is not strong enough or if it is compromised. Private keys are used to encrypt or decrypt data (depending on the type of algorithm, symmetric or asymmetric) and must not be available to anyone.

A more serious vulnerability is the use of a single key supplied with the application (in the resources or obtained from the server) to the data of different users. By retrieving the value of such a key from one instance of the application, you can decrypt data from another user.

## Recommendations

Although storing a private key with a password is not an immediate vulnerability, there are risks associated with its use. The following are recommendations for creating and storing keys for encryption operations. The recommended way to store keys is to use KeyChain.

KeyStore provides several basic features that greatly simplify the work with cryptographic keys:

* Random key generation.

* Secure key storage.

All you need to do is:

* Generate a random key when you run the application for the first time.

* If you want to encrypt data, get the key from KeyStore, encrypt the data with it, and save the encrypted data.

* If you want to decrypt data, get the key from KeyStore and use it to decrypt the data.

> The way you use Android KeyStore differs depending on the Android version (higher or lower than Android 6).

### Android 6 and higher

For API level 23 and higher, the implementation will be easier because the AES keys are generated by the system. An example can be found in the [API KeyGenParameterSpec](https://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html).

**Key generation:**

    private static final String AndroidKeyStore = "AndroidKeyStore";
    private static final String AES_MODE = "AES/GCM/NoPadding";
    keyStore = KeyStore.getInstance(AndroidKeyStore);
    keyStore.load(null);
    if (!keyStore.containsAlias(KEY_ALIAS)) {
        KeyGenerator keyGenerator = KeyGenerator.getInstance(KeyProperties.KEY_ALGORITHM_AES, AndroidKeyStore);
        keyGenerator.init(
                new KeyGenParameterSpec.Builder(KEY_ALIAS,
                        KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
                        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
                        .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
                        .setRandomizedEncryptionRequired(false) 
                        .build());
        keyGenerator.generateKey();
    }

**Receiving the key:**

    private java.security.Key getSecretKey(Context context) throws Exception {
    return keyStore.getKey(XEALTH_KEY_ALIAS, null);
    }

**Data encryption:**

    Cipher c = Cipher.getInstance(AES_MODE);
    c.init(Cipher.ENCRYPT_MODE, getSecretKey(context), new GCMParameterSpec(128, FIXED_IV.getBytes()));
    byte[] encodedBytes = c.doFinal(input);
    String encryptedBase64Encoded = Base64.encodeToString(encodedBytes, Base64.DEFAULT);
    return encryptedBase64Encoded;

**Data decryption:**

    Cipher c = Cipher.getInstance(AES_MODE);
    c.init(Cipher.DECRYPT_MODE, getSecretKey(context), new GCMParameterSpec(128, FIXED_IV.getBytes()));
    byte[] decodedBytes = c.doFinal(encrypted);
    return decodedBytes;

**Initialization vector**

The initialization vector (IV) is a cryptographic function that is responsible for the randomness of the first encryption block. Remember that the IV used for encryption must be the same as the one used for decryption. By default Android forces you to use a random IV every time, but you can disable this by calling `setRandomizedEncryptionRequired()` when generating the key.

Due to the security provided by Android KeyStore, a random IV is redundant, so a fixed IV can be used instead. If there is a need to use random IVs, you can call the `getIV()` method when encrypting the data and use the same IV when decrypting it.

### Lower than Android 6

For Android API versions lower than 23 (Android 6), a little more work is required. [KeyGenParameterSpec](http://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html) is only available in API 23, so the KeyStore itself cannot generate random AES keys. Instead, you need to generate the keys yourself using the [KeyGenParameterSpec](http://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html).

As the name suggests, KeyPairGeneratorSpec generates public and private key pairs. Public key encryption is primarily for signature and authentication and is not suitable for the encryption of large blocks of data, but can be combined with a block cipher such as AES.

That is the principle of KEK + DEK (Key Encryption Key + Data Encryption Key). We encrypt the data with a key, which in turn is encrypted with another key stored in the KeyStore. This approach has its advantages. For example, if you change the encryption key, it is enough to re-encrypt the AES key and not touch user data (not re-encrypt it). The algorithm is roughly as follows:

**Key generation:**

* Generate a pair of RSA keys.
* Generate a random AES key.
* Encrypt the AES key using the RSA public key.
* Save the encrypted key in Shared Preferences.

**Data encryption and storage:**

* Get the encrypted AES key from Shared Preferences.
* Decrypt the key using the RSA private key.
* Encrypt data using the AES key.

**Data receiving and decryption:**

* Get the encrypted AES key from Shared Preferences.
* Decrypt the key using the RSA private key.
* Decrypt data using the AES key.

**RSA key generation**

    private static final String     AndroidKeyStore = "AndroidKeyStore";
    keyStore = KeyStore.getInstance(AndroidKeyStore);
    keyStore.load(null);
    // Generate the RSA key pairs
    if (!keyStore.containsAlias(KEY_ALIAS)) {
        // Generate a key pair for encryption
        Calendar start = Calendar.getInstance();
        Calendar end = Calendar.getInstance();
        end.add(Calendar.YEAR, 30);
        KeyPairGeneratorSpec spec = new      KeyPairGeneratorSpec.Builder(context)
                .setAlias(KEY_ALIAS)
                .setSubject(new X500Principal("CN=" + KEY_ALIAS))
                .setSerialNumber(BigInteger.TEN)
                .setStartDate(start.getTime())
                .setEndDate(end.getTime())
                .build();
        KeyPairGenerator kpg = KeyPairGenerator.getInstance(KeyProperties.KEY_ALGORITHM_RSA, AndroidKeyStore);
        kpg.initialize(spec);
        kpg.generateKeyPair();
    }

**RSA encryption and decryption procedures:**

    private static final String RSA_MODE =  "RSA/ECB/PKCS1Padding";
    private byte[] rsaEncrypt(byte[] secret) throws Exception{
        KeyStore.PrivateKeyEntry privateKeyEntry = (KeyStore.PrivateKeyEntry) keyStore.getEntry(KEY_ALIAS, null);
        // Encrypt the text
        Cipher inputCipher = Cipher.getInstance(RSA_MODE, "AndroidOpenSSL");
        inputCipher.init(Cipher.ENCRYPT_MODE, privateKeyEntry.getCertificate().getPublicKey());
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        CipherOutputStream cipherOutputStream = new CipherOutputStream(outputStream, inputCipher);
        cipherOutputStream.write(secret);
        cipherOutputStream.close();
        byte[] vals = outputStream.toByteArray();
        return vals;
    }
    private  byte[]  rsaDecrypt(byte[] encrypted) throws Exception {
        KeyStore.PrivateKeyEntry privateKeyEntry = (KeyStore.PrivateKeyEntry)keyStore.getEntry(KEY_ALIAS, null);
        Cipher output = Cipher.getInstance(RSA_MODE, "AndroidOpenSSL");
        output.init(Cipher.DECRYPT_MODE, privateKeyEntry.getPrivateKey());
        CipherInputStream cipherInputStream = new CipherInputStream(
                new ByteArrayInputStream(encrypted), output);
        ArrayList values = new ArrayList<>();
        int nextByte;
        while ((nextByte = cipherInputStream.read()) != -1) {
            values.add((byte)nextByte);
        }
        byte[] bytes = new byte[values.size()];
        for(int i = 0; i < bytes.length; i++) {
            bytes[i] = values.get(i).byteValue();
        }
        return bytes;
    }

**AES key generation and storage:**

    SharedPreferences pref = context.getSharedPreferences(SHARED_PREFENCE_NAME, Context.MODE_PRIVATE);
    String enryptedKeyB64 = pref.getString(ENCRYPTED_KEY, null);
    if (enryptedKeyB64 == null) {
    byte[] key = new byte[16];
    SecureRandom secureRandom = new SecureRandom();
    secureRandom.nextBytes(key);
    byte[] encryptedKey = rsaEncrypt(key);
    enryptedKeyB64 = Base64.encodeToString(encryptedKey, Base64.DEFAULT);
    SharedPreferences.Editor edit = pref.edit();
    edit.putString(ENCRYPTED_KEY, enryptedKeyB64);
    edit.commit();
    }

**Data encryption and decryption:**

    private static final String AES_MODE = "AES/ECB/PKCS7Padding";
    private Key getSecretKey(Context context) throws Exception{
        SharedPreferences pref = context.getSharedPreferences(SHARED_PREFENCE_NAME, Context.MODE_PRIVATE);
        String enryptedKeyB64 = pref.getString(ENCRYPTED_KEY, null);          
        // need to check null, omitted here
        byte[] encryptedKey = Base64.decode(enryptedKeyB64, Base64.DEFAULT);
        byte[] key = rsaDecrypt(encryptedKey);
        return new SecretKeySpec(key, "AES");
    }
    public String encrypt(Context context, byte[] input) {
        Cipher c = Cipher.getInstance(AES_MODE, "BC");
        c.init(Cipher.ENCRYPT_MODE, getSecretKey(context));
        byte[] encodedBytes = c.doFinal(input);
        String encryptedBase64Encoded =  Base64.encodeToString(encodedBytes, Base64.DEFAULT);
        return encryptedBase64Encoded;
    }
    public byte[] decrypt(Context context, byte[] encrypted) {
        Cipher c = Cipher.getInstance(AES_MODE, "BC");
        c.init(Cipher.DECRYPT_MODE, getSecretKey(context));
        byte[] decodedBytes = c.doFinal(encrypted);
        return decodedBytes;
    }

## Links

1. [https://doridori.github.io/android-security-the-forgetful-keystore/#sthash.cxj8r3G6.dpbs](https://doridori.github.io/android-security-the-forgetful-keystore/#sthash.cxj8r3G6.dpbs)

2. [http://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html](http://developer.android.com/reference/android/security/keystore/KeyGenParameterSpec.html)

3. [http://developer.android.com/reference/android/security/KeyPairGeneratorSpec.html](http://developer.android.com/reference/android/security/KeyPairGeneratorSpec.html)

4. [https://medium.com/@ericfu/securely-storing-secrets-in-an-android-application-501f030ae5a3#:~:text=With%20these%2C%20storing%20secrets%20becomes,the%20encrypted%20data%20in%20Preferences.](https://medium.com/@ericfu/securely-storing-secrets-in-an-android-application-501f030ae5a3#:~:text=With%20these%2C%20storing%20secrets%20becomes,the%20encrypted%20data%20in%20Preferences.)

5. [https://www.owasp.org/index.php/Mobile\_Top\_10\_2016-M2-Insecure\_Data\_Storage](https://www.owasp.org/index.php/Mobile_Top_10_2016-M2-Insecure_Data_Storage)