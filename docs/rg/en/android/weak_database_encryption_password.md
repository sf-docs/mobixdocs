# Weak database encryption password

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

The password used to encrypt the database does not meet the criteria of length, simplicity or frequency of use. The main parameter for the security of data storage in an encrypted database on a device is the password that is used for encryption. If the chosen password is too simple or is present in the database of the most common passwords, there is a high probability of finding a password and compromising the information.

## Recommendations

When using a password to encrypt the database, there exists the question of how to store it securely. Upon successful authentication in the application, there are several options for the server part to send a secret that forms the password used to open or encrypt the database.

Another option is to generate an encryption key (that can be used as a database password) based on the user password. In this case, there is no need to store the encryption key, as it is generated "on the fly" based on the password entered by the user:

Rules:

1. Explicitly define the encryption mode and block padding.

2. Use crypto-resistant encryption technologies including algorithm, block encryption mode, and block padding mode.

3. Use "salt" when generating a password-based key.

4. Use a sufficient number of hashing iterations while generating the key based on the password.

5. Use a key with a length that ensures cryptographic strength of the encryption.
   
        package com.appsec.android.cryptsymmetricpasswordbasedkey;
       
        import java.security.InvalidAlgorithmParameterException;
        import java.security.InvalidKeyException;
        import java.security.NoSuchAlgorithmException;
        import java.security.SecureRandom;
        import java.security.spec.InvalidKeySpecException;
        import java.util.Arrays;
       
        import javax.crypto.BadPaddingException;
        import javax.crypto.Cipher;
        import javax.crypto.IllegalBlockSizeException;
        import javax.crypto.NoSuchPaddingException;
        import javax.crypto.SecretKey;
        import javax.crypto.SecretKeyFactory;
        import javax.crypto.spec.IvParameterSpec;
        import javax.crypto.spec.PBEKeySpec;
       
        public final class AesCryptoPBEKey {
       
            // *** 1 *** Явно определяйте режим шифрования и дополнения блоков.
            // *** 2 *** Используйте криптостойкие технологии шифрования, включающие алгоритм, режим блочного шифрования и режим дополнения блоков
            // Параметры передаваемые в метод getInstance класса Cipher: алгоритм шифрования, режим блочного шифрования, режим дополнения блоков
            // в этом примере следующие значения: алгоритм шифрования=AES, режим блочного шифрования=CBC, режим дополнения блоков=PKCS7Padding
            private static final String TRANSFORMATION = "AES/CBC/PKCS7Padding";
       
            // Строка, используемая для получения экземпляра класса, который будет генерировать ключ
            private static final String KEY_GENERATOR_MODE = "PBEWITHSHA256AND128BITAES-CBC-BC";
       
            // *** 3 *** В процессе генерации ключа на основе пароля используйте "соль" (salt)
            // Длина строки "соли" в байтах
            public static final int SALT_LENGTH_BYTES = 20;
       
            // *** 4 *** В процессе генерации ключа на основе пароля используйте достаточное количество итераций хеширования
            // Указание числа повторений смешиваний, используемых при генерации ключей с помощью PBE
            private static final int KEY_GEN_ITERATION_COUNT = 1024;
       
            // *** 5 *** Используйте ключ с длиной, которая обеспечит криптостойкость шифрования
            // Длина ключа в битах
            private static final int KEY_LENGTH_BITS = 128;
       
            private byte[] mSalt = null;
       
            public byte[] getSalt() {
                return mSalt;
            }
       
            private void initSalt() {
                mSalt = new byte[SALT_LENGTH_BYTES];
                SecureRandom sr = new SecureRandom();
                sr.nextBytes(mSalt);
            }
       
            private static final SecretKey generateKey(final char[] password, final byte[] salt) {
                SecretKey secretKey = null;
                PBEKeySpec keySpec = null;
       
                try {
                    // *** 2 *** Используйте криптостойкие технологии шифрования, включающие алгоритм, режим блочного шифрования и режим дополнения блоков
                    // Получение экземпляра класса для генерации ключа
                    // В этом примере используется класс KeyFactory, который применяет алгоритм SHA256 для генерации AES-CBC 128-битного ключа
                    SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance(KEY_GENERATOR_MODE);
       
                    // *** 3 *** В процессе генерации ключа на основе пароля используйте "соль" (salt)
                    // *** 4 *** В процессе генерации ключа на основе пароля используйте достаточное количество итераций хеширования
                    // *** 5 *** Используйте ключ с длиной, которая обеспечит криптостойкость шифрования
                    keySpec = new PBEKeySpec(password, salt, KEY_GEN_ITERATION_COUNT, KEY_LENGTH_BITS);
                    // Очистка пароля - требуется для усложнения процедуры отладки и невключения пароля в дамп памяти
                    Arrays.fill(password, '?');
                    // Генерация ключа
                    secretKey = secretKeyFactory.generateSecret(keySpec);
                } catch (NoSuchAlgorithmException e) {
                } catch (InvalidKeySpecException e) {
                } finally {
                    keySpec.clearPassword();
                }
       
                return secretKey;
            }
        }

In the future, the resulting key value can be used as a password to encrypt the database and there is no need to store it.

## Links

1. [https://github.com/sqlcipher/android-database-sqlcipher](https://github.com/sqlcipher/android-database-sqlcipher)

2. [https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05d-Testing-Data-Storage.md#sqlite-databases-encrypted](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05d-Testing-Data-Storage.md#sqlite-databases-encrypted)

3. [https://cwe.mitre.org/data/definitions/521.html](https://cwe.mitre.org/data/definitions/521.html)