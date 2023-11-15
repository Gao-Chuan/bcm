## Repository

URL: https://github.com/apache/gora.git

Commit: bb09d8916736e3e572eef2cb5e79fe11c80ad1de

## API

DataOutputStream

    https://github.com/openjdk/jdk/blob/3c6ffcadfec42c544c9b0d4188e50135f608b9db/src/java.base/share/classes/java/io/DataOutputStream.java#L69

## Caller

```java
package org.apache.gora.accumulo.store;

public class PartitionTest {
  private static Encoder encoder = new SignedBinaryEncoder();

  static long encl(long l) {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    DataOutputStream dos = new DataOutputStream(baos);
    try {
      dos.writeLong(l);
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
    return encoder.decodeLong(baos.toByteArray());
  }
}

```

## Description

When a DataOutputStream instance wraps an underlying ByteArrayOutputStream instance,
it is recommended to flush or close the DataOutputStream before invoking the underlying instances's toByteArray(). Also, it is a good practice to call flush/close explicitly as mentioned for example [here](http://stackoverflow.com/questions/2984538/how-to-use-bytearrayoutputstream-and-dataoutputstream-simultaneously-java).
This pull request add a flush method before toByteArray.


## Rule

Missing Call

## Response

### Confidence

95

### Types

Missing Call

### Crash

False

### Snippet

```java
DataOutputStream dos = new DataOutputStream(baos);
try {
dos.writeLong(l);
} catch (IOException e) {
throw new RuntimeException(e);
}
```

### Fix

```java
DataOutputStream dos = null;
try {
dos = new DataOutputStream(baos);
dos.writeLong(l);
} catch (IOException e) {
throw new RuntimeException(e);
} finally {
if (dos != null) {
try {
dos.close();
} catch (IOException e) {
// ignore
}
}
}
```

### Description

This code opens a DataOutputStream, writes to it, and then loses its reference, but never calls close(). Even if an Exception is thrown, close() is not called on the stream in a finally block. This can result in a resource leak, as the underlying system resource (here, a ByteArrayOutputStream) may remain open after the method exits.

## Review

Found the expected misuse? **False**

