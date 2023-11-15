## Repository

URL: https://github.com/itext/itextpdf.git

Commit: adda8eb38978eeea6e86aca64ae746754c103dae

## API

com.itextpdf.text.pdf.PdfArray

## Caller

```java
package com.itextpdf.text.pdf;

public final class SimpleBookmark implements SimpleXMLDocHandler {

    private ArrayList<HashMap<String, Object>> topList;
    private final Stack<HashMap<String, Object>> attr = new Stack<HashMap<String, Object>>();

	private static String makeBookmarkParam(PdfArray dest, IntHashtable pages)
	{
		StringBuffer s = new StringBuffer();
		PdfObject obj = dest.getPdfObject(0);
        if (obj.isNumber())
            s.append(((PdfNumber)obj).intValue() + 1);
        else
            s.append(pages.get(getNumber((PdfIndirectReference)obj))); //changed by ujihara 2004-06-13
		s.append(' ').append(dest.getPdfObject(1).toString().substring(1));
		for (int k = 2; k < dest.size(); ++k)
			s.append(' ').append(dest.getPdfObject(k).toString());
		return s.toString();
	}
}

```

## Description

On line 244, a `PdfObject` is fetched from a `PdfArray`. Since the array may
contain a `null` value, the object should be checked before it is used.


## Rule

Missing Null Check

## Response 0

### Confidence

80

### Types

Missing Null Check

### Crash

True

### Snippet

```java
s.append(' ').append(dest.getPdfObject(1).toString().substring(1));
```

### Fix

```java
if(dest.getPdfObject(1) != null)
	s.append(' ').append(dest.getPdfObject(1).toString().substring(1));
```

### Description

The method 'getPdfObject' of class 'PdfArray' can return null. But in the current code, there is no null-checking logic before calling 'toString' method on the object referenced by getPdfObject. Therefore, null return can cause NullPointerException.

## Response 1

### Confidence

60

### Types

Missing Null Check

### Crash

True

### Snippet

```java
for (int k = 2; k < dest.size(); ++k)
			s.append(' ').append(dest.getPdfObject(k).toString());
```

### Fix

```java
for (int k = 2; k < dest.size(); ++k) {
	if(dest.getPdfObject(k) != null)
		s.append(' ').append(dest.getPdfObject(k).toString());
}
```

### Description

The method 'getPdfObject' of class 'PdfArray' is accessed from 2 to dest.size() without checking whether the getValue will return null or not. It will lead to NullPointerException.

## Review

Found the expected misuse? **True**

