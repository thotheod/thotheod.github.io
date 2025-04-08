---
layout: post
title: "Encryption or hashing with Azure Key Vault"
date: 2025-04-07 15:18:00 +0300
categories: Development c-sharp
tags: azure app-innovation .net .net8 c-sharp
---
Cryptography is really a complex field. Among several cryptographic techniques, encryption and hashing are two fundamental concepts that serve different purposes in securing data.
Encryption is the process of converting plaintext into ciphertext using an algorithm and a key. The main goal of encryption is to protect the confidentiality of data, ensuring that only authorized parties can access it. Encryption can be symmetric (same key for encryption and decryption) or asymmetric (different keys for encryption and decryption). Hashing on the other hand, is a one-way function that takes an input and produces a fixed-size string of characters, which is typically a hash code. The main goal of hashing is to ensure data integrity, but also to store passwords securely. 

One of the key differences between encryption and hashing is that encryption is reversible (with the right key), while hashing is not. This means that encrypted data can be decrypted back to its original form, while hashed data cannot be reversed to retrieve the original input.

## Scenario
Let say I have some kind of subscription keys that I want to store securely. I want to achieve two things:
1. I want to be able to retrieve the keys in a secure way and distribute them back to subscribers.
2. I want to be able to verify the keys when they are used by subscribers.

In order to achieve this I can use a combination of encryption and hashing. I can encrypt the keys before storing them in a secure location and then hash the keys to create a unique identifier for each key. When a subscriber uses a key, I can verify it by comparing the hash of the provided key with the stored hash. To do that I can use Azure Key Vault, which is a cloud service that provides a secure storage solution for cryptographic keys and secrets. Azure Key Vault allows you to manage and control access to your keys and secrets, ensuring that they are protected from unauthorized access. In the next sections I will briefly discuss 
- Why I need both encryption and hashing
- How to use Azure Key Vault for encryption and hashing, for key management and access control
- Provide sample code to demonstrate the process


## Why I need encryption 
For my scenario to work I need encryption because:
- I need to store the keys securely
- I need a reversible process that allows to distribute the keys back to subscribers if required.


