# Storing sensitive information in memory

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
        <td>Detection method:<strong> DAST, HEAPDUMP</strong></td>
      </tr>
    </tbody>
</table>
## Description

Memory analysis can help developers identify the root causes of a number of problems that occur while an application is running on a device. However, it can also be used to access sensitive data. When an application runs on a device, user data or application-specific data can be stored in RAM and not cleared properly after a user exits the system or the application. Since Android stores applications in memory (even after exit) until it is restored, various sensitive information can remain in memory indefinitely. An attacker who discovers or steals the device can plug in a debugger and dump the application's memory.

## Recommendations

Do not store sensitive data (such as encryption keys) in RAM longer than necessary. Clear all variables that contain sensitive information after it has been used. Avoid using constant objects (such as `Android java.lang.String`) for cryptographic keys or passwords.

Store confidential information in primitive data arrays, such as byte arrays (`byte []`) and char arrays (`char []`). This helps to correctly clear confidential information from memory.

There are several methods of clearing information in memory, one of them is to overwrite the contents with zeros.

**Example: Java**

    byte[] secret = null;
    try{
        //get or generate the secret, do work with it, make sure you make no local copies
    } finally {
        if (null != secret) {
            Arrays.fill(secret, (byte) 0);
        }
    }

**Example: Kotlin**

    val secret: ByteArray? = null
    try {
        //get or generate the secret, do work with it, make sure you make no local copies
    } finally {
        if (null != secret) {
            Arrays.fill(secret, 0.toByte())
        }
    }

Unfortunately, this does not guarantee that the content will be overwritten at runtime. To optimize the bytecode, compiler will analyze and decide not to overwrite the data because it will not be used later. From the compiler's point of view, this is an unnecessary operation.

This problem doesn't have an ideal solution. For example, you can perform additional calculations (such as XOR for data in a dummy buffer), but you won't know if the compiler decided to remove those operations. On the other hand, using data outside the compiler (for example, serializing the data in a temporary file) guarantees that the data will be overwritten, but, obviously, this affects performance.

Moreover, using `Arrays.fill` to overwrite data might be a bad idea because this method could be intercepted, which is often done by various instruments for analysis. Another problem in the example given above is that the contents are overwritten with zeros only. Ideally, you should overwrite objects with sensitive information with random data or contents of other variables.

**Example: Java**

    byte[] nonSecret = somePublicString.getBytes("ISO-8859-1");
    byte[] secret = null;
    try{
        //get or generate the secret, do work with it, make sure you make no local copies
    } finally {
        if (null != secret) {
            for (int i = 0; i < secret.length; i++) {
                secret[i] = nonSecret[i % nonSecret.length];
            }
            FileOutputStream out = new FileOutputStream("/dev/null");
            out.write(secret);
            out.flush();
            out.close();
        }
    }

**Example: Kotlin**

    val nonSecret: ByteArray = somePublicString.getBytes("ISO-8859-1")
    val secret: ByteArray? = null
    try {
        //get or generate the secret, do work with it, make sure you make no local copies
    } finally {
        if (null != secret) {
            for (i in secret.indices) {
                secret[i] = nonSecret[i % nonSecret.size]
            }
            val out = FileOutputStream("/dev/null")
            out.write(secret)
            out.flush()
            out.close()
            }
    }

## Links

1. [https://mobile-security.gitbook.io/mobile-security-testing-guide/android-testing-guide/0x05d-testing-data-storage](https://mobile-security.gitbook.io/mobile-security-testing-guide/android-testing-guide/0x05d-testing-data-storage)

2. [https://cwe.mitre.org/data/definitions/316.html](https://cwe.mitre.org/data/definitions/316.html)

3. [https://www.pentestpartners.com/security-blog/how-to-extract-sensitive-plaintext-data-from-android-memory/](https://www.pentestpartners.com/security-blog/how-to-extract-sensitive-plaintext-data-from-android-memory/)

4. [https://securitygrind.com/dumping-and-analyzing-android-application-memory/](https://securitygrind.com/dumping-and-analyzing-android-application-memory/)

5. [https://developer.android.com/studio/profile/memory-profiler](https://developer.android.com/studio/profile/memory-profiler)