# Storing sensitive information in an unprotected database

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
        <td>Detection method:<strong> DAST, DATA BASES</strong></td>
      </tr>
    </tbody>
</table>
## Description

The application stores sensitive information in an unprotected database, which can lead to data compromise.

Although the file is stored inside the application directory, you should not store sensitive information in it. This information can be obtained in many ways, from local or cloud-based backups, to various file reading vulnerabilities and injections into the Content Provider.

In order to understand what kind of data needs to be protected, you first need to determine what data the application processes and stores and what part of that information is considered sensitive. In such a situation, it is common to rely on legislation and common sense. There is no point in encrypting absolutely all the information the application stores. This can affect speed and stability of the application. Instead, you should clearly define what exactly is sensitive data for your application or company, and focus your attention on that data.

## Recommendations

If you need to store sensitive information in a database, you must additionally encrypt the database or the data stored in it. As an example for database encryption, you can use the [sqlcipher](https://github.com/sqlcipher/sqlcipher).

**An example of using SQLCipher (Java):**

    package com.demo.sqlcipher;
    import java.io.File;
    import net.sqlcipher.database.SQLiteDatabase;
    import android.app.Activity;
    import android.os.Bundle;
    public class HelloSQLCipherActivity extends Activity {
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main);
            InitializeSQLCipher();
        }
        private void InitializeSQLCipher() {
            SQLiteDatabase.loadLibs(this);
            File databaseFile = getDatabasePath("demo.db");
            databaseFile.mkdirs();
            databaseFile.delete();
            SQLiteDatabase database = SQLiteDatabase.openOrCreateDatabase(databaseFile, "test123", null);
            database.execSQL("create table t1(a, b)");
            database.execSQL("insert into t1(a, b) values(?, ?)", new Object[]{"one for the money",
                                                                            "two for the show"});
        }
    }

**An example of using SQLCipher (Kotlin):**

    package com.demo.sqlcipher
    import android.os.Bundle
    import androidx.appcompat.app.AppCompatActivity
    import net.sqlcipher.database.SQLiteDatabase
    class MainActivity : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)
            SQLiteDatabase.loadLibs(this)
            val databaseFile = getDatabasePath("demo.db")
            if(databaseFile.exists()) databaseFile.delete()
            databaseFile.mkdirs()
            databaseFile.delete()
            val database = SQLiteDatabase.openOrCreateDatabase(databaseFile, "test123", null)
            database.execSQL("create table t1(a, b)")
            database.execSQL("insert into t1(a, b) values(?, ?)",
                arrayOf<Any>("one for the money", "two for the show")
            )
        }
    }

This example uses the hardcoded **"test123"** password for the database. In a real application, you should not use such an insecure password and store it in the source code or in an open form.

One way of not storing the password is to generate it "on the fly" using the user's password based on the key strengthening procedure.

Rules:

1. Explicitly define the encryption mode and block padding.

2. Use crypto-resistant encryption technologies including algorithm, block encryption mode, and block padding mode.

3. Use "salt" when generating a password-based key.

4. Use a sufficient number of hashing iterations while generating the key based on the password.

5. Use a key with a length that ensures cryptographic strength of the encryption.
   
        import android.os.Build
        package com.appsec.android.cryptsymmetricpasswordbasedkey;
        import javax.crypto.SecretKeyFactory
        import javax.crypto.spec.PBEKeySpec
        @Deprecated("Use Argon2 instead")
        internal object Pbkdf2Factory {
            private const val DEFAULT_ITERATIONS = 10_000
            private val systemAlgorithm by lazy {
                if (Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
                    "PBKDF2withHmacSHA1"
                } else {
                    "PBKDF2withHmacSHA256"
                }
            }
            fun createKey(
                passphraseOrPin: CharArray,
                salt: ByteArray,
                algorithm: String = systemAlgorithm,
                iterations: Int = DEFAULT_ITERATIONS
            ): Pbkdf2Key {
                @Suppress("MagicNumber")
                val keySpec = PBEKeySpec(passphraseOrPin, salt, iterations, 256)
                val secretKey = SecretKeyFactory.getInstance(algorithm).generateSecret(keySpec)
                return Pbkdf2Key(
                    secretKey.algorithm,
                    iterations,
                    salt,
                    secretKey.encoded
                )
            }
        }

In the future, the resulting key value can be used as a password to encrypt the database and there is no need to store it. Each time you enter a password, the key will be generated on the fly and sent to the database open function.

!!! note "Note!" With this approach, if the secret (user password) is changed, the data must be re-encrypted with a new secret if it is to be stored. Also, if the user's password is a pin code, it is better to use the KEK (Key Encryption Key) + DEK (Data Encryption Key) approach. This involves creating a key for encrypting the data and also encrypting this key with the user's password. With this approach, if you change the secret, you only need to re-encrypt the key; the encrypted user data remains untouched.

!!! note "Note" When you link the SQLCipher library, do not forget to add rules to Proguard to make the application work correctly.

The rules for ProGuard:

    -keep,includedescriptorclasses class net.sqlcipher.** { *; }
    -keep,includedescriptorclasses interface net.sqlcipher.** { *; }

## Links

1. [GitHub - sqlcipher/android-database-sqlcipher:](https://github.com/sqlcipher/android-database-sqlcipher)[ ](https://github.com/sqlcipher/android-database-sqlcipher)[Android SQLite API based on SQLCipher](https://github.com/sqlcipher/android-database-sqlcipher)
2. [owasp-mstg/0x05d-Testing-Data-Storage.md at master · OWASP/owasp-mstg](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05d-Testing-Data-Storage.md#sqlite-databases-encrypted)
3. [CWE - CWE-521:](https://cwe.mitre.org/data/definitions/521.html)[ ](https://cwe.mitre.org/data/definitions/521.html)[Weak Password Requirements (4.6) What is a Data Encryption Key (DEK)](https://cwe.mitre.org/data/definitions/521.html)
4. [Definition from Techopedia](https://www.techopedia.com/definition/5660/data-encryption-key-dek)