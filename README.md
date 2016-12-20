# Cinnabar

Cinnabar is that mineral powder that gives this bright red tint to Asian name seals. It is also a (hopefully) portable piece of C code to generate and check RSA signatures.

Key features:
- Implements RSASSA-PKCS1-v1_5 with SHA-256 as hashing function
- Supports any key length > 496 bits
- Easy to integrate: just drop cinnabar.h and cinnabar.c in your project
- Should compile on any platform with a decent 32-bit or 64-bit C compiler
- Works on both little endian and big endian platforms

It is primarily intended for embedded systems where usual crypto libraries are not available, and for applications where you must compile the exact same source code on (possibly very) different platforms.

## Key generation

Signing requires a RSA private key. Verifying a signature requires the corresponding public key. Cinnabar accepts keys in the PEM format. You can easily generate such keys using openssl, which should be available on any Unix-like system.

You can generate a private key by typing the following command in a terminal:

``openssl genrsa -out private.pem 2048``

That generates a 2048-bit RSA pair and write them to a file named ``private.pem``. From this private key, you can then ask openssl to generate a public key:

``openssl rsa -in private.pem -outform PEM -pubout -out public.pem``

You can open these files in a text editor and check that the private key is enclosed between ``-----BEGIN RSA PRIVATE KEY-----`` and ``-----END RSA PRIVATE KEY-----`` lines, while the public key is enclosed between ``-----BEGIN PUBLIC KEY-----`` and ``-----END PUBLIC KEY-----`` lines.

Remember that your private key should remain, huh, well… secret. This means that you must generate it on your computer (or on the final user's computer) and never grab one on a random web site. A private key must never be sent over an insecure channel such as an email. It is also usually not the best idea to embed a private key in your source code: an executable file is not a secure place and moreover, if your key is uncovered after your application is deployed on thousands of computers, your would not be able to change it easily. The exact key management scheme depends on your application though.

## Usage

To be portable and not rely upon any specific file system or operating system, Cinnabar does not directly read PEM files. Instead, your code must load key content in memory by any mean that pertains to your application or use case, and then pass Cinnabar functions a pointer to this content.

### Signing

You compute the signature of a piece of data by calling the following function:

``CnbrStatus CnbrSignature(CNBR_SIGNATURE * signature, const void * document_data, size_t document_length, const char * pem_private_key)``

- ``signature`` is a pointer to a statically allocated ``CNBR_SIGNATURE`` structure that will receive signature data
- ``document_data`` is a pointer to the data to sign
- ``document_length`` is the length (in bytes) of the data to sign
- ``pem_private_key`` is a pointer to the content of an RSA private key PEM file

On success, the function returns ``CnbrSuccess`` and the ``CNBR_SIGNATURE`` structure contains signature data and length. In the event of an error, the function returns one of the error code defined in the ``CnbrStatus`` enumeration. Your original data is unchanged (note the parameter is defined as a pointer to ``const`` data).

The ``CNBR_SIGNATURE`` structure holds a pointer to the signature data. How you use this signature depends on your application and use case. For example, if your application is signing a mail, you can encode this signature to base64 and join it as an attachment to the mail.

Once you are done with the signature data, you must call ``CnbrEraseSignature`` to erase any sensitive information and free allocated memory.

### Verifying a signature

You verify a signature by calling the following function:

``CnbrStatus CnbrVerifySignature(const uint8_t * signature_data, size_t signature_length, const void * document_data, size_t document_length, const char * pem_public_key)``

- ``signature_data`` is a pointer to a signature previously generated by the ``CnbrSignature`` function (or by any other RSASSA-PKCS1-v1_5 compliant implementation)
- ``signature_length`` is the length (in bytes) of the signature
- ``document_data`` is a pointer to the data whose signature is being verified
- ``document_length`` is the length (in bytes) of the data
- ``pem_public_key`` is a pointer to the content of a public key PEM file

If the signature is valid, the function returns ``CnbrSuccess``. If an error occurred or if the signature is not valid, the function returns one of the error code defined in the ``CnbrStatus`` enumeration.

A valid signature ensures authentication, non-repudiation and integrity: it gives a strong indication that the message was created by a known sender, that the sender cannot deny having signed the message, and that the message was not altered afterwards.

## Sample code

Work in progress.
