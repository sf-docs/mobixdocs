# A readable keystore, protected by a weak password, with private keys

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
        <td>Detection method:<strong> DAST, KEY INFORMATION</strong></td>
      </tr>
    </tbody>
</table>
## Description

An application uses a readable keystore, protected by a weak password, with private keys. This can lead to a forgery of key information. An encryption key must not be stored in a location with public access.

When using cryptographic operations on a device, you need to ensure maximum security of the main secret in such operations - the encryption key. When using asymmetric encryption, you need to store the private key securely; for symmetric algorithms, though, you need to protect the key that is used both for encryption and decryption of sensitive information.

The most secure option, without a doubt, is to store keys in Keychain.

Compromising key information in an application could be catastrophic depending on where in the application this information is used — starting with decryption of files or traffic, to compromising a private key that functions as a signature of the application.

## Recommendations

### Adding a new key into Keychain

    import Foundation
    import Security
    func computeSymmetricKey() -> String? {
        var keyData = Data(count: 32) // 32 bytes === 256 bits
        let result = keyData.withUnsafeMutableBytes {
            (mutableBytes: UnsafeMutablePointer) -> Int32 in
            SecRandomCopyBytes(kSecRandomDefault, keyData.count, mutableBytes)
        }
        if result == errSecSuccess {
            return keyData.base64EncodedString()
        } else {
            return nil
        }
    }
    let secretKey = computeSymmetricKey()

### Saving a new key in Keychain

    enum KeychainErrors:Error {
        case COULDNOTINSERT
        case COULDNOTREAD
    }
    func store (key: String, withTag: String) throws {
        let fromKey = key.data(using: .utf8)!
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassKey,
            kSecAttrApplicationTag as String: withTag,
            kSecValueRef as String: fromKey
        ]
        
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else { throw KeychainErrors.COULDNOTINSERT }
    }
    do {
        try store(key: secretKey!, withTag: "com.myapp.keys.localStore")
    } catch {
        print(error)
    }

### Reading a key from Keychain

    func read (tag: String) throws -> CFTypeRef {
        let readQuery: [String: Any] = [
            kSecClass as String: kSecClassKey,
            kSecAttrApplicationTag as String: tag,
            kSecAttrKeyType as String: kSecAttrKeyTypeRSA,
            kSecReturnRef as String: true
        ]
        
        var item: CFTypeRef?
        let readStatus = SecItemCopyMatching(readQuery as CFDictionary, &item)
        guard readStatus == errSecSuccess else { throw KeychainErrors.COULDNOTREAD }
        
        return item!
    }
    do {
        let keyReadFromKeychain = try read(tag: "com.myapp.keys.localStore")
        print(keyReadFromKeychain)
    } catch {
        print(error)
    }

### Using a key for encryption and decryption

    let algorithm: SecKeyAlgorithm = .rsaEncryptionOAEPSHA256
    let plainText = "this is our golden secret. Encrypt it!"
    var error: Unmanaged?
    guard let cipherText = SecKeyCreateEncryptedData(secretKey as! SecKey, algorithm,
        plainText as! CFData, &error) as Data? else {
            throw error!.takeRetainedValue() as Error
        }
    Отображение сертификатов
    func showCertificateInfo() -> String {
        var resultString = "--- Certificates in Keychain ---\n"
        var outputACert = false
        let query = [kSecMatchLimit: kSecMatchLimitAll,
                    kSecReturnRef: true,
                    kSecClass: kSecClassCertificate] as CFDictionary
        var result: CFTypeRef?
        let resultCode = SecItemCopyMatching(query, &result)
        if resultCode == errSecSuccess {
            if CFArrayGetTypeID() == CFGetTypeID(result) {
                let array = (result as? NSArray) as? [SecCertificate]
                array?.forEach { (item) in
                    resultString += self.displayCertificate(item)
                    outputACert = true
                }
            } else {
                // swiftlint:disable force_cast
                resultString += self.displayCertificate(result as! SecCertificate)
                // swiftlint:enable force_cast
                outputACert = true
            }
        }
        if !outputACert {
            resultString += "None\n"
        }
        resultString += "-------------------------------"
        return resultString
    }

## Links

1. [https://developer.apple.com/](https://developer.apple.com/documentation/security/certificate_key_and_trust_services/certificates/storing_a_certificate_in_the_keychain)
2. [CertificateSDK](https://github.com/jamf/CertificateSDK/blob/main/Certificate%20SDK%20Sample%20App/KeychainHandler.swift)
3. [Certificates](https://developer.apple.com/documentation/security/certificate_key_and_trust_services/certificates)
4. [Keychain Services](https://www.raywenderlich.com/9240-keychain-services-api-tutorial-for-passwords-in-swift)
5. [Basic iOS Security:](https://www.raywenderlich.com/129-basic-ios-security-keychain-and-hashing)[ ](https://www.raywenderlich.com/129-basic-ios-security-keychain-and-hashing)[Keychain and Hashing](https://www.raywenderlich.com/129-basic-ios-security-keychain-and-hashing)