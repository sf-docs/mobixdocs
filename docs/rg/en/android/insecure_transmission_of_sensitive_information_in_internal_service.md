# Insecure transmission of sensitive information in internal Service

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_info.png"/></td>
        <td>Severity:<strong> INFO</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> DAST, SENSITIVE INFO</strong></td>
      </tr>
    </tbody>
</table>
## Description

An application puts sensitive information into **Intent** to launch an internal **Service**. Generally this is not a vulnerability, but with root access such information can be intercepted.

Interprocess communication (IPC) on Android is performed using a special object — **Intent**. Parameters of **Intent** handlers are set in the main file of the application manifest - ***AndroidManifest.xml*** or, in case of dynamic **BroadcastReceivers**, in the application's code. If an implicit **Intent** is used, i.e. an Intent that does not specify a component; instead, it generally defines an action to be conducted, and lets the system determine which of the available components is best to run for that Intent. For example, if there is a need to display a place on a map, the implicit **Intent** object can request another application, which has such feature, to provide this information. Data in such messages could be compromised. Moreover, malicious applications could use mechanisms of delegation of process control, such as implicit calls to application components or objects like **PendingIntent**, for interception of control and fishing attacks.

The following object types are dangerous: **Activity**, **Service**, **BroadcastReceiver** and **ContentProvider**, because they are open to communication with other applications and don't belong to system Android calls (such as `android.intent.action.MAIN`).  **BroadcastReceiver** is, by default, open to interaction with other applications, so the interception of control or of an **Intent** with confidential information is possible.

## Recommendations

When calling external public **Service**, do not include sensitive information in **Intent**.

Risks from using a **Service** and corresponding countermeasures vary depending on the ways this **Service** is used. To find out what type of a **Service** you are supposed to create, follow the table and chart below.

<figure markdown>
![](../../../rg/img/peredacha-sensitive-informacziya-vo-vnutrennij-service.png)
</figure>
<figure markdown>
![](../../../rg/img/4.png)
</figure>
There are various implementations of a **Service**. Possible combinations of an implementation method and a service type are shown in the table below. OK stands for possible combination. A circle means impossible combination.

<figure markdown>
![](../../../rg/img/sistema_stingrej_nebezopasnaya-peredacha-sensitive-informaczii-v-service-02.png)
</figure>
The example below shows how to properly create and use an internal service. It is important to remember that sensitive information must not be sent when using external public **Service**.

!!! note "Note!" To ensure security of your application, always use an explicit **Intent** object for starting a **Service**, and don't declare **Intent** filters for your services. Starting a service using an implicit **Intent** bears security risks, because there is no certainty in what service will react to the **Intent**, and a user doesn't see what service launches. Starting from Android 5.0 (API level 21) the system throws an exception by calling the **bindService()** method using an implicit **Intent** object.

**Creating and using a private Service**

**Private Service** cannot be launched by other applications and, therefore, is the most secure. When using **Private Services** that are only used within the application, as long as you use explicit **Intents** to the class then you do not have to worry about sending it to any other application.

**Rules (creating a Private Service):**

1. Explicitly specify the `exported="false"` attribute.
2. Verify the received **Intent** and handle it in a secure manner despite the fact that it was sent from the same application.
3. You can include sensitive information in the result **Intent** because it is sent and received within the application.

**AndroidManifest.xml**

    <!--?xml version="1.0" encoding="utf-8"?-->
    <manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.appsec.android.service.privateservice"></manifest>
    <application android:icon="@drawable/ic_launcher" android:label="@string/app_name" android:allowbackup="false">
    <activity android:name=".PrivateUserActivity" android:label="@string/app_name" android:exported="true">
    <intent-filter>
    <action android:name="android.intent.action.MAIN"></action>
    <category android:name="android.intent.category.LAUNCHER"></category>
    </intent-filter>
    </activity></application>
    <!-- Private Service производный от класса Service -->
    <!-- *** 1 *** Явно указывайте атрибут exported="false" -->
    <service android:name=".PrivateStartService" android:exported="false"></service>

**PrivateStartService.java**

    package com.appsec.android.service.privateservice;
    
    import android.app.Service;
    import android.content.Intent;
    import android.os.IBinder;
    import android.widget.Toast;
    
    public class PrivateStartService extends Service {
        
        @Override
        public void onCreate() {
            Toast.makeText(this, "PrivateStartService - onCreate()", Toast.LENGTH_SHORT).show();
        }
    
        @Override
        public int onStartCommand(Intent intent, int flags, int startId) {
            // *** 2 *** Проводите проверку и безопасную обработку полученного Intent, несмотря на то, что он был получен из того же самого приложения
            // См.п. "Безопасная обработка входных данных"
            String param = intent.getStringExtra("PARAM");
            Toast.makeText(this,
                    String.format("PrivateStartService\nПолученные параметры: \"%s\"", param),
                    Toast.LENGTH_LONG).show();
    
            return Service.START_NOT_STICKY;
        }
    
        @Override
        public void onDestroy() {
            Toast.makeText(this, "PrivateStartService - onDestroy()", Toast.LENGTH_SHORT).show();
        }
    
        @Override
        public IBinder onBind(Intent intent) {
            return null;
        }
    }

**Rules (using a private Service):**

!!! note "Note!" To avoid an accidental launch of another application's **Service** always use explicit **Intent** objects for starting your own services and don't declare **Intent** filters for them.

1. Use the explicit **Intent** with class specified to call a **Service** within the application.
2. Sensitive information can be included in the data being sent because it is sent and received within the application.
3. Handle the received result data carefully and securely, even though the data came from a Service in the same application.

**PrivateUserActivity.java**

    package com.appsec.android.service.privateservice;
    
    import android.app.Activity;
    import android.content.Intent;
    import android.os.Bundle;
    import android.view.View;
    
    public class PrivateUserActivity extends Activity {
    
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.privateservice_activity);
        }
    
        // StartService
        
        public void onStartServiceClick(View v) {
            // *** 1 *** Используйте явный Intent с указанием имени класса Service внутри приложения
            Intent intent = new Intent(this, PrivateStartService.class);
    
            // *** 2 *** В передаваемые данные можно включать конфиденциальную информацию, т.к. их отправка и получение происходит внутри приложения
            intent.putExtra("PARAM", "Чувствительная ифнормация");
    
            startService(intent);
        }
    
        public void onStopServiceClick(View v) {
            doStopService();
        }
    
        @Override
        public void onStop() {
            super.onStop();
            doStopService();
        }
    
        private void doStopService() {
            // *** 1 *** Используйте явный Intent с указанием имени класса Service внутри приложения
            Intent intent = new Intent(this, PrivateStartService.class);
            stopService(intent);
        }
    }

## Links

1. [https://developer.android.com/guide/components/intents-filters?hl=ru](https://developer.android.com/guide/components/intents-filters?hl=ru)

2. [https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05h-Testing-Platform-Interaction.md](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05h-Testing-Platform-Interaction.md)

3. [https://developer.android.com/training/basics/intents/index.html](https://developer.android.com/training/basics/intents/index.html)