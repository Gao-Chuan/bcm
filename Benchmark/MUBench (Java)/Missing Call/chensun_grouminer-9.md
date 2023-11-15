## Repository

URL: https://github.com/ms969/ChenSun

Commit: cf23b99a0c78596b5dc5bb25691736c9abd0a84d

## API

java.sql.PreparedStatement

## Caller

```java
package database;

import java.io.UnsupportedEncodingException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Arrays;

import crypto.CryptoUtil;
import crypto.Hash;
import crypto.SharedKeyCrypto;

public class SocialNetworkDatabasePosts {
	private static String specialStrPostable = "*";
	private static String specialStrCreatedPost = "**";
	
	public static int addFFAParticipipant(Connection conn, int post, String username, String priv) {
		int status = 0;
		String query = "INSERT INTO freeforall.postprivileges (pid, username, privilege) " +
				"VALUES (?, ?, ?)";
		PreparedStatement pstmt = null;
		try {
			pstmt = conn.prepareStatement(query);
			pstmt.setInt(1, post);
			pstmt.setString(2, username);
			pstmt.setString(3, priv);
			status = pstmt.executeUpdate();
		} catch (SQLException e) {
			status = 0;
		}
		return status;
	}
}

```

## Description

The `PreparedStatement` opened in line 702 is never closed.
(Same findings as `mudetect-17`)


## Rule

Missing Call

## Response

### Confidence

90

### Types

Resource Leak

### Crash

False

### Snippet

```java
pstmt = conn.prepareStatement(query);
```

### Fix

```java
try {
	pstmt = conn.prepareStatement(query);
	pstmt.setInt(1, post);
	pstmt.setString(2, username);
	pstmt.setString(3, priv);
	status = pstmt.executeUpdate();
} catch (SQLException e) {
	status = 0;
} finally {
    try {
         if(pstmt != null) {
             pstmt.close();
         }
    } catch(SQLException se2) { }
}
```

### Description

The PreparedStatement is not being closed after its use. This object is a resource that should be closed explicitly to liberate system resources and to avoid memory leaks. If it is not closed, too many of these objects could be created and not correctly disposed of, ending up consuming all available resources, causing the system to slow down or even prevent other resources from being created when they are needed.

## Review

Found the expected misuse? **True**