### What kind of encryption algorithm to use?
For my scenario I am using an RSA algorithm. In particular I am using RSA-OAEP algorithm, which provides a good balance between security and performance and on top of that it is supported by Azure Key Vault, making it a suitable choice for my scenario. [Optimal Asymmetric Encryption Padding (OAEP)](https://en.wikipedia.org/wiki/Optimal_asymmetric_encryption_padding) is a padding scheme that is used with [RSA encryption](https://en.wikipedia.org/wiki/RSA_cryptosystem) to provide additional security. It helps to prevent certain types of attacks, such as chosen ciphertext attacks, by adding randomness to the plaintext before encryption. This makes it more difficult for an attacker to decrypt the ciphertext without the private key. In plain text  OAEP adds an element of randomness which can be used to convert a deterministic encryption scheme (e.g., traditional RSA) into a probabilistic scheme. And while this is good for security, it also means that the same plaintext will produce different ciphertexts each time it is encrypted. This can be a problem if you need to compare ciphertexts or if you want to store them in a database. In this case, you can use a hash function to create a unique identifier for each ciphertext. This allows you to store the ciphertexts securely while still being able to compare them.


### Why encryption is not used for storing passwords?
Storing passwords securely is a critical aspect of application security. While encryption can be used to protect sensitive data, it is not the best approach for storing passwords. The main reason is that encryption is reversible, meaning that if an attacker gains access to the encryption key, they can decrypt the passwords and compromise user accounts. Instead, hashing is the preferred method for storing passwords because it is a one-way function that cannot be reversed. This means that even if an attacker gains access to the hashed passwords, they cannot retrieve the original passwords without using brute force or other methods. 

### How to use Azure Key Vault for encryption?
First of all we need to create a key. To create a key in Azure Key Vault, you can use the Azure SDK for .NET or the Azure Portal. 
In the portal you need to give it a name, and then select the correct key type, which in our case, is RSA. You can also select the key size, which is the length of the key in bits. A larger key size provides more security, but it also requires more processing power to encrypt and decrypt data. The recommended key size for RSA is at least 2048 bits. To be able to do that, you need to have the correct RBAC role, which is Key Vault Administrator or Key Vault Crypto Officer, which can perform any action on the keys of a key vault, except manage permissions. You can also use the Azure CLI or PowerShell to create a key in Azure Key Vault.

![Create a key with Azure Portal](/images/encryption-hashing/01-create-a-key.jpg){: width="600" height="180" }


You can of course create a key in Azure Key Vault with .NET. Here’s a simple example:


```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Keys;
using Azure.Security.KeyVault.Keys.Cryptography;

var keyVaultUrl = "https://<YourKeyVaultName>.vault.azure.net/";
var client = new KeyClient(new Uri(keyVaultUrl), new DefaultAzureCredential());

// Create a new key
var keyName = "myKey";
var key = await client.CreateKeyAsync(keyName, KeyType.Rsa);
```

Once you have created a key, you can use it to encrypt and decrypt data. To encrypt data with Azure Key Vault, you can use the `CryptographyClient` class, which provides methods for encrypting and decrypting data using the key stored in Azure Key Vault. Here’s an example of how to encrypt and decrypt data using Azure Key Vault. The full code sample, utilizing 3 azure functions, can be found [here.](https://github.com/thotheod/samples-blog/blob/main/kv-encryption/EncryptionSamples.cs)

```csharp

//  get the KeyClient
var client = SetupKeyClient();

KeyVaultKey key;
try
{
    // Try to retrieve the key using the key client.
    key = client.GetKey(keyName);
    _logger.LogInformation("Key '{KeyName}' already exists. Using the existing key.", keyName);
}
catch (Azure.RequestFailedException ex) when (ex.Status == 404)
{
    // If the key does not exist, create a new key.
    _logger.LogInformation("Key '{KeyName}' does not exist.", keyName);
    return new BadRequestObjectResult($"Key {keyName} is not found.");
}

// Create a new cryptography client using the same Key Vault or Managed HSM endpoint, service version,
// and options as the KeyClient created earlier.
var cryptoClient = client.GetCryptographyClient(key.Name, key.Properties.Version);

byte[] plaintext = Encoding.UTF8.GetBytes(text);

// encrypt the data using the algorithm RSAOAEP
EncryptResult encryptResult = cryptoClient.Encrypt(EncryptionAlgorithm.RsaOaep, plaintext);

// decrypt the encrypted data.
DecryptResult decryptResult = cryptoClient.Decrypt(EncryptionAlgorithm.RsaOaep, encryptResult.Ciphertext);
string decryptedText = Encoding.UTF8.GetString(decryptResult.Plaintext);

return new OkObjectResult($"EncryptWithKey Function Call result: " + Environment.NewLine +
    $"Encrypted Text: {Convert.ToBase64String(encryptResult.Ciphertext)}" + Environment.NewLine +
    $"Decrypted Text: {decryptedText}");
```

## Why I need hashing
Since I am using RSA OAEP for encryption, this can be a problem if you need to compare ciphertexts because the same plaintext will produce different ciphertexts each time it is encrypted. So I cannot fullfill my seconds requirement - to verify the keys when they are used by subscribers. In this case, I can use a hash function to create a unique identifier for each ciphertext. This allows me to store the ciphertexts securely while still being able to compare them. Hashing is a one-way function that takes an input and produces a fixed-size string of characters, which is typically a hash code. The main goal of hashing is to ensure data integrity, but also to store passwords securely. 

### What kind of hashing algorithm to use?
Azure Key Vault is primarily designed for storing and managing cryptographic keys, secrets, and certificates. It does not directly support hashing operations. However, you can use Azure Key Vault to securely store a key that can be used in conjunction with a hashing algorithm. We'll use [HMAC (Hash-based Message Authentication Code)](https://en.wikipedia.org/wiki/HMAC) for this purpose, which combines a cryptographic hash function with a secret key. HMAC is widely used for data integrity and authentication, and it is supported by Azure Key Vault. HMAC can be used with various hash functions, such as SHA-256 or SHA-512. The choice of hash function depends on your security requirements and performance considerations. SHA-256 is a good default choice for most applications, as it provides a good balance between security and performance.

### How to use Azure Key Vault for hashing?
To hash data using Azure Key Vault, you need first to retrieve a valid key from Azure Keyvault, as done previously. Then you need to get the modulus of the key, which is a byte array and use it with a hashing algorithm to compute the hash. Then use the byte array  of the key to create a new HMACSHA256(), as shown below. 

The full code sample, can be found [here.](https://github.com/thotheod/samples-blog/blob/main/kv-encryption/EncryptionSamples.cs) (_function: HashText_)

```csharp
...
var client = SetupKeyClient();
KeyVaultKey key = client.GetKey(keyName);
...
....
.....

public static string HashApiKey(string text, byte[] secretKey)
{
    using var hmac = new HMACSHA256(secretKey);
    byte[] textBytes = Encoding.UTF8.GetBytes(text);
    byte[] hashBytes = hmac.ComputeHash(textBytes);
    return Convert.ToBase64String(hashBytes);
}

```

## Conclusion
If you run the sample function app, you will see that there are three functions as shown below:
![Functions](/images/encryption-hashing/03-functions.jpg){: width="600" height="180" }

Once we have created a key (let say named: key01) I can call the HashText function to hash a 'Text' with the given key. If the text value is a GUID, as shown below, I get a specific hash value any time I call the function with the same key. 

![Functions](/images/encryption-hashing/04-hash.jpg)



If I change just one letter of the text (GUID) I get a totally different hash value, as demonstrated below. 

![Functions](/images/encryption-hashing/04-hash-new-value.jpg)


However, if I call the EncryptWithKey function with the same text value, I get a different encrypted value each time I call the function with the same key. This is because of the OAEP algorithm, which adds an element of randomness to the encryption process. Moreover, in order to be able to see the cipher text, I need to convert it to a base64 string, because the encrypted value is a byte array. This ensures that the encrypted data can be safely transmitted or stored as a string format. 

![Functions](/images/encryption-hashing/05-encrypt.jpg)


## References

- [Encryption sample](https://github.com/thotheod/samples-blog/blob/main/kv-encryption/EncryptionSamples.cs)
- [RSA encryption](https://en.wikipedia.org/wiki/RSA_cryptosystem)
- [Optimal Asymmetric Encryption Padding (OAEP)](https://en.wikipedia.org/wiki/Optimal_asymmetric_encryption_padding) 
- [Azure Key Vault key client library for .NET - version 4.7.0](https://learn.microsoft.com/en-us/dotnet/api/overview/azure/security.keyvault.keys-readme?view=azure-dotnet#encrypt-and-decrypt)
- [Encrypt and Decrypt blob storage data with Azure Key Vault](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-encrypt-decrypt-blobs-key-vault?tabs=roles-azure-portal%2Cpackages-dotnetcli)
- [HMAC](https://en.wikipedia.org/wiki/HMAC)
