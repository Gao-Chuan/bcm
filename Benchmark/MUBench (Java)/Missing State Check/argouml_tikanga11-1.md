## Repository

URL: https://github.com/MUBench/argouml.git

Commit: f787ce1018bbbf3e913a9987dd5d28a61e525cec

## API

java.util.Iterator

## Caller

```java
package org.argouml.persistence;

class ZargoFilePersister extends UmlFilePersister {
    private Project loadFromZargo(File file, ProgressMgr progressMgr)
        throws OpenException {

        Project p = ProjectFactory.getInstance().createProject(file.toURI());
        try {
            progressMgr.nextPhase();

            // Load .argo project descriptor
            ArgoParser parser = new ArgoParser();
            String argoEntry = getEntryNames(file, ".argo").iterator().next();
            parser.readProject(p, new InputSource(makeZipEntryUrl(toURL(file),
                    argoEntry).toExternalForm()));

            List memberList = parser.getMemberList();

            LOG.info(memberList.size() + " members");

            String xmiEntry = getEntryNames(file, ".xmi").iterator().next();
            MemberFilePersister persister = getMemberFilePersister("xmi");
            persister.load(p, makeZipEntryUrl(toURL(file), xmiEntry));
            
            // Load the rest
            List<String> entries = getEntryNames(file, null);
            for (String name : entries) {
                String ext = name.substring(name.lastIndexOf('.') + 1);
                if (!"argo".equals(ext) && !"xmi".equals(ext)) {
                    persister = getMemberFilePersister(ext);
                    LOG.info("Loading member with "
                            + persister.getClass().getName());
                    persister.load(p, openZipEntry(toURL(file), name));
                }
            }

            progressMgr.nextPhase();
            ThreadUtils.checkIfInterrupted();
            p.postLoad();
            return p;
        } catch (InterruptedException e) {
            return null;
        } catch (MalformedURLException e) {
            throw new OpenException(e);
        } catch (IOException e) {
            throw new OpenException(e);
        } catch (SAXException e) {
            throw new OpenException(e);
        }
    }
}

```

## Description

If the zargo file to be loaded contains no `.argo` file, this method will throw an exception at line 261.


## Rule

Missing State Check

## Response

### Confidence

90

### Types

Missing Null Check

### Crash

True

### Snippet

```java
String argoEntry = getEntryNames(file, ".argo").iterator().next();
String xmiEntry = getEntryNames(file, ".xmi").iterator().next();
```

### Fix

```java
Iterator<String> argoIterator = getEntryNames(file, ".argo").iterator();
String argoEntry = argoIterator.hasNext()? argoIterator.next() : null;
if(argoEntry == null) {
   // Handle error or throw exception
}
Iterator<String> xmiIterator = getEntryNames(file, ".xmi").iterator();
String xmiEntry = xmiIterator.hasNext()? xmiIterator.next() : null;
if(xmiEntry == null) {
   // Handle error or throw exception
}
```

### Description

The code assumes that an iterator created from getEntryNames(file, ".argo"/".xmi") will always have at least one element. If the iterator is empty, a NoSuchElementException will be thrown, which is not caught and may cause a crash. It is recommended to always check if the iterator has a next element by using the hasNext() method before calling next().

## Review

Found the expected misuse? **True**

