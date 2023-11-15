## Repository

URL: https://github.com/google/closure-compiler.git

Commit: 43c245f0ff8d409e81e25687e69d34666b7cf26a~1

## API

java.util.List

## Caller

```java
package com.google.javascript.jscomp.deps;

public class SortedDependencies<INPUT extends DependencyInfo> {

  private final List<INPUT> inputs;

  // A topologically sorted list of the inputs.
  private final List<INPUT> sortedList;

  // A list of all the inputs that do not have provides.
  private final List<INPUT> noProvides;

  private final Map<String, INPUT> provideMap = Maps.newHashMap();

  private String cycleToString(List<INPUT> cycle) {
    List<String> symbols = Lists.newArrayList();
    for (int i = cycle.size() - 1; i >= 0; i--) {
      symbols.add(cycle.get(i).getProvides().iterator().next());
    }
    symbols.add(symbols.get(0));
    return Joiner.on(" -> ").join(symbols);
  }
}

```

## Description

On line 176, `List.get(0)` is invoked without prior check of that there is at least one element in the list (the loop that adds elements might be skipped, if an empty list is passed as a parameter).


## Rule

Missing State Check

## Response

### Confidence

90

### Types

Missing State Check

### Crash

True

### Snippet

```java
cycle.get(i).getProvides().iterator().next()
```

### Fix

```java
Iterator<String> symbolIterator = cycle.get(i).getProvides().iterator();
if (symbolIterator.hasNext()) {
 symbols.add(symbolIterator.next());
 } else {
 // Handle the case when there is no next element in the iterator
}
```

### Description

The code assumes that there will always be a next element during iteration of the list returned by getProvides(). This may cause NoSuchElementException if there is no next element during iteration. Make sure to handle the case where there is no next element.

## Review

Found the expected misuse? **False**

