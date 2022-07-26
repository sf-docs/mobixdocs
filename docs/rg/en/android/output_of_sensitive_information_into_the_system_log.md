# Output of sensitive information into the system log

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
        <td>Detection method:<strong> DAST, SENSITIVE INFO</strong></td>
      </tr>
    </tbody>
</table>
## Description

Android provides an option for applications to output information into the system log. Applications can send information into a log file using the `android.util.Log` class.

Before Android 4.0, any app with **READ\_LOGS** permission could access the entire system log, including system logs and logs of other apps. After Android 4.1, the **READ\_LOGS** permission specification has been changed and an application can only access its own data. However, by connecting an Android device to a PC, you can retrieve system log data from other applications.

This is why it is so important to ensure that applications do not send confidential information into the system log. 

The `android.util.Log` class provides a number of possibilities to output information:

* Log.d (Debug).

* Log.e (Error).

* Log.i (Info).

* Log.v (Verbose).

* Log.w (Warn).

Also, libraries with similar functionality can be used. One of them is [Timber](https://github.com/JakeWharton/timber).

**Example of vulnerable code**

    Log.d("authorize", "Login Success! access_token="
        + getAccessToken() + " expires="
        + getAccessExpires());

## Recommendations

Before publishing an application you need to ensure that confidential information does not go into the system log. Also, if an application uses third-party libraries, you must make sure that these libraries also do not send confidential information and are configured accordingly (a release version of each library is used or correct attributes are set).

One of the well-known solutions is to declare and use a user class of logging to automatically include / exclude the information into the system log in accordance with its build type (release / debug).

    if (BuildConfig.DEBUG) {
            ...
            serverEditText.setText("http://test.test");
            loginEditText.setText("user_test");
            passwordEditText.setText("12345");
            ...
        }

It is also good practice to use ProGuard to delete particular logging calls. And to exclude during the release build logging from **Timber** and **android.util.Log** libraries.

**Example of ProGuard settings**

    -assumenosideeffects class android.util.Log {
        public static boolean isLoggable(java.lang.String, int);
        public static int v(...);
        public static int i(...);
        public static int w(...);
        public static int d(...);
        public static int e(...);
    }
    -assumenosideeffects class timber.log.Timber* {
        public static *** v(...);
        public static *** d(...);
        public static *** i(...);
        public static *** e(...);
        public static *** w(...);
    }

**Enabling use of ProGuard for a release build of an application**

    buildTypes {
        releaseSomeBuildType {
            ...
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'your-proguard-file.pro'
        }
    } 

## Links

1. [https://www.owasp.org/index.php/Poor\_Logging\_Practice](https://www.owasp.org/index.php/Poor_Logging_Practice)

2. [https://cwe.mitre.org/data/definitions/778.html](https://cwe.mitre.org/data/definitions/778.html)

3. [https://source.android.com/setup/contribute/code-style#log-sparingly](https://source.android.com/setup/contribute/code-style#log-sparingly)