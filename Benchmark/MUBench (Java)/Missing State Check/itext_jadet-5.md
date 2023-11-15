## Repository

URL: https://github.com/itext/itextpdf.git

Commit: adda8eb38978eeea6e86aca64ae746754c103dae

## API

com.itextpdf.text.pdf.PdfArray

## Caller

```java
package com.itextpdf.text.pdf;

import java.io.IOException;
import java.util.HashMap;

public class PdfStructureTreeRoot extends PdfDictionary {

    private HashMap<Integer, PdfObject> parentTree = new HashMap<Integer, PdfObject>();
    private PdfIndirectReference reference;

    private PdfWriter writer;

    private void nodeProcess(PdfDictionary struc, PdfIndirectReference reference) throws IOException {
        PdfObject obj = struc.get(PdfName.K);
        if (obj != null && obj.isArray() && !(((PdfArray)obj).getPdfObject(0)).isNumber()) {
            PdfArray ar = (PdfArray)obj;
            for (int k = 0; k < ar.size(); ++k) {
                PdfStructureElement e = (PdfStructureElement)ar.getAsDict(k);
                ar.set(k, e.getReference());
                nodeProcess(e, e.getReference());
            }
        }
        if (reference != null)
            writer.addToBody(struc, reference);
    }
}

```

## Description

PdfArray.getPdfObject() is invoked without checking that the array contains sufficiently many objects.


## Rule

Missing State Check

## Response

### Confidence

75

### Types

Unsafe Casting

### Crash

True

### Snippet

```java
PdfStructureElement e = (PdfStructureElement)ar.getAsDict(k);
```

### Fix

```java
if (ar.getAsDict(k) instanceof PdfStructureElement) {
    PdfStructureElement e = (PdfStructureElement) ar.getAsDict(k);
}
```

### Description

The code immediately casts the return value of `ar.getAsDict(k)` without checking whether it's the expected type `PdfStructureElement`. If the object is not of the type `PdfStructureElement`, a ClassCastException would be thrown, leading to a crash.

## Review

Found the expected misuse? **False**

