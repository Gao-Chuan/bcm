## Repository

URL: https://github.com/google/closure-compiler.git

Commit: 43c245f0ff8d409e81e25687e69d34666b7cf26a~1

## API

org.kohsuke.args4j.spi.Parameters

https://github.com/kohsuke/args4j/blob/b819bd367a70fe102f7a7cab628c2e9f080705fe/args4j/src/org/kohsuke/args4j/spi/Parameters.java#L13

## Caller

```java
package com.google.javascript.jscomp;

import com.google.common.base.Preconditions;
import com.google.common.collect.ImmutableList;
import com.google.common.collect.Lists;
import com.google.common.collect.Maps;
import com.google.common.collect.Sets;
import com.google.common.io.LimitInputStream;

import org.kohsuke.args4j.CmdLineException;
import org.kohsuke.args4j.CmdLineParser;
import org.kohsuke.args4j.Option;
import org.kohsuke.args4j.OptionDef;
import org.kohsuke.args4j.spi.OptionHandler;
import org.kohsuke.args4j.spi.Parameters;
import org.kohsuke.args4j.spi.Setter;

import java.io.IOException;
import java.io.InputStream;
import java.io.PrintStream;
import java.util.List;
import java.util.Map;
import java.util.ResourceBundle;
import java.util.Set;
import java.util.logging.Level;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.util.zip.ZipEntry;
import java.util.zip.ZipInputStream;

public class CommandLineRunner extends
    AbstractCommandLineRunner<Compiler, CompilerOptions> {

  private static class Flags {
      @Override
      public int parseArguments(Parameters params) throws CmdLineException {
        String param = params.getParameter(0);
        if (param == null) {
          setter.addValue(true);
          return 0;
        } else {
          String lowerParam = param.toLowerCase();
          if (TRUES.contains(lowerParam)) {
            setter.addValue(true);
          } else if (FALSES.contains(lowerParam)) {
            setter.addValue(false);
          } else {
            setter.addValue(true);
            return 0;
          }
          return 1;
        }
      }
  }

}

```

## Description

Parameters.getParameter() may throw CmdLineException.


## Rule

Missing Exception Handling

## Response

### Confidence

80

### Types

Missing State Check

### Crash

False

### Snippet

```java
String param = params.getParameter(0);
if (param == null) {
  setter.addValue(true);
  return 0;
}
```

### Fix

```java
String param = params.getParameter(0);
if (param == null || param.isEmpty()) {
  setter.addValue(true);
  return 0;
}
```

### Description

The method params.getParameter(0) might return an empty string instead of null. The current code only checks for null but does not handle the case where an empty string is returned. In this case, the behavior of the system might be unpredictable as the value is directly used without any validation.

## Review

Found the expected misuse? **False**

