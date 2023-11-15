## Repository

URL: https://github.com/alibaba/druid.git

Commit: e10f2849d046265bf17360ab4aa9eb60fd3ab8de~1

## API

javax.crypto.Cipher

## Caller

```java
package com.alibaba.druid.filter.config;

public class ConfigTools {

	private static final String DEFAULT_PRIVATE_KEY_STRING = "MIIBVAIBADANBgkqhkiG9w0BAQEFAASCAT4wggE6AgEAAkEAocbCrurZGbC5GArEHKlAfDSZi7gFBnd4yxOt0rwTqKBFzGyhtQLu5PRKjEiOXVa95aeIIBJ6OhC2f8FjqFUpawIDAQABAkAPejKaBYHrwUqUEEOe8lpnB6lBAsQIUFnQI/vXU4MV+MhIzW0BLVZCiarIQqUXeOhThVWXKFt8GxCykrrUsQ6BAiEA4vMVxEHBovz1di3aozzFvSMdsjTcYRRo82hS5Ru2/OECIQC2fAPoXixVTVY7bNMeuxCP4954ZkXp7fEPDINCjcQDywIgcc8XLkkPcs3Jxk7uYofaXaPbg39wuJpEmzPIxi3k0OECIGubmdpOnin3HuCP/bbjbJLNNoUdGiEmFL5hDI4UdwAdAiEAtcAwbm08bKN7pwwvyqaCBC//VnEWaq39DCzxr+Z2EIk=";
	public static final String DEFAULT_PUBLIC_KEY_STRING = "MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKHGwq7q2RmwuRgKxBypQHw0mYu4BQZ3eMsTrdK8E6igRcxsobUC7uT0SoxIjl1WveWniCASejoQtn/BY6hVKWsCAwEAAQ==";
	public static String encrypt(byte[] keyBytes, String plainText)
			throws Exception {
		PKCS8EncodedKeySpec spec = new PKCS8EncodedKeySpec(keyBytes);
		KeyFactory factory = KeyFactory.getInstance("RSA");
		PrivateKey privateKey = factory.generatePrivate(spec);
		Cipher cipher = Cipher.getInstance("RSA");
		cipher.init(Cipher.ENCRYPT_MODE, privateKey);

		byte[] encryptedBytes = cipher.doFinal(plainText.getBytes("UTF-8"));
		String encryptedString = Base64.byteArrayToBase64(encryptedBytes);

		return encryptedString;
	}
}

```

## Description

A call to Cipher.init() may throw an InvalidKeyException.


## Rule

Missing Exception Handling

## Response

### Confidence

80

### Types

Bad Practice

### Crash

False

### Snippet

```java
['Cipher cipher = Cipher.getInstance("RSA");', '', 'cipher.init(Cipher.ENCRYPT_MODE, privateKey);']
```

### Fix

```java
[Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");]
```

### Description

The initialization of the Cipher instance does not specify any encryption mode or padding scheme. This lack of specification could be a security issue as provider-dependent defaults for mode and padding scheme would be used. An attacker could exploit such behavior for his advantage. Hence it is recommended to always specify a precise algorithm which includes mode and padding scheme, like 'RSA/ECB/PKCS1Padding'.

## Review

Found the expected misuse? **False**

