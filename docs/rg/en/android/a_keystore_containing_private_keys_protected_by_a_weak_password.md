# A keystore containing private keys protected by a weak password

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
        <td>Detection method:<strong> DAST, KEY INFORMATION</strong></td>
      </tr>
    </tbody>
</table>
## Description

An application uses a key storage that contains private keys protected by a weak password. A keystore and the keys stored in it must be protected by a strong password.

When using cryptographic operations on a device, you need to ensure maximum security of the main secret in such operations—the encryption key. When using asymmetric encryption, you need to store the private key securely; for symmetric algorithms, though, you need to protect the key that is used both for encryption and decryption of sensitive information. There are several ways to store the keys depending on a version of the operating system: in the AndroidKeyStore storage or in the application's directory in BKS. The most secure option, without a doubt, is to store keys in AndroidKeyStore. If there is a need to store the keys in BKS, you should ensure that the storage itself as well as all keys are protected by a strong password.

Compromising key information in an application could be catastrophic depending on where in the application this information is used — starting with decryption of files or traffic, to compromising a private key that functions as a signature of the application.

## Recommendations

Encryption keys must not be stored in a location with public access, even if this is the application's directory on an SD-card.

There are several ways to store the keys depending on a version of the operating system:

* On Android API\<18 the encryption keys must be stored inside the application's directory in BKS.
* On Android API>=18 RSA keys must be stored in AndroidKeyStore, and AES keys—in BKS.
* On Android API>=23 RSA and AES keys must be stored in AndroidKeyStore.

Also, don't forget that when using the BKS in the internal application's directory, an additional strong password is required to protect the keystore and the keys within it. A good idea would be to check the generated password against a database for most common passwords and ensure that the password meets minimum requirements:

* The password is at least 20 characters in length.
* At least one lower-case letter is required.
* At least one upper-case letter is required.
* At least one numeric character is required.
* At least one special character is required.

**Generation of a password-protected BKS storage containing a key protected with an additional password:**

    keytool -importcert -v -trustcacerts -file "C:\Users\Indra\Documents\myapp.com.cer"
    -alias IntermediateCA -keystore "C:\Users\Indra\Documents\appKeyStore.bks"
    -provider org.bouncycastle.jce.provider.BouncyCastleProvider
    -providerpath "C:\Users\Indra\Downloads\bcprov-jdk15on-154.jar"
    -storetype BKS -storepass StorePass123
    keytool -list -keystore "C:\Users\Indra\Documents\appKeyStore.bks"
    -provider org.bouncycastle.jce.provider.BouncyCastleProvider
    -providerpath "C:\Users\Indra\Downloads\bcprov-jdk15on-154.jar"
    -storetype BKS -storepass "StorePass123"
    ---------------------------------------------------------------------------------------------------
    openssl pkcs12 -export -in "/home/myapp/myapp_cert_2016/ssl_certificate.crt"
    -inkey "/home/myapp/myapp_cert_2016/domainname.key"
    -certfile "/home/myapp/myapp_cert_2016/ssl_certificate.crt" -out testkeystore.p12
    Export password : exportpass123
    keytool -importkeystore -srckeystore "C:\Users\Indra\myapp\testkeystore.p12"
    -srcstoretype pkcs12 -destkeystore ""C:\Users\Indra\myapp\wso2carbon.jks"
    -deststoretype JKS
    Destination keystore password : exportpass123
    ----------------------------------------------------- Final JKS Keystore generation ---------------
    # openssl pkcs12 -export -in "/home/myapp/myapp_cert_2016/ssl_certificate.crt"
    -inkey "/home/myapp/myapp_cert_2016/domainname.key"
    -certfile "/home/myapp/myapp_cert_2016/ssl_certificate.crt" -out myapp_cert.p12
    Export Password : StorePass123
    keytool -importkeystore -srckeystore "C:\Users\Indra\myapp\myapp_cert.p12"
    -srcstoretype pkcs12 -destkeystore "C:\Users\Indra\myapp\myapp_keystore.jks"
    -deststoretype JKS
    Import Password : StorePass123
    ----------------------------------------------------- Final BKS Keystore generation ---------------
    keytool -importkeystore -srckeystore "C:\Users\Indra\myapp\myapp_keystore.jks
    -deststoretype JKS" -destkeystore "C:\Users\Indra\myapp\myapp_keystore.bks"
    -srcstoretype JKS -deststoretype BKS -srcstorepass StorePass123
    -deststorepass StorePass123 -provider org.bouncycastle.jce.provider.BouncyCastleProvider
    -providerpath "C:\Users\Indra\Downloads\bcprov-jdk15on-154.jar"
    On error or exception steps to be taken
    - Comment above line and add the new line in java.security file in jre/lib/security
        #security.provider.7=com.sun.security.sasl.Provider
        security.provider.7=org.bouncycastle.jce.provider.BouncyCastleProvider
    - You need to install the Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy

To interact with AndroidKeyStore you can use an auxiliary library or create your own implementation. See below an example of essential code fragments needed to encrypt and store the key in AndroidKeyStore.

