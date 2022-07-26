# Application is not obfuscated

<table class='noborder'>
    <colgroup>
      <col/>
      <col/>
    </colgroup>
    <tbody>
      <tr>
        <td rowspan="2"><img src="../../../img/defekt_nizkij.png"/></td>
        <td>Severity:<strong> LOW</strong></td>
      </tr>
      <tr>
        <td>Detection method:<strong> SAST, APK</strong></td>
      </tr>
    </tbody>
</table>
## Description

The source code of the application is not obfuscated or is obfuscated with insufficient coverage.

Weak or no obfuscation of the application leads to easier examination of the code after decompilation. This allows an attacker to easily analyze the application code to find vulnerabilities or ways to bypass security.

## Recommendations

Before publishing an application, make sure that rules for obfuscation are enabled in the release version of the application and that they are configured properly.

One common solution is to use the ProGuard built-in obfuscator rules to automatically enable/disable obfuscation depending on the build type (release/debug).

**An example of ProGuard settings:**

    optimizationpasses 5
    -dontusemixedcaseclassnames
    -dontskipnonpubliclibraryclasses
    -dontskipnonpubliclibraryclassmembers
    -dontpreverify
    -verbose
    -dump class_files.txt
    -printseeds seeds.txt
    -printusage unused.txt
    -printmapping mapping.txt
    -optimizations !code/simplification/arithmetic,!field/*,!class/merging/*
    -allowaccessmodification
    -keepattributes *Annotation*
    -renamesourcefileattribute SourceFile
    -keepattributes SourceFile,LineNumberTable
    -repackageclasses ''

**Enabling the use of ProGuard for the release build of an application:**

    buildTypes {
        ...
        release {
            minifyEnabled true
            proguardFiles 'proguard-rules.pro', getDefaultProguardFile('proguard-android.txt')
        }
    }

It is also good practice to obfuscate the open source libraries used in the project. Recently, more and more libraries are distributed with ready-made configuration files for obfuscation. ProGuard knows how to look inside the archive, find the library configuration files and add it to the rest of the options. Check every library you use for such a file.

If the authors of the library do not pack the config file in the archive, maybe they took care and wrote the rules on their website, the repository page or in the README file. Try to find the config file for the version of the library you are using yourself.

## Links

1. [https://habr.com/ru/post/415499/](https://habr.com/ru/post/415499/)

2. [https://developer.android.com/studio/build/shrink-code](https://developer.android.com/studio/build/shrink-code)

3. [https://www.guardsquare.com/en/products/proguard/manual/examples](https://www.guardsquare.com/en/products/proguard/manual/examples)

4. [https://github.com/OWASP/owasp-stg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md#make-sure-that-free-security-features-are-activated-mstg-code-9](https://github.com/OWASP/owasp-stg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md#make-sure-that-free-security-features-are-activated-mstg-code-9)