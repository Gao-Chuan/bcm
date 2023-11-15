## Repository

URL: http://jigsaw.w3.org/

Commit: http://jigsaw.w3.org/Devel/Mirror/jigsaw_2.0.5.zip

## API

org.w3c.tools.resources.ResourceReference

## Caller

```java
package org.w3c.jigsaw.proxy;


import java.net.*;
import java.io.*;
import java.util.*;
import org.w3c.www.http.*;
import org.w3c.www.mime.*;
import org.w3c.jigsaw.html.*;
import org.w3c.tools.resources.*;
import org.w3c.jigsaw.http.* ;
import org.w3c.jigsaw.frames.HTTPFrame;
import org.w3c.jigsaw.http.socket.SocketClientFactory;
import org.w3c.util.ObservableProperties;

class Stats extends HTTPFrame {
    ProxyFrame proxy     = null;
    Date       startdate = null;
    boolean    hasICP    = true;

    public boolean lookupOther(LookupState ls, LookupResult lr) 
	throws org.w3c.tools.resources.ProtocolException
    {
	// Get the full URL from the request:
	Request request = (Request) ls.getRequest();
	URL     requrl  = ((request != null)
			   ? request.getURL()
			   : null);
	boolean host_equiv = false;

	// loop check
	if (request != null) {
	    String vias[] = request.getVia();
	    if ((url != null)
		&& (requrl.getPort() == url.getPort())
		&& (vias != null && vias.length > 5)) {	
		// maybe a loop, let's try to sort it out with an expensive
		// checking on IPs
		String hostaddr;
		
		ObservableProperties props = getServer().getProperties();
		hostaddr = props.getString(SocketClientFactory.BINDADDR_P,
					   default_hostaddr);
		if (requrl != null) {
		    InetAddress targhost;
		    String reqhostaddr;
		    
		    try {
			targhost = InetAddress.getByName(requrl.getHost());
			reqhostaddr = targhost.getHostAddress();
			host_equiv =  reqhostaddr.equals(hostaddr);
		    } catch (UnknownHostException uhex) {};
		}
	    }
	}
	if (((url != null)
	     && (requrl != null)
	     && ((requrl.getPort() == url.getPort()) || 
		 (requrl.getPort() * url.getPort() == -80))
	     && ( host_equiv || 
		  (requrl.getHost().equalsIgnoreCase(url.getHost()))))
	     || (ls.isInternal())) {
	    // Call super.lookup:
	    super.lookupOther(ls, lr);
	    if ( ls.hasMoreComponents() ) {
		ResourceReference root = getLocalRootResource();
		if ( root == null ) {
		    lr.setTarget(this.getResource().getResourceReference());
		    return true;
		}
		try {
		    // because the root eats the lookup state components
		    // we have to return true.
		    // Should not be continued by the caller.
		    FramedResource res = (FramedResource)root.lock();
		    boolean done = res.lookup(ls, lr);
		    if (! done)
			lr.setTarget(null);
		    return true;
		} catch (InvalidResourceException ex) {
		    // should never happen with the root resource
		    ex.printStackTrace();
		} finally {
		    root.unlock();
		}
		return true; // should never be reached 
	    } else {
		request.setState(STATE_CONTENT_LOCATION, "true");
		// return the index file.
		String index = getIndex();
		if ( index != null && index.length() > 0) {
		    ResourceReference root = getLocalRootResource();
		    try {
			DirectoryResource dir = 
			    (DirectoryResource)root.lock();
			ResourceReference rr = dir.lookup(index);
			if (rr != null) {
			    try {
				FramedResource rindex = 
				    (FramedResource) rr.lock();
				return rindex.lookup(ls,lr);
			    } catch (InvalidResourceException ex) {
			    } finally {
				rr.unlock();
			    }
			}
		    } catch (InvalidResourceException ex) {
			root.unlock();
		    }
		}
		lr.setTarget(drr);
		return true;
	    }
	} else {
	    // Always invoke super lookup, after notification that its a proxy
	    request.setProxy(true);
	    super.lookupOther(ls, lr);
	    if ( requrl.getProtocol().equals("ftp") ) {
//		if (ftphandler != null)
//		    lr.setTarget(ftphandler);
//		else 
		    lr.setTarget(null);
		return true;
	    } else {
		lr.setTarget(this.getResource().getResourceReference());
		return true;
	    }
	}
    }
}

```

## Description

In line 284 the `ResourceReference` `root` is locked, but there is no
matching `unlock()` assuming normal execution (there is one for an
exceptional case in line 297).
(Same finding as `mudetect-2`)


## Rule

Missing Call

## Response

### Confidence

80

### Types

Other

### Crash

False

### Snippet

```java
`ResourceReference root = getLocalRootResource();
		try {
			DirectoryResource dir = 
			    (DirectoryResource)root.lock();
			ResourceReference rr = dir.lookup(index);
			if (rr != null) {
			    try {
				FramedResource rindex = 
				    (FramedResource) rr.lock();
				return rindex.lookup(ls,lr);
			    } catch (InvalidResourceException ex) {
			    } finally {
				rr.unlock();
			    }
			}
		    } catch (InvalidResourceException ex) {
			root.unlock();
		    }`
```

### Fix

```java
`ResourceReference root = getLocalRootResource();
		try {
			DirectoryResource dir = 
			    (DirectoryResource)root.lock();
			ResourceReference rr = dir.lookup(index);
			if (rr != null) {
			    try {
				FramedResource rindex = 
				    (FramedResource) rr.lock();
				return rindex.lookup(ls,lr);
			    } catch (InvalidResourceException ex) {
			    } finally {
				rr.unlock();
			    }
			}
		    } catch (InvalidResourceException ex) {
			} finally {
			// Added unlock in finally block to ensure it always runs
			root.unlock();
		}`
```

### Description

The API 'org.w3c.tools.resources.ResourceReference.lock()' is used without a corresponding 'unlock()' in the same block for 'root' object. This can potentially lead to resource leaks as a locked resource might not get unlocked if an exception is thrown before the 'unlock()' call.

## Review

Found the expected misuse? **True**

