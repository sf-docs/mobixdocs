# Storing sensitive information in the keyboard cache

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
        <td>Detection method:<strong> DAST, SENSITIVE INFO</strong></td>
      </tr>
    </tbody>
</table>
## Description

Android has a mechanism to auto-complete words that the user enters in the text fields. In this case, if Android does not know the word the user enters, it can cache the word (or prompt the user to add the word to the dictionary). This feature can be very useful for messenger applications, for example. However, the keyboard cache may disclose sensitive information if it is used to enter such information (credit card data, login, password or personal user information).

The `android:inputType="textNoSuggestions"` parameter in the description of the (`<EditText/>`) is responsible for enabling or disabling the auto-complete option.

**An example of vulnerable code:**

    <EditText 
    android:id="@+id/KeyBoardCache"/>

## Recommendations

All input fields that request confidential information must have the following XML attribute enabled (to disable auto-completes):

    <EditText 
    android:id="@+id/KeyBoardCache" 
    android:inputType="textNoSuggestions"/>

## Links

1. [https://developer.android.com/reference/android/text/InputType.html#TYPE\_TEXT\_FLAG\_NO\_SUGGESTIONS](https://developer.android.com/reference/android/text/InputType.html#TYPE_TEXT_FLAG_NO_SUGGESTIONS)

2. [https://www.owasp.org/index.php/Mobile\_Top\_10\_2016-M2-Insecure\_Data\_Storage](https://www.owasp.org/index.php/Mobile_Top_10_2016-M2-Insecure_Data_Storage)

3. [https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05d-Testing-Data-Storage.md#determining-whether-the-keyboard-cache-is-disabled-for-text-input-fields-mstg-storage-5](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05d-Testing-Data-Storage.md#determining-whether-the-keyboard-cache-is-disabled-for-text-input-fields-mstg-storage-5)

4. [https://cwe.mitre.org/data/definitions/200.html](https://cwe.mitre.org/data/definitions/200.html)