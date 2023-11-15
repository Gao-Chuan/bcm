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

Uses a non-random key in CBC mode.


## Rule

Bad Practice

## Response

### Confidence

90

### Types

Missing Null Check

### Crash

False

### Snippet

```java
Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
IvParameterSpec ivSpec = new IvParameterSpec(iv);
c.init(Cipher.DECRYPT_MODE, key, ivSpec);
```

### Fix

```java
Cipher c = Cipher.getInstance("DES/CBC/PKCS5Padding");
IvParameterSpec ivSpec = new IvParameterSpec(iv);
if(c != null){
c.init(Cipher.DECRYPT_MODE, key, ivSpec);
} else {
System.err.println("Fail to get Cipher instance with DES/CBC/PKCS5Padding algorithm");
}
```

### Description

The 'Cipher' object is not being checked for null after creation. This could cause a NullPointerException during runtime. The preferable way is to first check if the object is not null, before using it.

## Review

Found the expected misuse? **False**

