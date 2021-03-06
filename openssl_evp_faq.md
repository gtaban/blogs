EVP = envelope 

# Public Key Algorithms

## EVP_PKEY_*

Uses a public key algorithm.
Uses EVP_PKEY_CTX

- EVP_PKEY_encrypt / decrypt
- EVP_PKEY_sign / verify: Does not hash data to be signed, so normally uses to sign digests. For normal signing, use EVP_DigestSign. https://wiki.openssl.org/index.php/Manual:EVP_PKEY_sign(3)
- EVP_PKEY_keygen

## EVP_*

Uses EVP_CIPHER_CTX

- EVP_Sign / Verify: Older APIs for sign/verify. Used for both HMAC and RSA
- EVP_Seal / Open: RSA+AES encryption
- EVP_DigestSign / DigestVerify: New APIs for sign/verify
- EVP_Digest: APIs used for hashing, example: https://www.openssl.org/docs/man1.0.2/crypto/EVP_DigestInit_ex.html
