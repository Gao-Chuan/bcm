## Repository

URL: https://github.com/google/closure-compiler.git

Commit: 67289ae4cbaba3ae70cd2e8fb92f3f2898039dfb~1

## API

com.google.javascript.rhino.jstype.UnionTypeBuilder

## Caller

```java
package com.google.javascript.rhino.jstype;

import static com.google.javascript.rhino.jstype.TernaryValue.UNKNOWN;

import com.google.common.collect.ImmutableSet;
import com.google.javascript.rhino.ErrorReporter;
import com.google.javascript.rhino.jstype.JSType.TypePair;

import java.util.Set;
import java.util.SortedSet;
import java.util.TreeSet;

public class UnionType extends JSType {
  private static final long serialVersionUID = 1L;

  Set<JSType> alternates;

  JSType meet(JSType that) {
    UnionTypeBuilder builder = new UnionTypeBuilder(registry);
    for (JSType alternate : alternates) {
      if (alternate.isSubtype(that)) {
        builder.addAlternate(alternate);
      }
    }

    if (that instanceof UnionType) {
      for (JSType otherAlternate : ((UnionType) that).alternates) {
        if (otherAlternate.isSubtype(this)) {
          builder.addAlternate(otherAlternate);
        }
      }
    } else if (that.isSubtype(this)) {
      builder.addAlternate(that);
    }
    JSType result = builder.build();
    if (result != null) {
      return result;
    } else if (this.isObject() && that.isObject()) {
      return getNativeType(JSTypeNative.NO_OBJECT_TYPE);
    } else {
      return getNativeType(JSTypeNative.NO_TYPE);
    }
  }

}

```

## Description

UnionTypeBuilder.build() returns a JSType that can never be null. Branching on a null check therefore results in dead code.


## Rule

Redundant Null Check

## Response

### Confidence

90

### Types

Other

### Crash

False

### Snippet

```java
JSType result = builder.build();
    if (result != null) {
      return result;
    } else if (this.isObject() && that.isObject()) {
      return getNativeType(JSTypeNative.NO_OBJECT_TYPE);
    } else {
      return getNativeType(JSTypeNative.NO_TYPE);
    }
```

### Fix

```java
JSType result = builder.build();
    if (result != null) {
      return result;
    } else if (this.isObject()) {
      return getNativeType(JSTypeNative.NO_OBJECT_TYPE);
    } else {
      return getNativeType(JSTypeNative.NO_TYPE);
    }
```

### Description

In the method 'meet(JSType)', there is a potential issue where it checks if both 'this' and 'that' are objects before returning a NoObjectType. However, if 'that' is not an object, it should not return NoObjectType, because the meet method still need to return a type even 'that' is not an object. The API misuse here is the condition checking, 'that.isObject()' should be eliminated.

## Review

Found the expected misuse? **False**