**Adding a new key into KeyStore**

    public void createNewKeys(View view) {
        String alias = aliasText.getText().toString();
        try {
            // Create new key if needed
            if (!keyStore.containsAlias(alias)) {
                Calendar start = Calendar.getInstance();
                Calendar end = Calendar.getInstance();
                end.add(Calendar.YEAR, 1);
                KeyPairGeneratorSpec spec = new KeyPairGeneratorSpec.Builder(this)
                        .setAlias(alias)
                        .setSubject(new X500Principal("CN=Sample Name, O=Android Authority"))
                        .setSerialNumber(BigInteger.ONE)
                        .setStartDate(start.getTime())
                        .setEndDate(end.getTime())
                        .build();
                KeyPairGenerator generator = KeyPairGenerator.getInstance("RSA", "AndroidKeyStore");
                generator.initialize(spec);
                KeyPair keyPair = generator.generateKeyPair();
            }
        } catch (Exception e) {
            Toast.makeText(this, "Exception " + e.getMessage() + " occured", Toast.LENGTH_LONG).show();
            Log.e(TAG, Log.getStackTraceString(e));
        }
        refreshKeys();
    }

**Deleting a key from KeyStore**

    public void deleteKey(final String alias) {
        AlertDialog alertDialog =new AlertDialog.Builder(this)
                .setTitle("Delete Key")
                .setMessage("Do you want to delete the key \"" + alias + "\" from the keystore?")
                .setPositiveButton("Yes", new DialogInterface.OnClickListener() {
                    public void onClick(DialogInterface dialog, int which) {
                        try {
                            keyStore.deleteEntry(alias);
                            refreshKeys();
                        } catch (KeyStoreException e) {
                            Toast.makeText(MainActivity.this,
                                    "Exception " + e.getMessage() + " occured",
                                    Toast.LENGTH_LONG).show();
                            Log.e(TAG, Log.getStackTraceString(e));
                        }
                        dialog.dismiss();
                    }
                })
                .setNegativeButton("No", new DialogInterface.OnClickListener() {
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                })
                .create();
        alertDialog.show();
    }

**Using a key for encryption**

    public void encryptString(String alias) {
        try {
            KeyStore.PrivateKeyEntry privateKeyEntry = (KeyStore.PrivateKeyEntry)keyStore.getEntry(alias, null);
            RSAPublicKey publicKey = (RSAPublicKey) privateKeyEntry.getCertificate().getPublicKey();
            // Encrypt the text
            String initialText = startText.getText().toString();
            if(initialText.isEmpty()) {
                Toast.makeText(this, "Enter text in the 'Initial Text' widget", Toast.LENGTH_LONG).show();
                return;
            }
            Cipher input = Cipher.getInstance("RSA/CBC/PKCS7Padding", "AndroidOpenSSL");
            input.init(Cipher.ENCRYPT_MODE, publicKey);
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            CipherOutputStream cipherOutputStream = new CipherOutputStream(
                    outputStream, input);
            cipherOutputStream.write(initialText.getBytes("UTF-8"));
            cipherOutputStream.close();
            byte [] vals = outputStream.toByteArray();
            encryptedText.setText(Base64.encodeToString(vals, Base64.DEFAULT));
        } catch (Exception e) {
            Toast.makeText(this, "Exception " + e.getMessage() + " occured", Toast.LENGTH_LONG).show();
            Log.e(TAG, Log.getStackTraceString(e));
        }
    }

**Using a key for decryption**

    public void decryptString(String alias) {
        try {
            KeyStore.PrivateKeyEntry privateKeyEntry = (KeyStore.PrivateKeyEntry)keyStore.getEntry(alias, null);
            RSAPrivateKey privateKey = (RSAPrivateKey) privateKeyEntry.getPrivateKey();
            Cipher output = Cipher.getInstance("RSA/CBC/PKCS7Padding", "AndroidOpenSSL");
            output.init(Cipher.DECRYPT_MODE, privateKey);
            String cipherText = encryptedText.getText().toString();
            CipherInputStream cipherInputStream = new CipherInputStream(
                    new ByteArrayInputStream(Base64.decode(cipherText, Base64.DEFAULT)), output);
            ArrayList values = new ArrayList<>();
            int nextByte;
            while ((nextByte = cipherInputStream.read()) != -1) {
                values.add((byte)nextByte);
            }
            byte[] bytes = new byte[values.size()];
            for(int i = 0; i < bytes.length; i++) {
                bytes[i] = values.get(i).byteValue();
            }
            String finalText = new String(bytes, 0, bytes.length, "UTF-8");
            decryptedText.setText(finalText);
        } catch (Exception e) {
            Toast.makeText(this, "Exception " + e.getMessage() + " occured", Toast.LENGTH_LONG).show();
            Log.e(TAG, Log.getStackTraceString(e));
        }
    }

## Links

1. [https://developer.android.com/training/articles/keystore#kotlin](https://developer.android.com/training/articles/keystore#kotlin)

2. [https://www.androidauthority.com/use-android-keystore-store-passwords-sensitive-information-623779/](https://www.androidauthority.com/use-android-keystore-store-passwords-sensitive-information-623779/)

3. [https://developer.android.com/guide/topics/security/cryptography](https://developer.android.com/guide/topics/security/cryptography)

4. [https://developer.android.com/reference/androidx/security/crypto/EncryptedFile](https://developer.android.com/reference/androidx/security/crypto/EncryptedFile)

5. [https://security.stackexchange.com/questions/128003/does-the-use-of-a-smartphones-secure-element-really-offer-security-benefits-to](https://security.stackexchange.com/questions/128003/does-the-use-of-a-smartphones-secure-element-really-offer-security-benefits-to)

6. [https://github.com/Q42/Qlassified-Android](https://github.com/Q42/Qlassified-Android)