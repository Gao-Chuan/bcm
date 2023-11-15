## Repository

URL: https://github.com/itext/itextpdf.git

Commit: adda8eb38978eeea6e86aca64ae746754c103dae

## API

java.util.StringTokenizer

## Caller

```java
package com.itextpdf.text.pdf;

import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.OutputStreamWriter;
import java.io.Reader;
import java.io.Writer;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Stack;
import java.util.StringTokenizer;

import com.itextpdf.text.error_messages.MessageLocalization;
import com.itextpdf.text.xml.XMLUtil;
import com.itextpdf.text.xml.simpleparser.IanaEncodings;
import com.itextpdf.text.xml.simpleparser.SimpleXMLDocHandler;
import com.itextpdf.text.xml.simpleparser.SimpleXMLParser;

public final class SimpleBookmark implements SimpleXMLDocHandler {

    private ArrayList<HashMap<String, Object>> topList;
    private final Stack<HashMap<String, Object>> attr = new Stack<HashMap<String, Object>>();

    private SimpleBookmark() {
    }

    @SuppressWarnings("unchecked")
    public static Object[] iterateOutlines(PdfWriter writer, PdfIndirectReference parent, List<HashMap<String, Object>> kids, boolean namedAsNames) throws IOException {
        PdfIndirectReference refs[] = new PdfIndirectReference[kids.size()];
        for (int k = 0; k < refs.length; ++k)
            refs[k] = writer.getPdfIndirectReference();
        int ptr = 0;
        int count = 0;
        for (Iterator<HashMap<String, Object>> it = kids.listIterator(); it.hasNext(); ++ptr) {
            HashMap<String, Object> map = it.next();
            Object lower[] = null;
            List<HashMap<String, Object>> subKid = (List<HashMap<String, Object>>)map.get("Kids");
            if (subKid != null && !subKid.isEmpty())
                lower = iterateOutlines(writer, refs[ptr], subKid, namedAsNames);
            PdfDictionary outline = new PdfDictionary();
            ++count;
            if (lower != null) {
                outline.put(PdfName.FIRST, (PdfIndirectReference)lower[0]);
                outline.put(PdfName.LAST, (PdfIndirectReference)lower[1]);
                int n = ((Integer)lower[2]).intValue();
                if ("false".equals(map.get("Open"))) {
                    outline.put(PdfName.COUNT, new PdfNumber(-n));
                }
                else {
                    outline.put(PdfName.COUNT, new PdfNumber(n));
                    count += n;
                }
            }
            outline.put(PdfName.PARENT, parent);
            if (ptr > 0)
                outline.put(PdfName.PREV, refs[ptr - 1]);
            if (ptr < refs.length - 1)
                outline.put(PdfName.NEXT, refs[ptr + 1]);
            outline.put(PdfName.TITLE, new PdfString((String)map.get("Title"), PdfObject.TEXT_UNICODE));
            String color = (String)map.get("Color");
            if (color != null) {
                try {
                    PdfArray arr = new PdfArray();
                    StringTokenizer tk = new StringTokenizer(color);
                    for (int k = 0; k < 3; ++k) {
                        float f = Float.parseFloat(tk.nextToken());
                        if (f < 0) f = 0;
                        if (f > 1) f = 1;
                        arr.add(new PdfNumber(f));
                    }
                    outline.put(PdfName.C, arr);
                } catch(Exception e){} //in case it's malformed
            }
            String style = (String)map.get("Style");
            if (style != null) {
                style = style.toLowerCase();
                int bits = 0;
                if (style.indexOf("italic") >= 0)
                    bits |= 1;
                if (style.indexOf("bold") >= 0)
                    bits |= 2;
                if (bits != 0)
                    outline.put(PdfName.F, new PdfNumber(bits));
            }
            createOutlineAction(outline, map, writer, namedAsNames);
            writer.addToBody(outline, refs[ptr]);
        }
        return new Object[]{refs[0], refs[refs.length - 1], Integer.valueOf(count)};
    }
}

```

## Description

On line 549, `StringTokenizer.nextToken()` is invoked without prior check of `hasMoreTokens()`. This usage may be safe from context, since the tokenized value is an RBG color value which has a known number of tokens. However, the usage is unsafe against invalid values and we, therefore, consider it a misuse. (Same finding as `mudetectxp-3`)


## Rule

Missing State Check

## Response

### Confidence

90

### Types

Bad Practice

### Crash

False

### Snippet

```java
try {
    PdfArray arr = new PdfArray();
    StringTokenizer tk = new StringTokenizer(color);
    for (int k = 0; k < 3; ++k) {
        float f = Float.parseFloat(tk.nextToken());
        if (f < 0) f = 0;
        if (f > 1) f = 1;
        arr.add(new PdfNumber(f));
    }
    outline.put(PdfName.C, arr);
} catch(Exception e){} //in case it's malformed
```

### Fix

```java
try {
    PdfArray arr = new PdfArray();
    StringTokenizer tk = new StringTokenizer(color);
    for (int k = 0; k < 3; ++k) {
        float f = Float.parseFloat(tk.nextToken());
        if (f < 0) f = 0;
        if (f > 1) f = 1;
        arr.add(new PdfNumber(f));
    }
    outline.put(PdfName.C, arr);
} catch(Exception e){
   // Log the exception
   System.err.println("Exception: " + e.getMessage());
   // You might want to consider re-throwing the exception
}
```

### Description

This snippet is using a StringTokenizer to parse a color string. For the API misuse, the code does not handle exceptions correctly. In the catch block, it only comments the reason of the exception but does nothing to handle the exception (e.g., logging the message of the exception). This is a bad practice because when the color string is malformed, this catch block is executed and the problem becomes invisible due to the silent catch block.

## Review

Found the expected misuse? **False**

