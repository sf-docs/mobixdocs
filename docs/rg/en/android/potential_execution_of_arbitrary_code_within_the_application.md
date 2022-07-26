# Potential execution of arbitrary code within the application

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_kritichnyj.png"/></td>
        <td>Severity:<strong> CRITICAL</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> DAST, IAST</strong></td>
      </tr>
    </tbody>
</table>
## Description

The application uses a publicly available archive, that can be replaced by an attacker and used to execute arbitrary code.

To implement this vulnerability and execute arbitrary code within the application, several conditions should be met:

* The application uses native code (loads binary libraries with the .so extension when starting or working).
* The application interacts with an archive (zip, 7zip, etc.) available in public directories (that is, it can be replaced by an attacker).
* When working with the archive, there are no checks for special characters in the file name (e.g., `zipFile.getName().contains("../")`). If there are no such checks, it means that it is possible to overwrite arbitrary files when unzipping the file.

**The vulnerability is implemented in the following way:**

* The attacker identifies the native libraries loaded into the application and which archive the application communicates with.
* An attacker prepares a native library containing code that performs certain actions when **JNI\_OnLoad** is loaded ( for example, changing the permissions on a directory to 777, making the directory accessible to everyone). The name of the file must match one of the libraries loaded by the application.

**Code Example:**

    #include <jni.h>
    #include <string.h>
    #include <stdlib.h>
    JNIEXPORT jint **JNI_OnLoad**(JavaVM* vm, void* reserved) {
    system("chmod -R 777 /data/user/0/com.example.app/");
    JNIEnv* env;
    if (vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6) != JNI_OK) {
    return JNI_ERR;
    }
    return JNI_VERSION_1_6;
    }

* Using special tools ([like this script](https://github.com/ptoomey3/evilarc)) prepares an archive that contains the library written inside.
* When working with an application (or when calling methods that work with archives, such as the Content Provider), the function to unpack the previously created archive is called.
* The application unzips the archive containing the navigation elements into the directory above (../../) and the path of the directory inside the application sandbox where you want to place the file.
* As a result of unpacking such an archive, the application replaces the library inside its directory with a library from the archive.
* The next time the application is run or the library is loaded, the code from the attacker's library will be executed.

## Recommendations

When using any file handling mechanism that can potentially be controlled by an attacker or when implementing interprocess communication mechanisms using files (Content Providers, etc.) it is necessary to validate file names for special characters that can be used for path traversal.

A simple example of such a check could be a code fragment for unpacking an archive or a file name/path analysis passed to the IPC mechanisms:

    zipFile.getName().contains("../")

## Links

1. [https://blog.oversecured.com/Oversecured-detects-dangerous-vulnerabilities-in-the-TikTok-Android-app/](https://blog.oversecured.com/Oversecured-detects-dangerous-vulnerabilities-in-the-TikTok-Android-app/)

2. [https://github.com/ptoomey3/evilarc](https://github.com/ptoomey3/evilarc)

3. [https://twitter.com/\_bagipro/status/1319365830728208386https://developer.android.com/reference/javax/crypto/KeyGenerator.html](https://twitter.com/_bagipro/status/1319365830728208386https://developer.android.com/reference/javax/crypto/KeyGenerator.html)