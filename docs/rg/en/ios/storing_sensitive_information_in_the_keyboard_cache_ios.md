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

iOS has a mechanism to auto-complete words that the user enters in the text fields. In this case, if iOS does not know the word the user enters, it can cache the word (or prompt the user to add the word to the dictionary). This feature can be very useful for messenger applications, for example. However, the keyboard cache may disclose sensitive information if it is used to enter such information (credit card data, login, password or personal user information).

## Recommendations

The `autocorrectionType` parameter in the `UITextField` object field is responsible for enabling or disabling the auto-complete option.

**Code Example:**

    UITextField *textField = [[UITextField alloc] initWithFrame:frame]; 
    textField.autocorrectionType = UITextAutocorrectionTypeNo;

All input fields for sensitive information must be marked with the `secureTextEntry`parameter.

**Code Example:**

    UITextField *textField = [[UITextField alloc] initWithFrame:frame]; 
    textField.secureTextEntry = YES;

It is recommended to use implementation of the custom keyboard for entering all sensitive data with caching of all input data disabled. It is also necessary to prohibit copying the entered information to the clipboard to access it from other applications.

**Code Example:**

    - (BOOL)canPerformAction:(SEL)action 
                withSender:(id)sender
    {
    UIMenuController *menuController = [UIMenuController sharedMenuController]; 
    if (menuController) {
        menuController.menuVisible = NO;
    }
    return NO;
    }

## Links

1. [https://develoler.apple.com](https://developer.apple.com/library/archive/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html)
2. [OWASP Mobile Top 10](https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05d-Testing-Data-Storage.md#determining-whether-the-keyboard-cache-is-disabled-for-text-input-fields-mstg-storage-5)
3. [owasp-mstg/0x05d-Testing-Data-Storage.md at master · OWASP/owasp-mstg](https://cwe.mitre.org/data/definitions/200.html)
4. [CWE - CWE-200:](https://cwe.mitre.org/data/definitions/200.html)[ ](https://cwe.mitre.org/data/definitions/200.html)[Exposure of Sensitive Information to an Unauthorized Actor (4.5)](https://cwe.mitre.org/data/definitions/200.html)
5. [Custom keyboard](https://developer.apple.com/documentation/uikit/keyboards_and_input/creating_a_custom_keyboard/configuring_a_custom_keyboard_interface)
6. [Cache keyboard review](https://www.programmersought.com/article/18395753289/)