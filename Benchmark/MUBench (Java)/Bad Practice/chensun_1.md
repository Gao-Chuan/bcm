## Repository

URL: https://github.com/ms969/ChenSun

Commit: cf23b99a0c78596b5dc5bb25691736c9abd0a84d

## API

javax.crypto.Cipher

## Caller

```java
package server;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;
import javax.crypto.spec.IvParameterSpec;

import crypto.CryptoUtil;
import crypto.Hash;

public class generateChecksumPostsAndReplies {
	
	public static String decAndCheck = "RuK3LVd6bQA=bDFM2Quk4LU=";

	/**
	 * @param args
	 */
	public static void main(String[] args) throws Exception{

		/*make decoder*/
		DESKeySpec keySpec = new DESKeySpec(CryptoUtil.decode("92LlYoVU1hU="));
		SecretKeyFactory factory = SecretKeyFactory.getInstance("DES");
		SecretKey key = factory.generateSecret(keySpec);
		
		byte[] iv = CryptoUtil.decode(decAndCheck.substring(0, 12));
		Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
		IvParameterSpec ivSpec = new IvParameterSpec(iv);
		c.init(Cipher.DECRYPT_MODE, key, ivSpec);
		
		byte[] content = c.doFinal(CryptoUtil.decode(decAndCheck.substring(12)));
		System.out.println(new String(content, "UTF8"));
		byte[] checksum = Hash.generateChecksum(content);
		System.out.println("\"" + CryptoUtil.encode(checksum) + "\"");
		
	}

}

```

## Description

Uses DES which is considered unsafe.


## Rule

Bad Practice

## Response

### Confidence

75

### Types

Bad Practice

### Crash

False

### Snippet

```java
Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
```

### Fix

```java
Cipher c = Cipher.getInstance("AES/CBC/PKCS5Padding");
```

### Description

The current cipher for data decryption is DES (Data Encryption Standard) which is considered insecure and deprecated because it uses 56-bit key size which can be brute-forced in a reasonable amount of time. In addition, it's vulnerable to many attacks (like F-N Attack, Rainbow Table Attack etc). It's recommended to replace with AES (Advanced Encryption Standard), which has 128, 192, or 256 bit key sizes. AES is resilient against known cryptographic vulnerabilities and it's the current encryption standard in the industry.

## Review

Found the expected misuse? **True**

