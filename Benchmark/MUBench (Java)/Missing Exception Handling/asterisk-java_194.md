## Repository

URL: https://github.com/asterisk-java/asterisk-java.git

Commit: 41461b41309bf9f027a46f178cb777a1a94b8c3f^1

## API

java.lang.Long

https://github.com/openjdk/jdk/blob/3c6ffcadfec42c544c9b0d4188e50135f608b9db/src/java.base/share/classes/java/lang/Long.java#L75

## Caller

```java
package org.asteriskjava.manager.event;

public class RtcpReceivedEvent extends AbstractRtcpEvent
{
    private static final long serialVersionUID = 2L;

    /**
     * Payload identifier for a sender report.
     */
    public static final int PT_SENDER_REPORT = 200;

    /**
     * Payload identifier for a receiver report.
     */
    public static final int PT_RECEIVER_REPORT = 201;
    public static final int PT_H261_FUR = 192;

    private InetAddress fromAddress;
    private Integer fromPort;
    private Long pt;
    private Long receptionReports;
    private Long senderSsrc;
    private Long packetsLost;
    private Long highestSequence;
    private Long sequenceNumberCycles;
    private Double lastSr;
    private Double rtt;
    
    private String channel;
    private String language;    
    private String report0SequenceNumberCycles;
    private String ssrc;
    private String linkedId;
    private String report0lsr;
    private Long sentOctets;
    private String report0Sourcessrc;
    private Double report0dlsr;
    private String uniqueid;
    private Integer reportCount;
    private Integer report0CumulativeLost;
    private Integer report0FractionLost;
    private Integer report0iaJitter;
    private Integer report0HighestSequence;
    private InetAddress toAddress;
    private Integer toPort;
    private String sentntp;
    private Integer sentPackets;
    private Long sentrtp;
    private String accountCode;

    public void setPt(String ptString)
    {
        // Format is "PT: %d(%s)"
        if (ptString == null || ptString.length() == 0)
        {
            this.pt = null;
            return;
        }

        if (ptString.indexOf('(') > 0)
        {
            this.pt = Long.parseLong(ptString.substring(0, ptString.indexOf('(')));
        }
        else
        {
            this.pt = Long.parseLong(ptString);
        }
    }
}

```

## Description

`RtcpReceivedEvent.java` calls `java.lang.long.parseLong` without first checking whether the argument parses. This lead to an uncaught `NumberFormatException`:
[Oracle Java 7 API specification](http://docs.oracle.com/javase/7/docs/api/java/lang/Long.html#parseLong%28java.lang.String,%20int%29).
This pull request adds a check with a more helpful exception message. Kindly let me know if you want me to change the message.


## Rule

Missing Exception Handling

## Response

### Confidence

90

### Types

Missing Exception Handling

### Crash

Yes

### Snippet

```java
if (ptString.indexOf('(') > 0) {
 this.pt = Long.parseLong(ptString.substring(0, ptString.indexOf('(')));
 } else {
 this.pt = Long.parseLong(ptString);
 }
```

### Fix

```java
if (ptString.indexOf('(') > 0) {
 try {
 this.pt = Long.parseLong(ptString.substring(0, ptString.indexOf('(')));
 } catch (NumberFormatException e) {
 this.pt = null;
 } } else {
 try {
 this.pt = Long.parseLong(ptString);
 } catch (NumberFormatException e) {
 this.pt = null;
 } }
```

### Description

Here, the method setPt(String) is converting a String to a Long value with the Long.parseLong method which is provided by java.lang.Long API. But, it's risky as the passed String might not be able to be converted to a Long, which results in the NumberFormatException and leads to a crash. To fix this, you should catch the NumberFormatException and, in this case, set the variable to null, which is also your current behavior for empty string and null input.

## Review

Found the expected misuse? **True**

