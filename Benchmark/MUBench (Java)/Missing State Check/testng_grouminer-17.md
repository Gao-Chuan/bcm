## Repository

URL: https://github.com/cbeust/testng.git

Commit: 677302cb8b5a2507df97c5822eef3a03ebc4e23a^1

## API

org.testng.reporters.XMLStringBuffer

## Caller

```java
package org.testng.reporters.jq;

import org.testng.IInvokedMethod;
import org.testng.ISuite;
import org.testng.ITestNGMethod;
import org.testng.ITestResult;
import org.testng.reporters.XMLStringBuffer;

import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class ChronologicalPanel extends BaseMultiSuitePanel {

  public String getContent(ISuite suite, XMLStringBuffer main) {
    XMLStringBuffer xsb = new XMLStringBuffer(main.getCurrentIndent());
    List<IInvokedMethod> invokedMethods = suite.getAllInvokedMethods();

    Collections.sort(invokedMethods, new Comparator<IInvokedMethod>() {
      @Override
      public int compare(IInvokedMethod arg0, IInvokedMethod arg1) {
        return (int)
            (arg0.getTestResult().getStartMillis() - arg1.getTestResult().getStartMillis());
      }
    });

    String currentClass = "";
    long start = 0;
    for (IInvokedMethod im : invokedMethods) {
      ITestNGMethod m = im.getTestMethod();
//    for (ITestResult tr : results) {
//      ITestNGMethod m = tr.getMethod();
      String cls = "test-method";
      if (m.isBeforeSuiteConfiguration()) {
        cls = "configuration-suite before";
      } else if (m.isAfterSuiteConfiguration()) {
        cls = "configuration-suite after";
      } else if (m.isBeforeTestConfiguration()) {
        cls = "configuration-test before";
      } else if (m.isAfterTestConfiguration()) {
        cls = "configuration-test after";
      } else if (m.isBeforeClassConfiguration()) {
        cls = "configuration-class before";
      } else if (m.isAfterClassConfiguration()) {
        cls = "configuration-class after";
      } else if (m.isBeforeMethodConfiguration()) {
        cls = "configuration-method before";
      } else if (m.isAfterMethodConfiguration()) {
        cls = "configuration-method after";
      }
      ITestResult tr = im.getTestResult();
      String methodName = Model.getTestResultName(tr);

      if (!m.getTestClass().getName().equals(currentClass)) {
        if (!"".equals(currentClass)) {
          xsb.pop(D);
        }
        xsb.push(D, C, "chronological-class");
        xsb.addRequired(D, m.getTestClass().getName(), C, "chronological-class-name");
        currentClass = m.getTestClass().getName();
      }
      xsb.push(D, C, cls);
      if (tr.getStatus() == ITestResult.FAILURE) {
        xsb.push("img", "src", Model.getImage("failed"));
        xsb.pop("img");
      }

      // No need to check for skipped methods since by definition, they were never
      // invoked.

      xsb.addRequired(S, methodName, C, "method-name");
      if (start == 0) {
        start = tr.getStartMillis();
      }
      xsb.addRequired(S, Long.toString(tr.getStartMillis() - start)  + " ms", C, "method-start");
      xsb.pop(D);
    }
    return xsb.toXML();
  }
}

```

## Description

In line 71, the `XMLStringBuffer` instance created in line 31 gets a tag
closed (`D`). This tag is never pushed before, which causes an exception
on `pop()`.
(Same finding as `mudetect-11`)


## Rule

Missing State Check

## Response

### Confidence

85

### Types

Other

### Crash

False

### Snippet

```java
xsb.addRequired(D, m.getTestClass().getName(), C, "chronological-class-name");
```

### Fix

```java
xsb.addRequired("chronological-class-name", m.getTestClass().getName());
```

### Description

The method addRequired in XMLStringBuffer requires only two arguments, the name of the tag and its value. However, in the current source code four arguments are being passed, which is incorrect and causes an API misuse problem.

## Review

Found the expected misuse? **False**

