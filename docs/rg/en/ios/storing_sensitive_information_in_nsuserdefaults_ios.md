# Storing sensitive information in NSUserDefaults

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
        <td>Detection method:<strong> DAST, ФАЙЛЫ ПРИЛОЖЕНИЯ</strong></td>
      </tr>
    </tbody>
</table>
## Description

An application stores sensitive information in a private file inside the application's directory.

To understand exactly what needs to be protected, you have to analyze what kind of data is being handled and stored by the application and what part of that data is confidential. In such a situation, it is common to rely on legislation and common sense. There is no point in encrypting absolutely all the information the application stores. This can affect speed and stability of the application. Instead, you should clearly define what exactly is sensitive data for your application or company, and focus your attention on that data.

It is generally assumed that as few confidential information as possible should be stored in a local storage (both internal and external). However, in most cases storing such information cannot be avoided. For example, in terms of usability, you should not force user to enter a complex password every time the application is launched. Most applications have to cache some authentication token locally. Personally identifiable information (PII) and other types of confidential data could also be retained if a specific scenario calls for it.

Application can store data in different formats, one of which is NSUserDefaults.

NSUserDefaults is designed to store relatively small amounts of frequently requested and rarely modified data. Other ways of using it may result in slower performance or more memory consumption than more appropriate solutions.

Storing sensitive data using the NSUserDefaults mechanism in a public domain is not recommended. Since its physical representation is just a file in the file system of the device, located inside the application directory at the relative path `/Library/Preferences/com.yourcompany.appName.plist`. Values from this file can be obtained through local or cloud backups, or by exploiting various vulnerabilities.

!!! note "Note!" Very often it is mistakenly thought that the data stored in the internal directory of an application is already protected by the sandbox mechanism and an attacker cannot get to it. There are many ways, ranging from a simple local or cloud-based application backup to physical access to the device and exploitation of various vulnerabilities. **Plain text information put into the application's directory is not protected!**

## Recommendations

