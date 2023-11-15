## Repository

URL: https://github.com/Rellynn/Minecraft-Launcher

Commit: e62d1bbe1fed39a6897b8b0f294e50fde1c146b9

## API

javax.crypto.Cipher

## Caller

```java
package net.minecraft.launcher.authentication;

import java.io.DataInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Random;

import javax.crypto.Cipher;
import javax.crypto.CipherInputStream;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.spec.PBEParameterSpec;
import javax.swing.SwingUtilities;

import net.minecraft.launcher.Launcher;
import net.minecraft.launcher.events.AuthenticationChangedListener;

import org.apache.commons.lang3.StringUtils;

public abstract class BaseAuthenticationService implements AuthenticationService {
    private static final String LEGACY_LASTLOGIN_PASSWORD = "passwordfile";
    private static final int LEGACY_LASTLOGIN_SEED = 43287234;

    private static Cipher getCipher(final int mode, final String password) throws Exception {
        final Random random = new Random(43287234L);
        final byte[] salt = new byte[8];
        random.nextBytes(salt);
        final PBEParameterSpec pbeParamSpec = new PBEParameterSpec(salt, 5);

        final SecretKey pbeKey = SecretKeyFactory.getInstance("PBEWithMD5AndDES").generateSecret(new PBEKeySpec(password.toCharArray()));
        final Cipher cipher = Cipher.getInstance("PBEWithMD5AndDES");
        cipher.init(mode, pbeKey, pbeParamSpec);
        return cipher;
    }
}
```

## Description

Uses PBEWithMD5AndDES which is considered unsafe.


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
final Cipher cipher = Cipher.getInstance("PBEWithMD5AndDES");
        cipher.init(mode, pbeKey, pbeParamSpec);
```

### Fix

```java
final Cipher cipher = Cipher.getInstance("PBEWithMD5AndDES");
        try {
            cipher.init(mode, pbeKey, pbeParamSpec); 
        } catch (InvalidKeyException | InvalidAlgorithmParameterException e) {
            // Handle exception here.
        }
```

### Description

Cipher init method throws checked exceptions (InvalidKeyException and InvalidAlgorithmParameterException) that must be caught or declared to be thrown. In the given code, these exceptions are not handled, which could lead to the statement failing silently.

## Review

Found the expected misuse? **False**

