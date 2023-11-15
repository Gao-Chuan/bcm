## Repository

URL: https://github.com/google/closure-compiler.git

Commit: 43c245f0ff8d409e81e25687e69d34666b7cf26a~1

## API

java.util.Map

## Caller

```java
package com.google.javascript.jscomp;

class SimpleDefinitionFinder implements CompilerPass, DefinitionProvider {
  private final AbstractCompiler compiler;
  private final Map<Node, DefinitionSite> definitionSiteMap;
  private final Multimap<String, Definition> nameDefinitionMultimap;
  private final Multimap<String, UseSite> nameUseSiteMultimap;

  private class DefinitionGatheringCallback extends AbstractPostOrderCallback {
    @Override
    public void visit(NodeTraversal traversal, Node node, Node parent) {
      // Arguments of external functions should not count as name
      // definitions.  They are placeholder names for documentation
      // purposes only which are not reachable from anywhere.
      if (inExterns && NodeUtil.isName(node) && parent.getType() == Token.LP) {
        return;
      }

      Definition def =
          DefinitionsRemover.getDefinition(node, inExterns);
      if (def != null) {
        String name = getSimplifiedName(def.getLValue());
        if (name != null) {
          Node rValue = def.getRValue();
          if ((rValue != null) &&
              !NodeUtil.isImmutableValue(rValue) &&
              !NodeUtil.isFunction(rValue)) {

            // Unhandled complex expression
            Definition unknownDef =
                new UnknownDefinition(def.getLValue(), inExterns);
            def = unknownDef;
          }

          // TODO(johnlenz) : remove this stub dropping code if it becomes
          // illegal to have untyped stubs in the externs definitions.
          if (inExterns) {
            // We need special handling of untyped externs stubs here:
            //    the stub should be dropped if the name is provided elsewhere.

            List<Definition> stubsToRemove = Lists.newArrayList();
            String qualifiedName = node.getQualifiedName();

            // If there is no qualified name for this, then there will be
            // no stubs to remove. This will happen if node is an object
            // literal key.
            if (qualifiedName != null) {
              for (Definition prevDef : nameDefinitionMultimap.get(name)) {
                if (prevDef instanceof ExternalNameOnlyDefinition
                    && !jsdocContainsDeclarations(node)) {
                  String prevName = prevDef.getLValue().getQualifiedName();
                  if (qualifiedName.equals(prevName)) {
                    // Drop this stub, there is a real definition.
                    stubsToRemove.add(prevDef);
                  }
                }
              }

              for (Definition prevDef : stubsToRemove) {
                nameDefinitionMultimap.remove(name, prevDef);
              }
            }
          }

          nameDefinitionMultimap.put(name, def);
          definitionSiteMap.put(node,
                                new DefinitionSite(node,
                                                   def,
                                                   traversal.getModule(),
                                                   traversal.inGlobalScope(),
                                                   inExterns));
        }
      }

      if (inExterns && (parent != null) && NodeUtil.isExpressionNode(parent)) {
        String name = getSimplifiedName(node);
        if (name != null) {

          // TODO(johnlenz) : remove this code if it becomes illegal to have
          // stubs in the externs definitions.

          // We need special handling of untyped externs stubs here:
          //    the stub should be dropped if the name is provided elsewhere.
          // We can't just drop the stub now as it needs to be used as the
          //    externs definition if no other definition is provided.

          boolean dropStub = false;
          if (!jsdocContainsDeclarations(node)) {
            String qualifiedName = node.getQualifiedName();
            if (qualifiedName != null) {
              for (Definition prevDef : nameDefinitionMultimap.get(name)) {
                String prevName = prevDef.getLValue().getQualifiedName();
                if (qualifiedName.equals(prevName)) {
                  dropStub = true;
                  break;
                }
              }
            }
          }

          if (!dropStub) {
            // Incomplete definition
            Definition definition = new ExternalNameOnlyDefinition(node);
            nameDefinitionMultimap.put(name, definition);
            definitionSiteMap.put(node,
                                  new DefinitionSite(node,
                                                     definition,
                                                     traversal.getModule(),
                                                     traversal.inGlobalScope(),
                                                     inExterns));
          }
        }
      }
    }

  }

}

```

## Description

On line 196, an element is fetched from a map and immediately iterated over. If the map does not have an element for the key (or maps the key to `null`), the lookup returns `null` and the iteration attempt crashes.


## Rule

Missing Null Check

## Response 0

### Confidence

95

### Types

Missing Null Check

### Crash

True

### Snippet

```java
definitionSiteMap.put(node,new DefinitionSite(node,def,traversal.getModule(),traversal.inGlobalScope(),inExterns));
```

### Fix

```java
if(definitionSiteMap!=null) { 
definitionSiteMap.put(node,new DefinitionSite(node,def,traversal.getModule(),traversal.inGlobalScope(),inExterns));
}
```

### Description

The 'definitionSiteMap' could possibly be null since there is no null-check before calling its 'put' method. This could potentially lead to a null pointer exception.

## Response 1

### Confidence

95

### Types

Missing Null Check

### Crash

True

### Snippet

```java
nameDefinitionMultimap.put(name, def);
```

### Fix

```java
if(nameDefinitionMultimap!=null) {
nameDefinitionMultimap.put(name, def);
}
```

### Description

The 'nameDefinitionMultimap' could possibly be null since there is no null-check before calling its 'put' method. This could potentially lead to a null pointer exception.

## Review

Found the expected misuse? **False**

