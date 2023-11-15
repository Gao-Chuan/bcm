## Repository

URL: https://github.com/asterisk-java/asterisk-java.git

Commit: 304421c261da68df03ad2fb96683241c8df12c0a^1

## API

java.net.URLDecoder

## Caller

```java
package org.asteriskjava.manager.event;

public class AsyncAgiEvent extends ResponseEvent
{
    static final long serialVersionUID = 0L;

    public static final String SUB_EVENT_START = "Start";
    public static final String SUB_EVENT_EXEC = "Exec";
    public static final String SUB_EVENT_END = "End";

    private String channel;
    private String subEvent;
    private String commandId;
    private String result;
    private String env;
    private String uniqueId;

    private List<String> decode(String s)
    {
        final List<String> result = new ArrayList<String>();

        if (s == null)
        {
            return result;
        }

        try
        {
            final String decodedString = URLDecoder.decode(s, "ISO-8859-1");
            result.addAll(Arrays.asList(decodedString.split("\n")));
        }
        catch (UnsupportedEncodingException e)
        {
            // won't happen as JDK ships with ISO-8859-1
            throw new RuntimeException("This JDK does not support ISO-8859-1 encoding", e);
        }

        return result;
    }
}

```

## Description

In line 206 of AsyncAgiEvent, it uses ISO-8859-1 encoding scheme but according to [Oracle Java 7 API specification](http://docs.oracle.com/javase/7/docs/api/java/net/URLDecoder.html#decode%28java.lang.String,%20java.lang.String%29), the World Wide Web Consortium Recommendation states that UTF-8 should be used. Not doing so may introduce incompatibilities. 
This pull request adds a fix by using UTF-8 encoding.


## Rule

Bad Practice

## Response

### Confidence

100

### Types

Bad Practice

### Crash

False

### Snippet

```java
throw new RuntimeException("This JDK does not support ISO-8859-1 encoding", e);
```

### Fix

```java
// Handle the exception in a proper way instead of throwing a Runtime error
    try
    {
        final String decodedString = URLDecoder.decode(s, "ISO-8859-1");
        result.addAll(Arrays.asList(decodedString.split("\n")));
    }
    catch (UnsupportedEncodingException e)
    {
        System.err.println("Failed to decode string: " + e);
    }
```

### Description

In the decode() function, if an UnsupportedEncodingException occurs, it currently throws a RuntimeException. Throwing RuntimeExceptions should be avoided and should instead log the exception.

## Review

Found the expected misuse? **False**

