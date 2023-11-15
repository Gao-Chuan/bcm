## Repository

URL: https://github.com/MUBench/battleforge.git

Commit: 213e25632fbbdfc69eb01ddb8e4045db9c8025fc

## API

java.lang.String

## Caller

```java
package de.battleforge.util;

public class BFProperties {

    private static final String UNKNOWN = "unkown";

    private static final String REGENERATION_WARNING = "DO NOT EDIT!!! File will be regenerated!";

    public enum BFProps {
        PHP_CONNECTION, LOGIN_USER, LOGIN_PASSWORD, USE_PROXY, USE_SYSTEM_PROXY, PROXY_SERVER, PROXY_PORT, PROXY_USER, PROXY_PASSWORD, SHOW_SPLASHSCREEN, THEME, LANGUAGE, COUNTRY, PLAY_VOICE, PLAY_MUSIC, PLAY_SOUND, IRC_SERVER, IRC_PORT, IRC_CHANNEL, IRC_NICKNAME, IRC_PASSWORD, IRC_USER, IRC_NAME, BROWSER, VERSION, DOC_DIR, LOGFILE

    } // end of enum BFProps

    /**
     * Our logger.
     */
    private static Logger sLogger = Logger.getLogger(BFProperties.class);

    /**
     * Properties
     */
    private static Properties sProperties = new Properties();

    /**
     * Our file in which the user-properties are stored.
     */
    private static File sUserPropertyFile;

    /**
     * Static initializer.
     */
    static {
        try {
            sProperties.load(BFProperties.class.getClassLoader().getResourceAsStream("battleforge.properties"));

        } catch (IOException ioe) {
            sLogger.error("IO-Exception occured", ioe);

        } // try
    } // static

    public static void setProperty(BFProps key, String value, boolean store) {
        String strKey = key.name().toLowerCase();

        String prop = sProperties.getProperty(strKey);

        if ((prop == null) || !prop.equals(value)) {
            if (BFProps.PROXY_PASSWORD.equals(key)) {
                try {
                    Cipher c = Cipher.getInstance("AES");

                    byte[] secretKey = new byte[16];
                    
                    secretKey[0]=78;  
                    secretKey[1]=-44; 
                    secretKey[2]=-10;
                    secretKey[3]=-42;
                    secretKey[4]=-44;
                    secretKey[5]=106;
                    secretKey[6]=-30;
                    secretKey[7]=-1;
                    secretKey[8]=-104;
                    secretKey[9]=105;
                    secretKey[10]=24;
                    secretKey[11]=-103;
                    secretKey[12]=-9;
                    secretKey[13]=-81;
                    secretKey[14]=-32;
                    secretKey[15]=95;

                    Key k = new SecretKeySpec(secretKey, "AES");
                    c.init(Cipher.ENCRYPT_MODE, k);

                    value = new String(c.doFinal(value.getBytes()));

                } catch (NoSuchAlgorithmException e1) {
                    e1.printStackTrace();

                } catch (NoSuchPaddingException e1) {
                    e1.printStackTrace();

                } catch (InvalidKeyException e1) {
                    // TODO Auto-generated catch block
                    e1.printStackTrace();

                } catch (IllegalBlockSizeException e1) {
                    // TODO Auto-generated catch block
                    e1.printStackTrace();

                } catch (BadPaddingException e1) {
                    // TODO Auto-generated catch block
                    e1.printStackTrace();

                }
            }

            sProperties.setProperty(strKey, value);

            if (store) {
                Properties props = new Properties();

                try {
                    FileInputStream fIn = new FileInputStream(sUserPropertyFile);

                    props.load(fIn);

                    fIn.close();

                    props.setProperty(strKey, value);

                    FileOutputStream fOut = new FileOutputStream(sUserPropertyFile);

                    props.store(fOut, REGENERATION_WARNING);

                    fOut.close();

                } catch (IOException e) {
                    sLogger.error("Exception occured", e);

                } // try
            } // if

            updateSystemProperties();

        } // if
    } 
} // end of class BFProperties

```

## Description

Text is converted to bytes for encoding without an explicit encoding.


## Rule

Bad Practice

## Response

### Confidence

80

### Types

Bad Practice

### Crash

True

### Snippet

```java
value = new String(c.doFinal(value.getBytes()));
```

### Fix

```java
value = new String(c.doFinal(value.getBytes()), StandardCharsets.UTF_8);
```

### Description

The line 'value = new String(c.doFinal(value.getBytes()));' contains a potential bug. The conversion of byte[] to String depends on the system's default charset and hence maybe platform dependent. It might lead to strange behaviors in case of cross-platform usage. The fix is to provide a charset for byte to string conversion.

## Review

Found the expected misuse? **True**

