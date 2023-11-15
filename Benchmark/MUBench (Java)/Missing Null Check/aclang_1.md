## Repository

URL: https://git-wip-us.apache.org/repos/asf/commons-lang.git

Commit: 1fe5439baf32af2114958e3cfc3512bd72c84773~1

## API

org.apache.commons.lang.text.StrBuilder

## Caller

```java
package org.apache.commons.lang.text;

public class StrBuilder implements Cloneable {
    public StrBuilder appendFixedWidthPadLeft(Object obj, int width, char padChar) {
        if (width > 0) {
            ensureCapacity(size + width);
            String str = (obj == null ? getNullText() : obj.toString());
            int strLen = str.length();
            if (strLen >= width) {
                str.getChars(strLen - width, strLen, buffer, size);
            } else {
                int padLen = width - strLen;
                for (int i = 0; i < padLen; i++) {
                    buffer[size + i] = padChar;
                }
                str.getChars(0, strLen, buffer, size + padLen);
            }
            size += width;
        }
        return this;
    }
}

```

## Description

StrBuilder.getNullText() may return null.


## Rule

Missing Null Check

## Response

### Confidence

90

### Types

Missing Null Check

### Crash

True

### Snippet

```java
String str = (obj == null ? getNullText() : obj.toString());
int strLen = str.length();
```

### Fix

```java
String str = (obj == null ? getNullText() : obj.toString());
int strLen = (str==null ? 0 : str.length());
```

### Description

API misuse problem detected in the 'appendFixedWidthPadLeft(Object, int, char)' method of the 'StrBuilder' class. The variable 'str' can be null if the input object 'obj' is null and 'nullText' is also null. The API method 'getNullText()' can return null. In this case, calling the 'length()' method on a null String 'str' would result in a Null-Deference exception.

## Review

Found the expected misuse? **True**

