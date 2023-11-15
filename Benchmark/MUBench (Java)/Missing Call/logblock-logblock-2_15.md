## Repository

URL: https://github.com/LogBlock/LogBlock-2.git

Commit: 40548aad551c3ec59a72433937afeaa40a052cb4

## API

java.io.DataOutputStream

## Caller

```java
package org.logblock.entry;

import org.junit.Assert;
import org.junit.Test;
import org.logblock.entry.blob.PaintingBlob;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.DataInputStream;
import java.io.DataOutputStream;

public class BlobTest
{

    @Test
    public void paintingTest() throws Exception
    {
        ByteArrayOutputStream byteOutput = new ByteArrayOutputStream();
        DataOutputStream outStream = new DataOutputStream(byteOutput);

        PaintingBlob blobOut = BlobEntry.create(1, PaintingBlob.class);
        blobOut.setArt("artistic");
        blobOut.setDirection((byte) 5);
        blobOut.write(outStream);

        DataInputStream inputStream = new DataInputStream(new ByteArrayInputStream(byteOutput.toByteArray()));
        PaintingBlob blobIn = BlobEntry.create(1, PaintingBlob.class);
        blobIn.read(inputStream);

        Assert.assertEquals(blobOut, blobIn);
    }
}

```

## Description

When a DataOutputStream instance wraps an underlying ByteArrayOutputStream instance,
it is recommended to flush or close the DataOutputStream before invoking the underlying instances's toByteArray().
Also, it is a good practice to call flush/close explicitly as mentioned for example [here](http://stackoverflow.com/questions/2984538/how-to-use-bytearrayoutputstream-and-dataoutputstream-simultaneously-java).
This pull request adds a close method before calling toByteArray().


## Rule

Missing Call

## Response

### Confidence

100

### Types

Other

### Crash

False

### Snippet

```java
DataOutputStream outStream = new DataOutputStream(byteOutput);
```

### Fix

```java
try (DataOutputStream outStream = new DataOutputStream(byteOutput)) { // other code }
```

### Description

The DataOutputStream is not closed after being used. This will lead to resource leakage problems and may influence the performance of the software. Always close the streams after using them to free up resources.

## Review

Found the expected misuse? **False**

