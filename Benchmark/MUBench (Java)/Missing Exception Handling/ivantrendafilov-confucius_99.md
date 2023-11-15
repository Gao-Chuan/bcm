## Repository

URL: https://github.com/IvanTrendafilov/Confucius.git

Commit: 2c3028782eaaf2337bde558b4a700210a10f543e

## API

java.lang.Short

## Caller

```java
package org.trendafilov.confucius.core;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.trendafilov.confucius.Configurable;

import java.io.InputStream;
import java.util.*;
import java.util.Map.Entry;

public abstract class AbstractConfiguration implements Configurable {
	private final static Logger LOG = LoggerFactory.getLogger(AbstractConfiguration.class);

	private final static String ITEM_SEPARATOR = ",";
	protected static String FILE_PARAM = "conf.properties";
	protected static String CONTEXT_PARAM = "conf.context";

	private final ConfigurationDataProvider configurationDataProvider;
	private final String context;
	private final Map<String, String> initialState;

	public short getShortValue(String key) {
		return Short.parseShort(getKey(key));
	}
}

```

## Description

`AbstractConfiguration.java` calls `java.lang.Short.parseShort` without first checking whether the argument parses.
This lead to an uncaught `NumberFormatException`: [Oracle Java 7 API specification](http://docs.oracle.com/javase/7/docs/api/java/lang/Long.html#parseLong%28java.lang.String,%20int%29).

This pull request adds a check with a  more helpful exception message and tests for this issue.


## Rule

Missing Exception Handling

## Response

### Confidence

90

### Types

Missing Exception Handling

### Crash

True

### Snippet

```java
public short getShortValue(String key) {
		return Short.parseShort(getKey(key));
	}
```

### Fix

```java
public short getShortValue(String key) {
		try {
			return Short.parseShort(getKey(key));
		} catch (NumberFormatException e) {
			LOG.error("The value for the key '" + key + "' is not a valid Short number").
			return 0;
		}
	}
```

### Description

The API 'Short.parseShort(String s)' throws NumberFormatException if the string does not contain a parsable short. However, the user code does not handle such exceptions, which will potentially crash the program when the exception is thrown.

## Review

Found the expected misuse? **True**

