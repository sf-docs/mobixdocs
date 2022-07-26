# Possibility to access an arbitrary ContentProvider

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
        <td>Detection method:<strong> IAST</strong></td>
      </tr>
    </tbody>
</table>
## Description

This vulnerability allows access to an internal non-exportable **ContentProvider**.

The vulnerability is present in applications that use **Intent** from an untrusted source (e.g. obtained from a third-party application using `getIntent`, `getParcelableExtra` or `onActivityResult` methods) to return data using the `setResult` method.

For example, a malicious application can use such code:

    Intent intent = new Intent();
    intent.setData(Uri.parse("content://com.victim.provider/secret_data.txt"));
    intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
    intent.setClassName("vuln.app.pkg", "vuln.app.pkg.SomeActivity");
    startActivityForResult(intent, 0);

Target vulnerable application (***SomeActivity.java***):

    super.onCreate(savedInstanceState);
    setResult(RESULT_OK, getIntent());
    finish();

As a result, the malicious application will gain access to the **ContentProvider** `com.victim.provider` of the vulnerable application.

## Recommendations

To avoid such problems in the application, you need to make sure to follow a few rules:

1. Implement private/in-house visibility for components that accept **Intent** and use it in the `setResult` method.  For example, declaration of **Activity** internal - no `intent-filter` or `exported` flag is set to `false`.
   
        <?xml version="1.0" encoding="utf-8"?>
        <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.swordfishsecurity.appsec.android.activity.privateactivity" >
       
            <application
                android:allowBackup="false"
                android:icon="@drawable/ic_launcher"
                android:label="@string/app_name" >
       
                <!-- Private activity -->
                <!-- *** 1 *** Не используйте taskAffinity -->
                <!-- *** 2 *** Не используйте launchMode -->
                <!-- *** 3 *** Явно указывайте атрибут exported="false" -->
                <activity
                    android:name=".PrivateActivity"
                    android:label="@string/app_name"
                    android:exported="false" />
       
                <!-- Public activity запускаемая по умолчанию -->
                <activity
                    android:name=".PrivateUserActivity"
                    android:label="@string/app_name"
                    android:exported="true" >
                    <intent-filter>
                        <action android:name="android.intent.action.MAIN" />
                        <category android:name="android.intent.category.LAUNCHER" />
                    </intent-filter>
                </activity>
            </application>
        </manifest>

2. Validate **Intent** for maliciousness:
   
   1. Such **Intent** should not be directed to private/in-house components or components of external applications.
      
           Intent intent = getIntent();
           Intent redirectIntent = (Intent) intent.getParcelableExtra(“redirect_intent”);
           ComponentName name = redirectIntent.resolveActivity(getPackageManager());
           // проверяем целевое имя пакета и класса
           if(name.getPackageName().equals(“safe_package”) && name.getClassName().equals(“safe_class”)) {
           startActivity(redirectIntent);
           }
   
   2. If you still want to run an external application component, then you need to validate/sanitize **Permissions** passed into "**to-be-redirected Intent**".  
An example of validation:
      
               Intent resultIntent = (Intent) intent.getParcelableExtra(“result_intent”);
               int flags = resultIntent.getFlags();
               if((flags & Intent.FLAG_GRANT_READ_URI_PERMISSION == 0) && (flags & Intent.FLAG_GRANT_WRITE_URI_PERMISSION == 0)) {
               setResult(RESULT_OK, resultIntent);
               }
      
      An example of sanitization:
      
               resultIntent.removeFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
               resultIntent.removeFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
               setResult(RESULT_OK, intent);

## Links

1. [https://developer.android.com/guide/topics/manifest/activity-element#exported](https://developer.android.com/guide/topics/manifest/activity-element#exported)

2. [https://blog.oversecured.com/Android-Access-to-app-protected-components/](https://blog.oversecured.com/Android-Access-to-app-protected-components/)