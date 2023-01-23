---
sidebar_position: 1
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Write Codemods Like a Pro

This document will guide you through the thought framework of getting started to write codemods like a pro.


## Overview

In this tutorial, we start taking the first steps towards writing codemods that solve real problems.

We will showcase a codemod that you would see in the wild in a popular open-source project, go over how this codemod was built, and the thought process of a codemod developer.

We suggest you follow this tutorial using [ASTExplorer](https://astexplorer.net/) w/ JSCodeShift.

If you prefer testing on your local development environment, check our guide for [configuring your local environment for codemod development](/docs/).

---

## Prerequisites

### Understanding ASTs
This tutorial requires understanding of Abstract Syntax Trees (ASTs). You should be able to inspect, traverse, & manipulate ASTs. If you're not familiar with ASTs, make sure to check out [our guide on understanding ASTs first](/docs/).

### Writing simple codemods
Before getting into this tutorial, make sure you're familiar with writing simple codemods. If you're not, make sure to go over [our tutorial for writing your first codemod](/docs/), first.

---


## Scenario

In this example, we will take a look at the JS Codemod [`no-vars`](https://github.com/cpojer/js-codemod/blob/master/transforms/no-vars.js).

The `no-vars` codemod, converts all occurrences of `var` and `let` declarations to `const` declarations.

<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
// red start
var exampleVariable = "hello world"; 
// red end
```

</div>

<div class="flex-column">

```jsx  title="After Change"
// green start
const exampleVariable = "hello world"; 
// green end
```

</div>
</div>

### Considerations

If you are new to codemod development, you might get tricked into thinking that developing a transformation for this scenario is simpler than it is.

One might think that it is safe to simply:
- Find all variable declarations.
- Filter by variable declarations where `kind` is `var`.
- Replace found declarations with `const`.

However, this would probably lead to breaking your code in most cases.

The first shift in thought process to becoming a codemod pro is realizing that ***codemod design is a very delicate process***.

A codemod pro's thought process would usually be very careful before having any assumptions about code.


### Your First Steps To Being a Codemod Pro (Thinking 10 Steps Ahead)

If you take a closer look, you'd start noticing a lot of possible problems with simply replacing each `var` with `const`.

Consider scenarios where the variable declaration kind is in fact `var`, but the variable is either:
- An loop's index declaration.
- A mutated variable.

```js
for (var i = 0; i < 5; i++) { // Replacing var with const here would cause faulty code
  // ...
}
```

```js
var hello = "hello world"; // Replacing var here breaks the code as the hello variable is mutated
hello = "hello universe";
```

A more experienced codemod developer would consider such cases and think about:
- Replacing var declarations that are mutated with const.
- Avoid replacing for loop declarations with const (except For...of/in loops).
- Using `let` as a fallback for `var` declarations in such cases.

With that in mind, now let's take a look at a step-by-step process of how the codemod pro [Christoph Nakazawa](https://github.com/cpojer) tackles those cases in his [`no-vars`](https://github.com/cpojer/js-codemod/blob/master/transforms/no-vars.js) transform.


---

## Developing the Codemod


### Example Input

```js
var notMutatedVar = "definitely not mutated";
var mutatedVar = "yep, i'm mutated";

for (var i = 0; i < 5; i++) {
  mutatedVar = "foo";
  var anotherInsideLoopVar = "should i be changed?";
}

for (var x in text) {
  text += x + " ";
}
```
### Required Output
```js
const notMutatedVar = "definitely not mutated";
let mutatedVar = "yep, i'm mutated";

for (let i = 0; i < 5; i++) {
  mutatedVar = "foo";
  const anotherInsideLoopVar = "should i be changed?";
}

for (const x in text) {
  text += x + " ";
}
```

### The Transform

<Tabs>
<TabItem value="transform" label="The Transform" default>

```jsx  title="The Transform"
const updatedAnything = root.find(j.VariableDeclaration).filter(
    dec => dec.value.kind === 'var'
  ).filter(declaration => {
    return declaration.value.declarations.every(declarator => {
      return !isTruelyVar(declaration, declarator);
    });
  }).forEach(declaration => {
    const forLoopWithoutInit = isForLoopDeclarationWithoutInit(declaration);
    if (
      declaration.value.declarations.some(declarator => {
        return (!declarator.init && !forLoopWithoutInit) || isMutated(declaration, declarator);
      })
    ) {
      declaration.value.kind = 'let';
    } else {
      declaration.value.kind = 'const';
    }
  }).size() !== 0;
  return updatedAnything ? root.toSource() : null;
```

</TabItem>

<TabItem value="helper-functions" label="Helper Functions" default>

```jsx  title="Code with Helper Functions"
export const parser = 'babel'

export default function transformer(file, api, options) {
  const j = api.jscodeshift;

  const root = j(file.source);

  const TOP_LEVEL_TYPES = [
    'Function',
    'FunctionDeclaration',
    'FunctionExpression',
    'ArrowFunctionExpression',
    'Program',
  ];
  const FOR_STATEMENTS = [
    'ForStatement',
    'ForOfStatement',
    'ForInStatement',
  ];
  const getScopeNode = blockScopeNode => {
    let scopeNode = blockScopeNode;
    let isInFor = FOR_STATEMENTS.indexOf(blockScopeNode.value.type) !== -1;
    while (TOP_LEVEL_TYPES.indexOf(scopeNode.node.type) === -1) {
      scopeNode = scopeNode.parentPath;
      isInFor = isInFor || FOR_STATEMENTS.indexOf(scopeNode.value.type) !== -1;
    }
    return {scopeNode, isInFor};
  };
  const findFunctionDeclaration = (node, container) => {
    while (node.value.type !== 'FunctionDeclaration' && node !== container) {
      node = node.parentPath;
    }
    return node !== container ? node : null;
  };
  const isForLoopDeclarationWithoutInit = declaration => {
    const parentType = declaration.parentPath.value.type;
    return parentType === 'ForOfStatement' || parentType === 'ForInStatement';
  };

  const extractNamesFromIdentifierLike = id => {
    if (!id) {
      return [];
    } else if (id.type === 'ObjectPattern') {
      return id.properties.map(
        d => (d.type === 'SpreadProperty' ? [d.argument.name] : extractNamesFromIdentifierLike(d.value))
      ).reduce((acc, val) => acc.concat(val), []);
    } else if (id.type === 'ArrayPattern') {
      return id.elements.map(extractNamesFromIdentifierLike).reduce((acc, val) => acc.concat(val), []);
    } else if (id.type === 'Identifier') {
      return [id.name];
    } else if (id.type === 'RestElement') {
      return [id.argument.name];
    } else {
      return [];
    }
  };
  const getDeclaratorNames = (declarator) => {
    return extractNamesFromIdentifierLike(declarator.id);
  };
  const isIdInDeclarator = (declarator, name) => {
    return getDeclaratorNames(declarator).indexOf(name) !== -1;
  };
  const getLocalScope = (scope, parentScope) => {
    const names = [];
    while (scope !== parentScope) {
      if (Array.isArray(scope.value.body)) {
        scope.value.body.forEach(node => {
          if (node.type === 'VariableDeclaration') {
            node.declarations.map(getDeclaratorNames).forEach(dNames => {
              dNames.forEach(name => {
                if (names.indexOf(name) === -1) {
                  names.push(name);
                }
              });
            });
          }
        });
      }
      if (Array.isArray(scope.value.params)) {
        scope.value.params.forEach(id => {
          extractNamesFromIdentifierLike(id).forEach(name => {
            if (names.indexOf(name) === -1) {
              names.push(name);
            }
          });
        });
      }
      scope = scope.parentPath;
    }
    return names;
  };
  const hasLocalDeclarationFor = (nodePath, parentScope, name) => {
    return (
      getLocalScope(nodePath, parentScope).indexOf(name) !== -1
    );
  };

  const isTruelyVar = (node, declarator) => {
    const blockScopeNode = node.parentPath;
    const {scopeNode, isInFor} = getScopeNode(blockScopeNode);

    // if we are in a for loop of some kind, and the variable
    // is referenced within a closure, revert to `var`
    // It would be safe to do the conversion if you can verify
    // that the callback is run synchronously
    const isUsedInClosure = (
      isInFor &&
      j(blockScopeNode).find(j.Function).filter(
        functionNode => (
          j(functionNode).find(j.Identifier).filter(
            id => isIdInDeclarator(declarator, id.value.name)
          ).size() !== 0
        )
      ).size() !== 0
    );

    // if two attempts are made to declare the same variable,
    // revert to `var`
    // TODO: if they are in different block scopes, it may be
    //       safe to convert them anyway
    const isDeclaredTwice = j(scopeNode)
      .find(j.VariableDeclarator)
      .filter(otherDeclarator => {
        return (
          otherDeclarator.value !== declarator &&
          getScopeNode(otherDeclarator).scopeNode === scopeNode &&
          getDeclaratorNames(otherDeclarator.value).some(
            name => isIdInDeclarator(declarator, name)
          )
        );
      }).size() !== 0;

    return isUsedInClosure || isDeclaredTwice || j(scopeNode)
      .find(j.Identifier)
      .filter(n => {
        if (!isIdInDeclarator(declarator, n.value.name)) {
          return false;
        }
        // If the variable is used in a function declaration that gets
        // hoisted, it could get called early
        const functionDeclaration = findFunctionDeclaration(n, scopeNode);
        const isCalledInHoistedFunction = (
          functionDeclaration &&
          j(scopeNode)
            .find(j.Identifier)
            .filter(n => {
              return (
                n.value.name === functionDeclaration.value.id.name &&
                n.value.start < declarator.start
              );
            }).size() !== 0
        );
        if (isCalledInHoistedFunction) {
          return true;
        }
        const referenceScope = getScopeNode(n.parent).scopeNode;
        if (
          referenceScope === scopeNode ||
          !hasLocalDeclarationFor(n, scopeNode, n.value.name)
        ) {
          // if the variable is referenced outside the current block
          // scope, revert to using `var`
          const isOutsideCurrentScope = (
            j(blockScopeNode).find(j.Identifier).filter(
              innerNode => innerNode.node.start === n.node.start
            ).size() === 0
          );

          // if a variable is used before it is declared, revert to
          // `var`
          // TODO: If `isDeclaredTwice` is improved, and there is
          //       another declaration for this variable, it may be
          //       safe to convert this anyway
          const isUsedBeforeDeclaration = (
            n.value.start < declarator.start
          );

          return (
            isOutsideCurrentScope ||
            isUsedBeforeDeclaration
          );
        }
      }).size() > 0;
  };

  /**
   * isMutated utility function to determine whether a VariableDeclaration
   * contains mutations. Takes an optional VariableDeclarator node argument to
   * return only whether that specific Identifier is mutated
   *
   * @param {ASTPath} node VariableDeclaration path
   * @param {ASTNode} [declarator] VariableDeclarator node
   * @return {Boolean}
   */
  const isMutated = (node, declarator) => {
    const scopeNode = node.parent;

    const hasAssignmentMutation = j(scopeNode)
      .find(j.AssignmentExpression)
      .filter(n => {
        return extractNamesFromIdentifierLike(n.value.left).some(name => {
          return isIdInDeclarator(declarator, name);
        });
      }).size() > 0;

    const hasUpdateMutation = j(scopeNode)
      .find(j.UpdateExpression)
      .filter(n => {
        return isIdInDeclarator(declarator, n.value.argument.name);
      }).size() > 0;

    return hasAssignmentMutation || hasUpdateMutation;
  };

  const updatedAnything = root.find(j.VariableDeclaration).filter(
    dec => dec.value.kind === 'var'
  ).filter(declaration => {
    return declaration.value.declarations.every(declarator => {
      return !isTruelyVar(declaration, declarator);
    });
  }).forEach(declaration => {
    const forLoopWithoutInit = isForLoopDeclarationWithoutInit(declaration);
    if (
      declaration.value.declarations.some(declarator => {
        return (!declarator.init && !forLoopWithoutInit) || isMutated(declaration, declarator);
      })
    ) {
      declaration.value.kind = 'let';
    } else {
      declaration.value.kind = 'const';
    }
  }).size() !== 0;
  return updatedAnything ? root.toSource() : null;
}
```

</TabItem>

</Tabs>



### Breaking it Down
Now let's break down you you build such a transform.


#### Getting Declarations
Firstly, we get all the `var` declarations.

```js
const updatedAnything = root.find(j.VariableDeclaration).filter(
    dec => dec.value.kind === 'var'    // getting all var declarations
  )
```

#### Checking Declaration Parent

Then we check if the var is inside a closure, declared twice, or is a function declaration which might be hoisted.

If any of those cases occur, we refrain from transforming the `var` declaration to `const`, rather we convert it to a `let` declaration.

```js
.filter(declaration => {
    return declaration.value.declarations.every(declarator => {
      // checking if the var is inside a closure
      // or declared twice or is a function declaration which might be hoisted
      return !isTruelyVar(declaration, declarator);
    });
```

#### Checking Parent Loops
Then, we check if the `var` declaration is within either:
- A For...of/in loop
- If variable is mutated.

```js
.forEach(declaration => {
    // True if parent path is either a For...of or in loop
    const forLoopWithoutInit = isForLoopDeclarationWithoutInit(declaration);
    if (
      declaration.value.declarations.some(declarator => {
        // If declarator is not initialized and parent loop is initialized
        // or
        // If var is mutated
        return (!declarator.init && !forLoopWithoutInit) || isMutated(declaration, declarator);
      })
    ) {
        // In either one of the previous cases, we fall back to using let instead of const
      declaration.value.kind = 'let';
    } else {
        // Else, var is safe to be converted to const
      declaration.value.kind = 'const';
    }
  }).size() !== 0;
```

In the case of (1) the variable being mutated or (2) the declarator not being initialized and parent loop being a For...of/in loop, then we resort to replacing `var` with `let`.

Otherwise, we can safely replace `var` with `const`.

#### Applying the Transform
Finally, we replace the AST with the manipulated AST.

```js
return updatedAnything ? root.toSource() : null;
```


---

## Wrapping Up

After applying this transform, we successfully get our desired code output.

```js
const notMutatedVar = "definitely not mutated";
let mutatedVar = "yep, i'm mutated";

for (let i = 0; i < 5; i++) {
  mutatedVar = "foo";
  const anotherInsideLoopVar = "should i be changed?";
}

for (const x in text) {
  text += x + " ";
}
```

### Takeaways
- Being an experienced codemod developer requires careful thought while designing your codemod.
- Always make sure you aren't making wishful assumptions in your codemod cases.
- Design fallbacks for undesired scenarios within your codemods.
- And most importantly, get active within the codemod community! At Intuita, we're keen on building a [community haven for all the codemod enthusiasts out there](https://github.com/intuita-inc).