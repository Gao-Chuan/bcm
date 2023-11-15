## Repository

URL: https://github.com/itext/itextpdf.git

Commit: adda8eb38978eeea6e86aca64ae746754c103dae

## API

java.lang.String

## Caller

```java
package com.itextpdf.text.pdf;

import java.util.Arrays;
import com.itextpdf.text.error_messages.MessageLocalization;

import com.itextpdf.text.ExceptionConverter;
import com.itextpdf.text.Rectangle;
import com.itextpdf.text.BaseColor;

public class BarcodeEAN extends Barcode{
        
	private static final int GUARD_EMPTY[] = {};
	private static final int GUARD_UPCA[] = {0, 2, 4, 6, 28, 30, 52, 54, 56, 58};
	private static final int GUARD_EAN13[] = {0, 2, 28, 30, 56, 58};
	private static final int GUARD_EAN8[] = {0, 2, 20, 22, 40, 42};
	private static final int GUARD_UPCE[] = {0, 2, 28, 30, 32};
	private static final float TEXTPOS_EAN13[] = {6.5f, 13.5f, 20.5f, 27.5f, 34.5f, 41.5f, 53.5f, 60.5f, 67.5f, 74.5f, 81.5f, 88.5f};
	private static final float TEXTPOS_EAN8[] = {6.5f, 13.5f, 20.5f, 27.5f, 39.5f, 46.5f, 53.5f, 60.5f};
	private static final byte BARS[][] = 
    {
        {3, 2, 1, 1}, // 0
        {2, 2, 2, 1}, // 1
        {2, 1, 2, 2}, // 2
        {1, 4, 1, 1}, // 3
        {1, 1, 3, 2}, // 4
        {1, 2, 3, 1}, // 5
        {1, 1, 1, 4}, // 6
        {1, 3, 1, 2}, // 7
        {1, 2, 1, 3}, // 8
        {3, 1, 1, 2}  // 9
    };
    
	private static final int TOTALBARS_EAN13 = 11 + 12 * 4;
	private static final int TOTALBARS_EAN8 = 11 + 8 * 4;
	private static final int TOTALBARS_UPCE = 9 + 6 * 4;
	private static final int TOTALBARS_SUPP2 = 13;
	private static final int TOTALBARS_SUPP5 = 31;
	private static final int ODD = 0;
	private static final int EVEN = 1;
    
    private static final byte PARITY13[][] =
    {
        {ODD, ODD,  ODD,  ODD,  ODD,  ODD},  // 0
        {ODD, ODD,  EVEN, ODD,  EVEN, EVEN}, // 1
        {ODD, ODD,  EVEN, EVEN, ODD,  EVEN}, // 2
        {ODD, ODD,  EVEN, EVEN, EVEN, ODD},  // 3
        {ODD, EVEN, ODD,  ODD,  EVEN, EVEN}, // 4
        {ODD, EVEN, EVEN, ODD,  ODD,  EVEN}, // 5
        {ODD, EVEN, EVEN, EVEN, ODD,  ODD},  // 6
        {ODD, EVEN, ODD,  EVEN, ODD,  EVEN}, // 7
        {ODD, EVEN, ODD,  EVEN, EVEN, ODD},  // 8
        {ODD, EVEN, EVEN, ODD,  EVEN, ODD}   // 9
    };
    
    private static final byte PARITY2[][] =
    {
        {ODD,  ODD},   // 0
        {ODD,  EVEN},  // 1
        {EVEN, ODD},   // 2
        {EVEN, EVEN}   // 3
    };
    
    private static final byte PARITY5[][] =
    {
        {EVEN, EVEN, ODD,  ODD,  ODD},  // 0
        {EVEN, ODD,  EVEN, ODD,  ODD},  // 1
        {EVEN, ODD,  ODD,  EVEN, ODD},  // 2
        {EVEN, ODD,  ODD,  ODD,  EVEN}, // 3
        {ODD,  EVEN, EVEN, ODD,  ODD},  // 4
        {ODD,  ODD,  EVEN, EVEN, ODD},  // 5
        {ODD,  ODD,  ODD,  EVEN, EVEN}, // 6
        {ODD,  EVEN, ODD,  EVEN, ODD},  // 7
        {ODD,  EVEN, ODD,  ODD,  EVEN}, // 8
        {ODD,  ODD,  EVEN, ODD,  EVEN}  // 9
    };
    
    private static final byte PARITYE[][] =
    {
        {EVEN, EVEN, EVEN, ODD,  ODD,  ODD},  // 0
        {EVEN, EVEN, ODD,  EVEN, ODD,  ODD},  // 1
        {EVEN, EVEN, ODD,  ODD,  EVEN, ODD},  // 2
        {EVEN, EVEN, ODD,  ODD,  ODD,  EVEN}, // 3
        {EVEN, ODD,  EVEN, EVEN, ODD,  ODD},  // 4
        {EVEN, ODD,  ODD,  EVEN, EVEN, ODD},  // 5
        {EVEN, ODD,  ODD,  ODD,  EVEN, EVEN}, // 6
        {EVEN, ODD,  EVEN, ODD,  EVEN, ODD},  // 7
        {EVEN, ODD,  EVEN, ODD,  ODD,  EVEN}, // 8
        {EVEN, ODD,  ODD,  EVEN, ODD,  EVEN}  // 9
    };
    
    public static byte[] getBarsSupplemental2(String _code) {
        int code[] = new int[2];
        for (int k = 0; k < code.length; ++k)
            code[k] = _code.charAt(k) - '0';
        byte bars[] = new byte[TOTALBARS_SUPP2];
        int pb = 0;
        int parity = (code[0] * 10 + code[1]) % 4;
        bars[pb++] = 1;
        bars[pb++] = 1;
        bars[pb++] = 2;
        byte sequence[] = PARITY2[parity];
        for (int k = 0; k < sequence.length; ++k) {
            if (k == 1) {
                bars[pb++] = 1;
                bars[pb++] = 1;
            }
            int c = code[k];
            byte stripes[] = BARS[c];
            if (sequence[k] == ODD) {
                bars[pb++] = stripes[0];
                bars[pb++] = stripes[1];
                bars[pb++] = stripes[2];
                bars[pb++] = stripes[3];
            }
            else {
                bars[pb++] = stripes[3];
                bars[pb++] = stripes[2];
                bars[pb++] = stripes[1];
                bars[pb++] = stripes[0];
            }
        }
        return bars;
    }   
}

```

## Description

On line 365 `_code.charAt()` is invoked without ensuring that the respective
string actually has sufficiently many characters. From the constant
initialization on line 363 we take that `k` will assume the values 0 and 1,
while `_code` might be any string.


## Rule

Missing State Check

## Response

### Confidence

85

### Types

Missing Null Check

### Crash

True

### Snippet

```java
for (int k = 0; k < code.length; ++k)
            code[k] = _code.charAt(k) - '0';
```

### Fix

```java
if (_code != null) {
for (int k = 0; k < code.length; ++k)
            code[k] = _code.charAt(k) - '0';
}
```

### Description

This issue is related to the Java API method `charAt` inside the for loop. The problem is that if the method getBarsSupplemental2 is called with a null String parameter, then the method '_code.charAt(k) - '0'' will throw a NullPointerException because '_code.charAt(k)' is trying to access a method of a null object '_code'. This NullPointerException could potentially crash the program. Therefore, it is required to include a null check to the string '_code' before usage to prevent a potential NullPointerException.

## Review

Found the expected misuse? **False**

