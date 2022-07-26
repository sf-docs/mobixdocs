# Possibility to access an arbitrary file using ContentProvider

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

This vulnerability allows access to application files using **exported ContentProvider**.

The vulnerability is present in applications where implementation of the `openFile` method of a **ContentProvider** derived class does not properly check the Uri-parameter.  A malicious application can specifically generate a Uri, pass it to this **ContentProvider** and gain access to an arbitrary file.

An example of vulnerable code:

    @Override
    public ParcelFileDescriptor openFile(Uri uri, String mode) throws FileNotFoundException {
    File file = new File(getContext().getFilesDir(), uri.getLastPathSegment());
    return ParcelFileDescriptor.open(file, ParcelFileDescriptor.MODE_READ_WRITE);
    }

A malicious application can use such code:

    Uri uri = Uri.parse("content://vuln.app.pkg.some_authority/private_internal_file");
    try {
    Log.d("Evil", IOUtils.toString(getContentResolver().openInputStream(uri), Charset.defaultCharset()));
    } catch (Throwable th) {
    Log.e("Evil", "Error was occured during openInputStream call");
    throw new RuntimeException(th);
    }

As a result, a malicious application will access the `private_internal_file` file in the application directory of the vulnerable application (`vuln.app.pkg`).

## Recommendations

To avoid such problems in the application, you need to make sure to follow a few rules:

1. Implement private/in-house visibility in **ContentProvider**.
   
   As an example, declaring **ContentProvider** as internal:
   
        <provider
        android:name=".PrivateProvider"
        android:authorities="notvuln.app.pkg.some_authority"
        android:exported="false" />
   
   In order to protect a **ContentProvider** from being used by third-party applications, you need to define a `permission` with `protectionLevel="signature"` and put it in the declaration of that **ContentProvider**:
   
        <?xml version="1.0" encoding="utf-8"?>
        <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="notvuln.app.pkg">
       
            <!-- *** 1 *** Определите in-house полномочие (permission) с protectionLevel="signature" -->
            <permission
                android:name="notvuln.app.pkg.inhouseprovider.MY_PERMISSION"
                android:protectionLevel="signature" />
       
            <application
                android:icon="@drawable/ic_launcher"
                android:label="@string/app_name" >
       
                <!-- *** 2 *** Ограничьте доступ к **ContentProvider** при его объявлении с помощью in-house полномочия -->
                <!-- *** 3 *** Явно указывайте атрибут exported="true" -->
                <provider
                    android:name=".InhouseProvider"
                    android:authorities="notvuln.app.pkg.inhouseprovider"
                    android:permission="notvuln.app.pkg.inhouseprovider.MY_PERMISSION"
                    android:exported="true" />
            </application>
        </manifest>

2. If a **ContentProvider** must remain public to third-party applications, you need to validate the canonical file path just before returning it to the requesting application:
   
        @Override
        public ParcelFileDescriptor openFile (Uri uri, String mode) throws FileNotFoundException {
        File file = new File(sdcardDir, uri.getLastPathSegment());
        if (!file.getCanonicalPath().startsWith(sdcardDir)) {
            throw new IllegalArgumentException();
        }
        return ParcelFileDescriptor.open(file, ParcelFileDescriptor.MODE_READ_ONLY);
        }