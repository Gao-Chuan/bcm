## Repository

URL: https://github.com/jriecken/gae-java-mini-profiler.git

Commit: 80f3a59ebe7bfa02506ff53b63ebeaf63cfbf525

## API

java.lang.Long


https://github.com/openjdk/jdk/blob/3c6ffcadfec42c544c9b0d4188e50135f608b9db/src/java.base/share/classes/java/lang/Long.java#L75

## Caller

```java
package com.google.appengine.tools.appstats;

import java.util.*;

import com.google.appengine.api.memcache.MemcacheServiceFactory;
import com.google.appengine.tools.appstats.StatsProtos.*;

public class MiniProfilerAppstats
{
  public static Map<String, Object> getAppstatsDataFor(String appstatsId, Integer maxStackFrames)
  {
    Map<String, Object> appstatsMap = null;
    MemcacheWriter writer = new MemcacheWriter(null, MemcacheServiceFactory.getMemcacheService("__appstats__"));
    StatsProtos.RequestStatProto appstats = writer.getFull(Long.parseLong(appstatsId));
    if (appstats != null)
    {
      appstatsMap = new HashMap<String, Object>();
      appstatsMap.put("totalTime", appstats.getDurationMilliseconds());

      Map<String, Map<String, Object>> rpcInfoMap = new LinkedHashMap<String, Map<String, Object>>();
      for (AggregateRpcStatsProto rpcStat : appstats.getRpcStatsList())
      {
        Map<String, Object> rpcInfo = rpcInfoMap.get(rpcStat.getServiceCallName());
        if (rpcInfo == null)
        {
          rpcInfo = new LinkedHashMap<String, Object>();
          rpcInfoMap.put(rpcStat.getServiceCallName(), rpcInfo);
        }

        rpcInfo.put("totalCalls", rpcStat.getTotalAmountOfCalls());
        rpcInfo.put("totalTime", Long.valueOf(0));
      }

      List<Map<String, Object>> callInfoMap = new ArrayList<Map<String, Object>>();
      for (IndividualRpcStatsProto rpcStat : appstats.getIndividualStatsList())
      {
        // Update the total time for the RPC method
        Map<String, Object> rpcInfo = rpcInfoMap.get(rpcStat.getServiceCallName());
        rpcInfo.put("totalTime", ((Long) rpcInfo.get("totalTime")) + rpcStat.getDurationMilliseconds());

        // Get info about this specific call
        Map<String, Object> callInfo = new LinkedHashMap<String, Object>();
        callInfoMap.add(callInfo);
        callInfo.put("serviceCallName", rpcStat.getServiceCallName());
        callInfo.put("totalTime", rpcStat.getDurationMilliseconds());
        callInfo.put("startOffset", rpcStat.getStartOffsetMilliseconds());
        callInfo.put("request", truncate(rpcStat.getRequestDataSummary(), 100));
        callInfo.put("response", truncate(rpcStat.getResponseDataSummary(), 100));
        // Get the stack trace
        List<String> callStack = new ArrayList<String>();
        int i = 0;
        for (StackFrameProto frame : rpcStat.getCallStackList())
        {
          if (maxStackFrames != null && i == maxStackFrames)
          {
            break;
          }
          callStack.add(String.format("%s.%s:%d", frame.getClassOrFileName(), frame.getFunctionName(), frame.getLineNumber()));
          i++;
        }
        callInfo.put("callStack", callStack);
      }
      appstatsMap.put("rpcStats", !rpcInfoMap.isEmpty() ? rpcInfoMap : null);
      appstatsMap.put("rpcCalls", !callInfoMap.isEmpty() ? callInfoMap : null);
    }
    return appstatsMap;
  }
}

```

## Description

MiniProfilerAppstats.java calls `java.lang.Long.parseLong` without first
checking whether the argument parses. This lead to an uncaught `NumberFormateException`:
[Oracle Java 7 API specification](http://docs.oracle.com/javase/7/docs/api/java/lang/Long.html#parseLong%28java.lang.String,%20int%29).

This pull request adds a check and a test for this issue.


## Rule

Missing Exception Handling

## Response 0

### Confidence

95

### Types

API misuse

### Crash

False

### Snippet

```java
rpcInfo.put("totalTime", Long.valueOf(0));
```

### Fix

```java
// Correct use of the API Long.valueOf
rpcInfo.put("totalTime", Long.valueOf("0"));
```

### Description

The API Long.valueOf is used incorrectly. The method Long.valueOf expects a string argument but here it is given a number '0'. This could potentially lead to an unexpected behaviour.

## Response 1

### Confidence

90

### Types

Missing Null Check

### Crash

True

### Snippet

```java
rpcInfo.put("totalTime", ((Long) rpcInfo.get("totalTime")) + rpcStat.getDurationMilliseconds());
```

### Fix

```java
// Necessary null checks to prevent NullPointerException
Long totalTime = (Long) rpcInfo.get("totalTime");
// Ensure that totalTime is not null before trying to unbox it to a long
if (totalTime != null) {
  rpcInfo.put("totalTime", totalTime + rpcStat.getDurationMilliseconds());
}
```

### Description

There is potential for a NullPointerException to occur when the value associated with "totalTime" in the rpcInfo map is being unboxed to a long primitive type. If the rpcInfo map does not contain a value for "totalTime", or if that value is null, trying to unbox it to a long will throw a NullPointerException.

## Review

Found the expected misuse? **False**