Any sensitive information that is stored on the device must be encrypted. This can be done in many ways. One of them is encryption based on a key derived from user data (password, pin code, etc.) using key strengthening algorithms ([Key Stretching](https://en.wikipedia.org/wiki/Key_stretching)). This allows you to get an encryption key from a fairly simple password by applying the hash function to it several times along with a salt. The salt is a certain sequence of random data. A common mistake is to exclude salt from the algorithm. Salt gives the key much more entropy. Without it, it is much easier to retrieve/recover/find the key. Moreover, without using salt, two identical passwords will have the same hash value and, consequently, the same final encryption key value.

In this case, since the key strengthening algorithm is used, there is no need to store the key somewhere. Each time the need for a key arises, it is enough to use the user's data to generate it.

To encrypt and decrypt use the `CCCrypt` function with `kCCEncrypt` or `kCCDecrypt`. Since a block cipher is used, it is necessary to pad the message if it does not match the multiplicity of the block size. Using the `KCCOptionPKCS7Padding` parameter, define the padding type as PKCS7:

### Encryption

    class func encryptData(_ clearTextData : Data, withPassword password : String) -> Dictionary<String, Data>
        {
            var setupSuccess = true
            var outDictionary = Dictionary<String, Data>.init()
            var key = Data(repeating:0, count:kCCKeySizeAES256)
            var salt = Data(count: 8)
            salt.withUnsafeMutableBytes { (saltBytes: UnsafeMutablePointer<UInt8>) -> Void in
                let saltStatus = SecRandomCopyBytes(kSecRandomDefault, salt.count, saltBytes)
                if saltStatus == errSecSuccess
                {
                    let passwordData = password.data(using:String.Encoding.utf8)!
                    key.withUnsafeMutableBytes { (keyBytes : UnsafeMutablePointer<UInt8>) in
                        let derivationStatus = CCKeyDerivationPBKDF(CCPBKDFAlgorithm(kCCPBKDF2), password, passwordData.count, saltBytes, salt.count, CCPseudoRandomAlgorithm(kCCPRFHmacAlgSHA512), 14271, keyBytes, key.count)
                        if derivationStatus != Int32(kCCSuccess)
                        {
                            setupSuccess = false
                        }
                    }
                }
                else
                {
                    setupSuccess = false
                }
            }
            
            var iv = Data.init(count: kCCBlockSizeAES128)
            iv.withUnsafeMutableBytes { (ivBytes : UnsafeMutablePointer<UInt8>) in
                let ivStatus = SecRandomCopyBytes(kSecRandomDefault, kCCBlockSizeAES128, ivBytes)
                if ivStatus != errSecSuccess
                {
                    setupSuccess = false
                }
            }
            
            if (setupSuccess)
            {
                var numberOfBytesEncrypted : size_t = 0
                let size = clearTextData.count + kCCBlockSizeAES128
                var encrypted = Data.init(count: size)
                let cryptStatus = iv.withUnsafeBytes {ivBytes in
                    encrypted.withUnsafeMutableBytes {encryptedBytes in
                    clearTextData.withUnsafeBytes {clearTextBytes in
                        key.withUnsafeBytes {keyBytes in
                            CCCrypt(CCOperation(kCCEncrypt),
                                    CCAlgorithm(kCCAlgorithmAES),
                                    CCOptions(kCCOptionPKCS7Padding + kCCModeCBC),
                                    keyBytes,
                                    key.count,
                                    ivBytes,
                                    clearTextBytes,
                                    clearTextData.count,
                                    encryptedBytes,
                                    size,
                                    &numberOfBytesEncrypted)
                            }
                        }
                    }
                }
                if cryptStatus == Int32(kCCSuccess)
                {
                    encrypted.count = numberOfBytesEncrypted
                    outDictionary["EncryptionData"] = encrypted
                    outDictionary["EncryptionIV"] = iv
                    outDictionary["EncryptionSalt"] = salt
                }
            }
        
            return outDictionary;
        }

And, accordingly, the decryption function:

### Decryption

    class func decryp(fromDictionary dictionary : Dictionary<String, Data>, withPassword password : String) -> Data
        {
            var setupSuccess = true
            let encrypted = dictionary["EncryptionData"]
            let iv = dictionary["EncryptionIV"]
            let salt = dictionary["EncryptionSalt"]
            var key = Data(repeating:0, count:kCCKeySizeAES256)
            salt?.withUnsafeBytes { (saltBytes: UnsafePointer<UInt8>) -> Void in
                let passwordData = password.data(using:String.Encoding.utf8)!
                key.withUnsafeMutableBytes { (keyBytes : UnsafeMutablePointer<UInt8>) in
                    let derivationStatus = CCKeyDerivationPBKDF(CCPBKDFAlgorithm(kCCPBKDF2), password, passwordData.count, saltBytes, salt!.count, CCPseudoRandomAlgorithm(kCCPRFHmacAlgSHA512), 14271, keyBytes, key.count)
                    if derivationStatus != Int32(kCCSuccess)
                    {
                        setupSuccess = false
                    }
                }
            }
            
            var decryptSuccess = false
            let size = (encrypted?.count)! + kCCBlockSizeAES128
            var clearTextData = Data.init(count: size)
            if (setupSuccess)
            {
                var numberOfBytesDecrypted : size_t = 0
                let cryptStatus = iv?.withUnsafeBytes {ivBytes in
                    clearTextData.withUnsafeMutableBytes {clearTextBytes in
                    encrypted?.withUnsafeBytes {encryptedBytes in
                        key.withUnsafeBytes {keyBytes in
                            CCCrypt(CCOperation(kCCDecrypt),
                                    CCAlgorithm(kCCAlgorithmAES128),
                                    CCOptions(kCCOptionPKCS7Padding + kCCModeCBC),
                                    keyBytes,
                                    key.count,
                                    ivBytes,
                                    encryptedBytes,
                                    (encrypted?.count)!,
                                    clearTextBytes,
                                    size,
                                    &numberOfBytesDecrypted)
                            }
                        }
                    }
                }
                if cryptStatus! == Int32(kCCSuccess)
                {
                    clearTextData.count = numberOfBytesDecrypted
                    decryptSuccess = true
                }
            }
            
            return decryptSuccess ? clearTextData : Data.init(count: 0)
        }

To verify that these functions work and that the encryption/decryption is correct, you can use a simple example:

### Example

    class func encryptionTest()
        {
            let clearTextData = "some clear text to encrypt".data(using:String.Encoding.utf8)!
            let dictionary = encryptData(clearTextData, withPassword: "123456")
            let decrypted = decryp(fromDictionary: dictionary, withPassword: "123456")
            let decryptedString = String(data: decrypted, encoding: String.Encoding.utf8)
            print("decrypted cleartext result - ", decryptedString ?? "Error: Could not convert data to string")
        }

In this example, we package all the necessary information and return it as a dictionary, so that all the pieces can later be used to successfully decrypt data. This requires storing initialization vector (IV) and salt either in Keychain or on the server.

## Links

1. [https://developer.apple.com/](https://developer.apple.com/)
2. [https://en.wikipedia.org/wiki/Key\_stretching](https://en.wikipedia.org/wiki/Key_stretching)
3. [https://en.wikipedia.org/wiki/PBKDF2](https://en.wikipedia.org/wiki/PBKDF2)
4. [https://en.wikipedia.org/wiki/Padding\_(cryptography)#PKCS7](https://en.wikipedia.org/wiki/Padding_(cryptography)#PKCS7)