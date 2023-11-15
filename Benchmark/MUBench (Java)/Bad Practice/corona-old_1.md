## Repository

URL: https://github.com/cftdanny/corona.old

Commit: 0d0d18b7cde563c6098734cea7819d946a027211

## API

javax.crypto.Cipher

https://github.com/openjdk/jdk/blob/3c6ffcadfec42c544c9b0d4188e50135f608b9db/src/java.base/share/classes/javax/crypto/Cipher.java#L173

## Caller

```java
package com.corona.crypto;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

public class AESCypher implements Cypher {

	protected static final String ALGORITHM = "AES";

	private Cipher encryptor;

	private Cipher decryptor;
	
	private void setEncryptKey(final byte[] key) throws CypherException {
		
		SecretKey secretKey = new SecretKeySpec(key, ALGORITHM);
		try {
			this.encryptor = Cipher.getInstance(ALGORITHM);
			this.encryptor.init(Cipher.ENCRYPT_MODE, secretKey);
		} catch (Exception e) {
			throw new CypherException("Fail to create and initialize [{0}] cipher", e, ALGORITHM);
		}
	}
}

```

## Description

Uses AES with default mode and padding.  Since these values might change per crypto provider this is considered a bad practice.


## Rule

Bad Practice

## Response

### Confidence

85

### Types

Bad Practice

### Crash

False

### Snippet

```java
this.encryptor = Cipher.getInstance(ALGORITHM);\nthis.encryptor.init(Cipher.ENCRYPT_MODE, secretKey);
```

### Fix

```java
this.encryptor = Cipher.getInstance(ALGORITHM + "/ECB/PKCS5Padding");\nthis.encryptor.init(Cipher.ENCRYPT_MODE, secretKey);
```

### Description

In the provided code, the 'getInstance' API of the 'javax.crypto.Cipher' class has been used without specifying a block mode and padding. While the use of 'getInstance' API with solely the algorithm name doesn't necessarily cause a crash, it's considered a bad practice not to explicitly specify the block mode and padding since the actual configuration would then be dependent upon underlying JVM's default configuration and the code becomes less predictable. We recommend always specify both the cipher mode (e.g., ECB, CBC, etc.) and padding (e.g., NoPadding, PKCS5Padding, etc.) when using this API. We add ECBE/PKCS5Padding to the Cipher.getInstance to fix the issue.

## Review

Found the expected misuse? **True**

