## Repository

URL: https://github.com/hoverruan/weiboclient4j.git

Commit: 6ca0c73f77f10d8ec1fed2c768c719dcf9ea7fc5

## API

java.lang.Long

## Caller

```java
package weiboclient4j.params;

import static weiboclient4j.utils.StringUtils.join;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

public class Cid extends LongParam {
    public Cid(long value) {
        super(value);
    }

    public Cid(String value) {
        this(Long.parseLong(value));
    }
}

```

## Description

`Cid(String)` calls `java.lang.long.parseLong` without first checking whether the argument parses.
This lead to an uncaught `NumberFormatException`: [Oracle Java 7 API specification](http://docs.oracle.com/javase/7/docs/api/java/lang/Long.html#parseLong%28java.lang.String,%20int%29).

This pull request adds a check with a  more helpful exception message.


## Rule

Missing Exception Handling

## Response

### Confidence

95

### Types

Missing Exception Handling

### Crash

True

### Snippet

```java
public Cid(String value) {
    this(Long.parseLong(value));
}
```

### Fix

```java
public Cid(String value) {
    try {
        this(Long.parseLong(value));
    } catch (NumberFormatException e) {
        throw new IllegalArgumentException("Invalid long number", e);
    }
}
```

### Description

The `Long.parseLong` API is misused here. This API throws a `NumberFormatException` when it fails to parse a string into a long, but there's no exception handling in this method. Therefore when a string that cannot be parsed into a long is passed to the method `Cid(String value)`, it will crash the program.

## Review

Found the expected misuse? **True**

