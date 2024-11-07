[[Information Security]]
Cordoba
****

Digital signatures are the biggest feature introduced by public-key cryptography, as it can not be implemented with symmetric cryptography.

The purpose of digital signatures authenticity and integrity of a document. **The signatory claims ownership of the document**

1. Authority A — who wants to sign a document — will generate a hash of the document to sign (SHA-512), and encrypt this hash with their private key.
2. The authority now publishes the document along with the signature, so people can download both of them. 
3. Downloader generates the SHA-512 for the file he downloaded, and decrypt the signature with the authority's public key
4. If the file's hash matches the decrypted signature's hash, the file has been signed by Authority A.
*As long as no one has Authority A's private key, no one can imitate their signature.*


**Symmetric encryption protects two parties from a third one attempting to alter the message, but it does not protect the two parties against each other (big problem!)**
Digital signature satisfies those three properties:
- Authenticate the content (file hash encryption)
- Verifiable by third parties (public-key cryptography)
- Verify the author (only the authority possesses the private key that creates the valid signature)


****
## Direct Digital Signature

A direct digital signature scheme exclusively involves the sender (source) and receiver (destination) without intermediary parties.
1. **Public Key Requirement**: The destination (verifier) must already know the source’s (signer) public key for verification.
2. **Confidentiality**: Achieved by encrypting the message and its signature with a shared symmetric key.


**Order of operation: Sign First, Encrypt Second**
The signature is applied to the plaintext before encryption. This approach enables third-party verification of the signature without requiring access to the decryption key, which preserves confidentiality in dispute resolutions.


****
## Security Considerations

- **Private Key Security**: The scheme's security relies on the sender’s private key's integrity. A compromised private key allows an attacker to forge signatures.
- **Key Theft Claims**: A sender could deny sending a message by claiming key theft. To mitigate this, messages often include a **timestamp** and require prompt reporting of compromised keys to a central authority.
- **Digital Certificates**: To further reduce the risks of key misuse, digital certificates issued by Certificate Authorities (CAs) authenticate the ownership of public keys and support secure communication between parties.


****
## Threat Mitigations

- **Stolen Key Usage**: If an attacker uses a stolen private key, timestamps and prompt compromise reporting help detect unauthorised use.
- **Biometric Limitation**: Unlike physical signatures, anyone with access to the private key can generate a digital signature, which emphasises the need for secure key management and trusted verification methods.

