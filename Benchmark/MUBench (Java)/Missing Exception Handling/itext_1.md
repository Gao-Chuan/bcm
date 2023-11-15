## Repository

URL: https://github.com/itext/itextpdf.git

Commit: adda8eb38978eeea6e86aca64ae746754c103dae

## API

javax.crypto.Cipher

## Caller

```java
package com.itextpdf.text.pdf;

public class PdfPublicKeySecurityHandler {

    static final int SEED_LENGTH = 20;

    private ArrayList<PdfPublicKeyRecipient> recipients = null;

    private byte[] seed = new byte[SEED_LENGTH];

    private KeyTransRecipientInfo computeRecipientInfo(X509Certificate x509certificate, byte[] abyte0)
        throws GeneralSecurityException, IOException
    {
        ASN1InputStream asn1inputstream =
            new ASN1InputStream(new ByteArrayInputStream(x509certificate.getTBSCertificate()));
        TBSCertificateStructure tbscertificatestructure =
            TBSCertificateStructure.getInstance(asn1inputstream.readObject());
        AlgorithmIdentifier algorithmidentifier = tbscertificatestructure.getSubjectPublicKeyInfo().getAlgorithmId();
        IssuerAndSerialNumber issuerandserialnumber =
            new IssuerAndSerialNumber(
                tbscertificatestructure.getIssuer(),
                tbscertificatestructure.getSerialNumber().getValue());
        Cipher cipher = Cipher.getInstance(algorithmidentifier.getObjectId().getId());
        cipher.init(1, x509certificate);
        DEROctetString deroctetstring = new DEROctetString(cipher.doFinal(abyte0));
        RecipientIdentifier recipId = new RecipientIdentifier(issuerandserialnumber);
        return new KeyTransRecipientInfo( recipId, algorithmidentifier, deroctetstring);
    }
}

```

## Description

Cipher.init may throw InvalidKeyException.


## Rule

Missing Exception Handling

## Response

### Confidence

90

### Types

Other

### Crash

True

### Snippet

```java
Cipher cipher = Cipher.getInstance(algorithmidentifier.getObjectId().getId());
cipher.init(1, x509certificate);
```

### Fix

```java
Cipher cipher = Cipher.getInstance(algorithmidentifier.getObjectId().getId());
cipher.init(Cipher.ENCRYPT_MODE, x509certificate.getPublicKey());
```

### Description

In the computeRecipientInfo() method, the init method of the Cipher class is being called with an X509Certificate object passed in directly, however, only Key object is accepted. Thus, misuse of API occurs and it may cause a crash. The certificate's public key should be retrieved using getPublicKey() method and passed to Cipher's init() method.

## Review

Found the expected misuse? **False**

