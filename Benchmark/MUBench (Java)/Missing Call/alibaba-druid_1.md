## Repository

URL: https://github.com/alibaba/druid.git

Commit: e10f2849d046265bf17360ab4aa9eb60fd3ab8de~1

## API

init

	https://github.com/openjdk/jdk/blob/3c6ffcadfec42c544c9b0d4188e50135f608b9db/src/java.base/share/classes/javax/crypto/Cipher.java#L1245

## Caller

```java
package com.alibaba.druid.filter.config;

public class ConfigTools {
	private static final String DEFAULT_PRIVATE_KEY_STRING = "MIIBVAIBADANBgkqhkiG9w0BAQEFAASCAT4wggE6AgEAAkEAocbCrurZGbC5GArEHKlAfDSZi7gFBnd4yxOt0rwTqKBFzGyhtQLu5PRKjEiOXVa95aeIIBJ6OhC2f8FjqFUpawIDAQABAkAPejKaBYHrwUqUEEOe8lpnB6lBAsQIUFnQI/vXU4MV+MhIzW0BLVZCiarIQqUXeOhThVWXKFt8GxCykrrUsQ6BAiEA4vMVxEHBovz1di3aozzFvSMdsjTcYRRo82hS5Ru2/OECIQC2fAPoXixVTVY7bNMeuxCP4954ZkXp7fEPDINCjcQDywIgcc8XLkkPcs3Jxk7uYofaXaPbg39wuJpEmzPIxi3k0OECIGubmdpOnin3HuCP/bbjbJLNNoUdGiEmFL5hDI4UdwAdAiEAtcAwbm08bKN7pwwvyqaCBC//VnEWaq39DCzxr+Z2EIk=";
	public static final String DEFAULT_PUBLIC_KEY_STRING = "MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKHGwq7q2RmwuRgKxBypQHw0mYu4BQZ3eMsTrdK8E6igRcxsobUC7uT0SoxIjl1WveWniCASejoQtn/BY6hVKWsCAwEAAQ==";
	
	public static String decrypt(PublicKey publicKey, String cipherText)
			throws Exception {
		Cipher cipher = Cipher.getInstance("RSA");
		try {
			cipher.init(Cipher.DECRYPT_MODE, publicKey);
		} catch (InvalidKeyException e) {
			// for ibm jdk
			RSAPublicKey rsaPublicKey = (RSAPublicKey) publicKey;
			RSAPrivateKeySpec spec = new RSAPrivateKeySpec(rsaPublicKey.getModulus(), rsaPublicKey.getPublicExponent());
			Key fakePublicKey = KeyFactory.getInstance("RSA").generatePrivate(spec);
			cipher.init(Cipher.DECRYPT_MODE, fakePublicKey);
		}
		
		if (cipherText == null || cipherText.length() == 0) {
			return cipherText;
		}

		byte[] cipherBytes = Base64.base64ToByteArray(cipherText);
		byte[] plainBytes = cipher.doFinal(cipherBytes);

		return new String(plainBytes);
	}
}

```

## Description

An instance of Cipher is used twice (the init() method is called again), which is an invalid operation.


## Rule

Missing Call

## Response

### Confidence

95

### Types

Missing Exception Handling

### Crash

True

### Snippet

```java
try {
			cipher.init(Cipher.DECRYPT_MODE, publicKey);
		} catch (InvalidKeyException e) {
			// for ibm jdk
			RSAPublicKey rsaPublicKey = (RSAPublicKey) publicKey;
			RSAPrivateKeySpec spec = new RSAPrivateKeySpec(rsaPublicKey.getModulus(), rsaPublicKey.getPublicExponent());
			Key fakePublicKey = KeyFactory.getInstance("RSA").generatePrivate(spec);
			cipher.init(Cipher.DECRYPT_MODE, fakePublicKey);
		}
```

### Fix

```java
try {
			cipher.init(Cipher.DECRYPT_MODE, publicKey);
		} catch (InvalidKeyException e) {
			// for ibm jdk if publicKey does not work, try with specification
			RSAPublicKey rsaPublicKey = (RSAPublicKey) publicKey;
			RSAPrivateKeySpec spec = new RSAPrivateKeySpec(rsaPublicKey.getModulus(), rsaPublicKey.getPublicExponent());
			Key fakePublicKey = KeyFactory.getInstance("RSA").generatePrivate(spec);
			try {
				cipher.init(Cipher.DECRYPT_MODE, fakePublicKey);
			} catch (InvalidKeyException e) {
				throw new Exception("Invalid key for cipher initialization!", e);
			}
		}
```

### Description

In the catch block, the method 'init' is called on the 'cipher' object, but Exception Handling is not done properly. If, for some reason, the publicKey provided is not applicable or has a wrong format, the 'init' method call will throw an InvalidKeyException. This exception is not being caught around the 'init' method call, which could potentially lead to a crash.

## Review

Found the expected misuse? **False**

