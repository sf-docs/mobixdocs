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

iOS provides an option for applications to output information into a system log. Applications can send information to the system log using the following:

* the NSLog macro
* the printf family of functions
* the NSAssert family of functions
* Macros

In the process of developing an application, sections of code with logging of sensitive information (passwords, personal data, encryption keys) can get into the final version.

## Recommendations

### Changing the behavior of logging macros depending on the type of application build.

Code Example:

    #ifdef DEBUG
    #   define NSLog (...) NSLog(__VA_ARGS__)
    #else
    #   define NSLog (...)
    #endif

### Using the OSLogPrivacy framework to differentiate the output depending on the sensitivity of the data.

Code Example:

    @frozen struct OSLogPrivacy
    ... 
    Logger().info("User bank account number: \(accountNumber, privacy: .private)")

## Links

1. [Poor Logging Practice \| OWASP](https://www.owasp.org/index.php/Poor_Logging_Practice)
2. [CWE-215:](https://cwe.mitre.org/data/definitions/215.html)[ ](https://cwe.mitre.org/data/definitions/215.html)[Insertion of Sensitive Information Into Debugging Code](https://cwe.mitre.org/data/definitions/215.html)
3. [OSLogPrivacy](https://developer.apple.com/documentation/os/oslogprivacy)