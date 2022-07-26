# Transmission of sensitive information in SQL query parameters

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
        <td>Detection method:<strong> DAST, SQL</strong></td>
      </tr>
    </tbody>
</table>
## Description

An application uses sensitive information when sending queries to the database. SQL query interception is not a vulnerability if measures are used to detect application tooling using tools such as Frida or Xposed, root access checks are performed, and the database storing sensitive information is [encrypted](./storing_sensitive_information_in_a_protected_database.md) using [a strong password](./weak_database_encryption_password.md).

Intercepted data is used by Mobix to search for the intercepted [value in the collected data](./storage_or_use_of_previously_found_sensitive_information.md).

## Recommendations

To protect against runtime password interception, it is necessary to use protection measures to detect application tooling and root access detection. One of the good ways is to use the [DetectFrida](https://github.com/darvincisec/DetectFrida)and [ DetectMagiskHide](https://github.com/darvincisec/DetectMagiskHide). These libraries implement checks in native code, which makes their analysis and modification much more difficult.

## Links

1. [https://github.com/sqlcipher/android-database-sqlcipher](https://github.com/sqlcipher/android-database-sqlcipher)

2. [https://github.com/darvincisec/DetectMagiskHide](https://github.com/darvincisec/DetectMagiskHide)

3. [https://github.com/darvincisec/DetectFrida](https://github.com/darvincisec/DetectFrida)

4. [https://darvincitech.wordpress.com/2019/12/23/detect-frida-for-android/](https://darvincitech.wordpress.com/2019/12/23/detect-frida-for-android/)

5. [https://darvincitech.wordpress.com/2019/11/04/detecting-magisk-hide/](https://darvincitech.wordpress.com/2019/11/04/detecting-magisk-hide/)

6. [https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05d-Testing-Data-Storage.md#sqlite-databases-encrypted](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05d-Testing-Data-Storage.md#sqlite-databases-encrypted)