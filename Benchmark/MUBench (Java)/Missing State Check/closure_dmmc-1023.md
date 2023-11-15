## Repository

URL: https://github.com/google/closure-compiler.git

Commit: 43c245f0ff8d409e81e25687e69d34666b7cf26a~1

## API

java.util.Iterator

## Caller

```java
package com.google.javascript.jscomp.deps;

import com.google.common.base.Joiner;
import com.google.common.base.Preconditions;
import com.google.common.collect.ArrayListMultimap;
import com.google.common.collect.HashMultimap;
import com.google.common.collect.HashMultiset;
import com.google.common.collect.ImmutableList;
import com.google.common.collect.Lists;
import com.google.common.collect.Maps;
import com.google.common.collect.Multimap;
import com.google.common.collect.Multimaps;
import com.google.common.collect.Multiset;
import com.google.common.collect.Sets;

import java.util.ArrayDeque;
import java.util.Collection;
import java.util.Collections;
import java.util.Comparator;
import java.util.Deque;
import java.util.List;
import java.util.Map;
import java.util.PriorityQueue;
import java.util.Set;

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

Iterator.next() is invoked without prior check of hasNext(). (Same finding as `mudetect-16` and `mudetectxp-0`)


## Rule

Missing State Check

## Response

### Confidence

85

### Types

Missing State Check

### Crash

True

### Snippet

```java
symbols.add(cycle.get(i).getProvides().iterator().next());
```

### Fix

```java
if(cycle.get(i).getProvides().iterator().hasNext()) {
 symbols.add(cycle.get(i).getProvides().iterator().next());
}
```

### Description

The method 'cycleToString' calls 'next()' on an Iterator without checking if it has a next element using 'hasNext()'. This can lead to a NoSuchElementException if there are no more elements.

## Review

Found the expected misuse? **True**

