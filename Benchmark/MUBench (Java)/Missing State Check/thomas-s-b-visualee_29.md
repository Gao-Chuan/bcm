## Repository

URL: https://github.com/Thomas-S-B/visualee.git

Commit: 410a80f0f4774b093c3689d0e0c75bc46c78688e

## API

java.util.Scanner

## Caller

```java
package de.strullerbaumann.visualee.examiner;

import de.strullerbaumann.visualee.dependency.boundary.DependencyContainer;
import de.strullerbaumann.visualee.dependency.entity.Dependency;
import de.strullerbaumann.visualee.dependency.entity.DependencyType;
import de.strullerbaumann.visualee.logging.LogProvider;
import de.strullerbaumann.visualee.source.boundary.JavaSourceContainer;
import de.strullerbaumann.visualee.source.entity.JavaSource;
import java.util.ArrayDeque;
import java.util.Deque;
import java.util.Scanner;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public abstract class Examiner {

   private static final String[] JAVA_TOKENS = {
      "void",
      "private",
      "protected",
      "transient",
      "public",
      "static",
      "@"
   };

   protected static void findAndSetPackage(JavaSource javaSource) {
      Scanner scanner = Examiner.getSourceCodeScanner(javaSource.getSourceCode());
      while (scanner.hasNext()) {
         String token = scanner.next();
         if (javaSource.getPackagePath() == null && token.equals("package")) {
            token = scanner.next();
            if (token.endsWith(";")) {
               String packagePath = token.substring(0, token.indexOf(';'));
               javaSource.setPackagePath(packagePath);
            }
         }
      }
   }
}

```

## Description

In line 256, `scanner.next()` is called without checking if there are more
elements. Because the scanner is built from the `JavaSource` parameter,
which may be invalid (e.g., package keyword followed by an empty package
name), this may lead to a runtime exception without a useful error message.


## Rule

Missing State Check

## Response

### Confidence

90

### Types

Resource Leak

### Crash

True

### Snippet

```java
Scanner scanner = Examiner.getSourceCodeScanner(javaSource.getSourceCode());
```

### Fix

```java
try (Scanner scanner = Examiner.getSourceCodeScanner(javaSource.getSourceCode())) {
   // rest of the code
}
```

### Description

The Scanner object is never closed, causing a resource leak. The Scanner should be closed after done using it to release its associate resources. Here, Scanner is used in a try-with-resource statement to ensure that the close() method is called at the end of the statement.

## Review

Found the expected misuse? **False**

