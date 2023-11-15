## Repository

URL: https://github.com/ms969/ChenSun

Commit: cf23b99a0c78596b5dc5bb25691736c9abd0a84d

## API

java.sql.ResultSet

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

The `ResultSet` fetched in line 644 is never closed.
(This is the same findings as `jadet-4` and `mudetect-19`)


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
ResultSet postResult = null;
ResultSet repliesResult = null;
try {
	originalPost = conn.prepareStatement(getOriginalPost);
	replies = conn.prepareStatement(getReplies);
	postResult = originalPost.executeQuery();
	repliesResult = replies.executeQuery();
} catch (SQLException e) {
	e.printStackTrace();
} catch (UnsupportedEncodingException e) {
	// This should not happen.
}
```

### Fix

```java
ResultSet postResult = null;
ResultSet repliesResult = null;
try {
	originalPost = conn.prepareStatement(getOriginalPost);
	replies = conn.prepareStatement(getReplies);
	postResult = originalPost.executeQuery();
	repliesResult = replies.executeQuery();
} catch (SQLException e) {
	e.printStackTrace();
} catch (UnsupportedEncodingException e) {
	// This should not happen.
} finally {
	if (postResult != null) postResult.close();
	if (repliesResult != null) repliesResult.close();
	if (originalPost != null) originalPost.close();
	if (replies != null) replies.close();
}
```

### Description

The ResultSet and PreparedStatement are resources which should be closed after their use to prevent possible resource leaks. Although the JVM can sometimes recover resources when they're no longer in use, it's best to explicitly close them. The 'try' block should be followed by a 'finally' block where ResultSet and PreparedStatement should be closed.

## Review

Found the expected misuse? **True**

