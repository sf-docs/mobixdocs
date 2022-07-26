# Storing sensitive information in a protected database

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
        <td>Detection method:<strong> DAST, API</strong></td>
      </tr>
    </tbody>
</table>
## Description

The application stores sensitive information in a protected database. In general this is not a vulnerability, but it is necessary to make sure that a [strong password] is used to encrypt the database.

Sensitive data found is used by the system to find its use or [storage in the collected data].

## Recommendations

To protect against data interception at runtime, it is necessary to use protection measures to detect application tooling and root access detection. One of the good ways is to use the [DetectFrida](https://github.com/darvincisec/DetectFrida)and [ DetectMagiskHide](https://github.com/darvincisec/DetectMagiskHide). These libraries implement checks in native code, which makes their analysis and modification much more difficult.

## Links

1. [https://github.com/sqlcipher/android-database-sqlcipher](https://github.com/sqlcipher/android-database-sqlcipher)

2. [https://github.com/darvincisec/DetectMagiskHide](https://github.com/darvincisec/DetectMagiskHide)

3. [https://github.com/darvincisec/DetectFrida](https://github.com/darvincisec/DetectFrida)

4. [https://darvincitech.wordpress.com/2019/12/23/detect-frida-for-android/](https://darvincitech.wordpress.com/2019/12/23/detect-frida-for-android/)

5. [https://darvincitech.wordpress.com/2019/11/04/detecting-magisk-hide/](https://darvincitech.wordpress.com/2019/11/04/detecting-magisk-hide/)

6. [https://github.com/OWASP/owasp-stg/blob/master/Document/0x05d-Testing-Data-Storage.md#sqlite-databases-encrypted](https://github.com/OWASP/owasp-stg/blob/master/Document/0x05d-Testing-Data-Storage.md#sqlite-databases-encrypted)