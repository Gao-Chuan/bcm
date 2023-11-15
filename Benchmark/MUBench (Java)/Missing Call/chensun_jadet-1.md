## Repository

URL: https://github.com/ms969/ChenSun

Commit: cf23b99a0c78596b5dc5bb25691736c9abd0a84d

## API

java.sql.PreparedStatement

https://github.com/openjdk/jdk/blob/3c6ffcadfec42c544c9b0d4188e50135f608b9db/src/java.sql/share/classes/java/sql/PreparedStatement.java#L63

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
	
	public static String getPost(Connection conn, String username, String boardName, 
			String regionName, int postNum) {
		String getOriginalPost = "";
		String getReplies = "";
		String postAndReplies = "";
		
		/*No joining of results because of redundancy of data returned*/
		if (boardName.equals("freeforall")) {
			getOriginalPost = "SELECT * FROM freeforall.posts " +
			"WHERE pid = ?";
			getReplies = "SELECT * FROM freeforall.replies " +
			"WHERE pid = ? ORDER BY dateReplied ASC";
		}
		else {
			getOriginalPost = "SELECT * FROM " + boardName + ".posts " +
			"WHERE pid = ? AND rname = ?";
			getReplies = "SELECT * FROM " + boardName + ".replies " +
			"WHERE pid = ? AND rname = ? ORDER BY dateReplied ASC";
		}
		
		PreparedStatement originalPost = null;
		ResultSet postResult = null;
		
		PreparedStatement replies = null;
		ResultSet repliesResult = null;
		
		boolean sqlex = false;
		try {
			originalPost = conn.prepareStatement(getOriginalPost);
			replies = conn.prepareStatement(getReplies);
			originalPost.setInt(1, postNum);
			replies.setInt(1, postNum);
			if (!boardName.equals("freeforall")) {
				originalPost.setString(2, regionName);
				replies.setString(2, regionName);
			}
			
			postResult = originalPost.executeQuery();
			if (postResult.next()) { /*Only expect one post result*/
				//Make sure the checksum is correct
				if (!Arrays.equals(
						Hash.generateChecksum((SharedKeyCrypto.decrypt(postResult.getString("content"))).getBytes("UTF8")),
						CryptoUtil.decode(postResult.getString("checksum")))) {
					postAndReplies +=
							"print ----- Post# " + postNum + "[" + postResult.getString("postedBy") + "]----- " +
									postResult.getTimestamp("datePosted").toString() + ";print \t" +
									"Content could not be fetched -- Integrity Failure!" + ";";
				}
				else {
					postAndReplies += 
						"print ----- Post# " + postNum + "[" + postResult.getString("postedBy") + "]----- " +
						postResult.getTimestamp("datePosted").toString() + ";print \t" +
						SharedKeyCrypto.decrypt(postResult.getString("content")) + ";";
				}
				
				repliesResult = replies.executeQuery();
				while (repliesResult.next()) { //Print out all replies
					//for each reply, make sure the checksum is correct.
					if(!Arrays.equals(
							Hash.generateChecksum((SharedKeyCrypto.decrypt(repliesResult.getString("content"))).getBytes("UTF8")),
							CryptoUtil.decode(repliesResult.getString("checksum")))) {
						postAndReplies += "print ----- Reply[" + repliesResult.getString("repliedBy") + "] ----- " +
						repliesResult.getTimestamp("dateReplied").toString() + ";print \t" +
						"Content could not be fetched -- Integrity Failure!" + ";";
					}
					else {
						postAndReplies += "print ----- Reply[" + repliesResult.getString("repliedBy") + "] ----- " +
						repliesResult.getTimestamp("dateReplied").toString() + ";print \t" +
						SharedKeyCrypto.decrypt(repliesResult.getString("content")) + ";";
					}
				}
			}
			// if there's no postResult, the post DNE.
		}
		catch (SQLException e) {
			e.printStackTrace();
			sqlex = true;
		} catch (UnsupportedEncodingException e) {
			// This should not happen.
		}
		if (postAndReplies.equals("") && !sqlex) {
			return "print Error: Post does not exist. Refresh. If the problem persists, contact an admin.";
		}
		else if (sqlex) {
			return "print Error: Database error while querying post and replies. Contact an admin.";
		}
		else return postAndReplies;
	}
}

```

## Description

The `PreparedStatement` opened in line 635 is never closed.
(Same finding as `mudetect-19` and `mudetectxp-7`)


## Rule

Missing Call

## Response

### Confidence

100

### Types

Missing Call

### Crash

False

### Snippet

```java
PreparedStatement originalPost = null;
ResultSet postResult = null;
PreparedStatement replies = null;
ResultSet repliesResult = null;
```

### Fix

```java
Finally { 
	if (originalPost != null) { 
		try { 
			originalPost.close(); 
		} catch (SQLException se) { 
			se.printStackTrace(); 
		} 
	} 
	if (postResult != null) { 
		try { 
			postResult.close(); 
		} catch (SQLException se) { 
			se.printStackTrace(); 
		} 
	} 
	if (replies != null) { 
		try { 
			replies.close(); 
		} catch (SQLException se) { 
			se.printStackTrace(); 
		} 
	} 
	if (repliesResult != null) { 
		try { 
			repliesResult.close(); 
		} catch (SQLException se) { 
			se.printStackTrace(); 
		} 
	} 
}
```

### Description

The PreparedStatement and ResultSet objects are not closed. If any object opens system resources such as files, DB connections, it is recommended to call the 'close()' method after finished working with them, to release these resources back to the system and prevent memory leaks.

## Review

Found the expected misuse? **True**

