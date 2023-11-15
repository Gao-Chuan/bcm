## Repository

URL: https://git-wip-us.apache.org/repos/asf/commons-math.git

Commit: d270055e874148a2742604be36ab977eec030fba~1

## API

org.apache.commons.math3.geometry.euclidean.twod.Line

## Caller

```java
package org.apache.commons.math3.geometry.euclidean.twod;

public class SubLine extends AbstractSubHyperplane<Euclidean2D, Euclidean1D> {
    public Vector2D intersection(final SubLine subLine, final boolean includeEndPoints) {

        // retrieve the underlying lines
        Line line1 = (Line) getHyperplane();
        Line line2 = (Line) subLine.getHyperplane();

        // compute the intersection on infinite line
        Vector2D v2D = line1.intersection(line2);

        // check location of point with respect to first sub-line
        Location loc1 = getRemainingRegion().checkPoint(line1.toSubSpace(v2D));

        // check location of point with respect to second sub-line
        Location loc2 = subLine.getRemainingRegion().checkPoint(line2.toSubSpace(v2D));

        if (includeEndPoints) {
            return ((loc1 != Location.OUTSIDE) && (loc2 != Location.OUTSIDE)) ? v2D : null;
        } else {
            return ((loc1 == Location.INSIDE) && (loc2 == Location.INSIDE)) ? v2D : null;
        }

    }
}

```

## Description

SubLine.intersection() may return null.


## Rule

Missing Null Check

## Response

### Confidence

80

### Types

Missing Null Check

### Crash

False

### Snippet

```java
Vector2D v2D = line1.intersection(line2);
```

### Fix

```java
Vector2D v2D = line1.intersection(line2);
if (v2D == null) {
    return null;
}
```

### Description

The api 'intersection' from the Line class can return null if the two lines are parallel. However, in the 'intersection' method of SubLine class, the possibility of v2D being null isn't handled. This can lead to a NullPointerException in the subsequent calls to 'checkPoint' method.

## Review

Found the expected misuse? **True**

