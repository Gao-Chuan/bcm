## Repository

URL: http://jigsaw.w3.org/

Commit: http://jigsaw.w3.org/Devel/Mirror/jigsaw_2.0.5.zip

## API

java.util.Enumeration

## Caller

```java
package org.w3c.jigsaw.servlet;

import java.io.*;
import java.util.*;
import java.security.Principal;

import javax.servlet.*;
import javax.servlet.http.*;

import org.w3c.util.*;
import org.w3c.jigsaw.http.*;
import org.w3c.jigsaw.forms.URLDecoder;
import org.w3c.jigsaw.forms.URLDecoderException;
import org.w3c.jigsaw.resources.VirtualHostResource;
import org.w3c.www.http.*;
import org.w3c.www.mime.*;
import org.w3c.jigsaw.auth.AuthFilter; // for auth infos access

import org.w3c.tools.resources.*;

public class JigsawHttpServletRequest implements HttpServletRequest {
    private final static int STREAM_STATE_INITIAL = 0;

    private final static int STREAM_READER_USED = 1;

    private final static int INPUT_STREAM_USED = 2;

    private int stream_state = STREAM_STATE_INITIAL;

    public final static 
	String STATE_PARAMETERS = "org.w3c.jigsaw.servlet.stateParam";

    private static MimeType type = MimeType.APPLICATION_X_WWW_FORM_URLENCODED ;

    private Request request = null;

    private Servlet servlet = null;

    private Hashtable queryParameters = null;

    protected JigsawHttpServletResponse response = null;

    protected JigsawHttpSession httpSession = null;

    protected JigsawHttpSessionContext sessionContext = null;

    protected String requestedSessionID = null;

    public Locale getLocale() {
	return (Locale)getLocales().nextElement();
    }

    public Enumeration getLocales() {
	HttpAcceptLanguage languages[] = request.getAcceptLanguage();
	if (languages == null) {
	    Vector def = new Vector();
            def.addElement(Locale.getDefault());
            return def.elements();
	}

	Vector locales = new Vector(); 

	for (int i = 0 ; i < languages.length ; i++) {
	    HttpAcceptLanguage language = languages[i];
	    double quality = language.getQuality();
	    String lang    = language.getLanguage();
	    String country = "";
	    int    idx     = lang.indexOf('-');
	    if (idx > -1) {
		country = lang.substring(idx + 1).trim();
		lang    = lang.substring(0, idx).trim();
	    }
	    // insert the Locale in ordered list
	    int     qidx = 0;
	    int     size = locales.size();
	    if (size > 0) {
		QLocale ql   = (QLocale) locales.firstElement();
		while ((qidx < size) && (ql.getLanguageQuality() >= quality)) {
		    try {
			ql = (QLocale) locales.elementAt(++qidx);
		    } catch (ArrayIndexOutOfBoundsException ex) {
			//end of vector, so append
		    }
		}
		locales.insertElementAt(new QLocale(lang, country, quality),
					qidx);
	    } else {
		locales.addElement(new QLocale(lang, country, quality));
	    }
	}
	// because Locale is final :(
	int    size    = locales.size(); 
	Vector vlocale = new Vector(size);
	for (int i = 0 ; i < size ; i ++) {
	    vlocale.addElement(((QLocale)locales.elementAt(i)).getLocale());
	}
	return vlocale.elements();
    }
}

```

## Description

On line 434, an element is fetched through `Enumeration.nextElement()`
without checking that there is such an element.


## Rule

Missing State Check

## Response

### Confidence

90

### Types

Other

### Crash

False

### Snippet

```java
public Locale getLocale() {
	return (Locale)getLocales().nextElement();
}
```

### Fix

```java
/* Here the 'getLocales()' method will compute the full list of Locales, even if we just need the first one. Change the method 'getLocale()' to compute just the first Locale for better performance. */
public Locale getLocale() {
	HttpAcceptLanguage languages[] = request.getAcceptLanguage();
	if (languages == null) {
	    return Locale.getDefault();
	}
	HttpAcceptLanguage language = languages[0];
	double quality = language.getQuality();
	String lang    = language.getLanguage();
	String country = "";
	int    idx     = lang.indexOf('-');
	if (idx > -1) {
	    country = lang.substring(idx + 1).trim();
	    lang    = lang.substring(0, idx).trim();
	}
	return new Locale(lang, country);
}
```

### Description

The problem here resides in the 'getLocale()' method. This method calls 'getLocales().nextElement();' to get its result. However, 'getLocales()' method does a lot more than necessary. It calculates all the Locales, ordered by their quality, but we only need the first one. To improve the performance, we can redo the logic of 'getLocales()' situated inside 'getLocale()', stopping when we have the first Locale.

## Review

Found the expected misuse? **False**

