## Repository

URL: https://github.com/cbeust/testng.git

Commit: 677302cb8b5a2507df97c5822eef3a03ebc4e23a^1

## API

org.testng.reporters.XMLStringBuffer

https://android.googlesource.com/platform/external/testng/+/10c223b/src/main/java/org/testng/reporters/XMLStringBuffer.java

## Caller

```java
package org.testng.reporters;

import org.testng.IReporter;
import org.testng.IResultMap;
import org.testng.ISuite;
import org.testng.ISuiteResult;
import org.testng.ITestContext;
import org.testng.ITestResult;
import org.testng.collections.ListMultiMap;
import org.testng.collections.Maps;
import org.testng.internal.Utils;
import org.testng.xml.XmlSuite;

import java.io.File;
import java.io.IOException;
import java.util.List;
import java.util.Map;

public class JqReporter implements IReporter {
  private static final String C = "class";
  private static final String D = "div";
  private static final String S = "span";

  private int m_testCount = 0;
  private String m_outputDirectory;
  private Map<String, String> m_testMap = Maps.newHashMap();

  private XMLStringBuffer generateSuites(List<XmlSuite> xmlSuites,
      List<ISuite> suites, XMLStringBuffer main) {
     for (ISuite suite : suites) {
      if (suite.getResults().size() == 0) {
        continue;
      }

      XMLStringBuffer xsb = new XMLStringBuffer(main.getCurrentIndent());
      XMLStringBuffer header = new XMLStringBuffer(main.getCurrentIndent());

      xsb.push(D, C, "suite-content");
      Map<String, ISuiteResult> results = suite.getResults();
      XMLStringBuffer xs1 = new XMLStringBuffer(xsb.getCurrentIndent());
      XMLStringBuffer xs2 = new XMLStringBuffer(xsb.getCurrentIndent());
      XMLStringBuffer xs3 = new XMLStringBuffer(xsb.getCurrentIndent());
      int failed = 0;
      int skipped = 0;
      int passed = 0;
      for (ISuiteResult result : results.values()) {
        ITestContext context = result.getTestContext();
        failed += context.getFailedTests().size();
        generateTests("failed", context.getFailedTests(), context, xs1);
        skipped += context.getSkippedTests().size();
        generateTests("skipped", context.getSkippedTests(), context, xs2);
        passed += context.getPassedTests().size();
        generateTests("passed", context.getPassedTests(), context, xs3);
      }
      xsb.addOptional(D, "Failed" + " tests", C, "result-banner " + "failed");
      xsb.addString(xs1.toXML());
      xsb.addOptional(D, "Skipped" + " tests", C, "result-banner " + "skipped");
      xsb.addString(xs2.toXML());
      xsb.addOptional(D, "Passed" + " tests", C, "result-banner " + "passed");
      xsb.addString(xs3.toXML());


      header.push(D, C, "suite");
      header.push(D, C, "suite-header");
      header.addOptional(S, suite.getName(), C, "suite-name");
      header.push(D, C, "stats");
      int total = failed + skipped + passed;
      String stats = String.format("%s, %d failed, %d skipped, %d passed",
          pluralize(total, "method"), failed, skipped, passed);
      header.push("ul");

      // Method stats
      header.push("li");
      header.addOptional(S, stats, C, "method-stats");
      header.pop("li");

      // Tests
      header.push("li");
      header.addOptional(S, String.format("%s ", pluralize(results.values().size(), "test"),
          C, "test-stats"));
      header.pop("li");

      // List of tests
      header.push("ul");
      for (ISuiteResult tr : results.values()) {
        String testName = tr.getTestContext().getName();
        header.push("li");
        header.addOptional("a", testName, "href", "#" + m_testMap.get(testName));
        header.pop("li");
      }
      header.pop("ul");

      header.pop("ul");
      header.pop(D);

      header.pop(D);

      main.addString(header.toXML());
      main.addString(xsb.toXML());
    }

    return main;
  }
}

```

## Description

In line 58, the `XMLStringBuffer` instance created in line 55 gets a new tag
pushed ("suite-content"). This tag is never poped, which results in a missing
closing tag in the XML retrieved via `toXML()` in line 119.


## Rule

Missing Call

## Response

### Confidence

100

### Types

code smells

### Crash

False

### Snippet

```java
xsb.push(D, C, "suite-content");
//...
xsb.addOptional(D, "Failed" + " tests", C, "result-banner " + "failed");
//...
xsb.addOptional(D, "Skipped" + " tests", C, "result-banner " + "skipped");
//...
xsb.addOptional(D, "Passed" + " tests", C, "result-banner " + "passed");
```

### Fix

```java
xsb.push(D, C, "suite-content");
//...
xsb.addOptional(D, "Failed tests", C, "result-banner failed");
//...
xsb.addOptional(D, "Skipped tests", C, "result-banner skipped");
//...
xsb.addOptional(D, "Passed tests", C, "result-banner passed");
```

### Description

In the source code, several strings are being joined with '+', but the strings are static and can instead be joined directly. It is not wrong but joining static strings with '+' is unnecessary and can look confusing. This is considered a code smell and should be avoided.

## Review

Found the expected misuse? **False**

