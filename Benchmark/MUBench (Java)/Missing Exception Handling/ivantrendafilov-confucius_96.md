## Repository

URL: https://github.com/IvanTrendafilov/Confucius.git

Commit: 2c3028782eaaf2337bde558b4a700210a10f543e

## API

java.lang.Long

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
	
	public long getLongValue(String key) {
		return Long.parseLong(getKey(key));
	}
}

```

## Description

`AbstractConfiguration.java` calls `java.lang.Long.parseLong` without first checking whether the argument parses.
This lead to an uncaught `NumberFormatException`: [Oracle Java 7 API specification](http://docs.oracle.com/javase/7/docs/api/java/lang/Long.html#parseLong%28java.lang.String,%20int%29).

This pull request adds a check with a  more helpful exception message and tests for this issue.


## Rule

Missing Exception Handling

## Response

### Confidence

80

### Types

Missing Exception Handling

### Crash

True

### Snippet

```java
public long getLongValue(String key) {\n\t\treturn Long.parseLong(getKey(key));\n\t}
```

### Fix

```java
public long getLongValue(String key) {\n\t\ttry {\n\t\t\treturn Long.parseLong(getKey(key));\n\t\t} catch (NumberFormatException e) {\n\t\t\t// Handle this exception\n\t\t}\n\t\treturn 0; // or some suitable default\n\t}
```

### Description

The method 'getLongValue' uses 'Long.parseLong()' without any exception handling. If the 'getKey(key)' method returns a value that cannot be parsed into a 'long', then a 'NumberFormatException' will be thrown, causing the application to crash.

## Review

Found the expected misuse? **True**

