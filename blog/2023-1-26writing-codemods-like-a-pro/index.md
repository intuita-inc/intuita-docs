---
slug: writing-codemods-like-a-pro
title: Writing Codemods Like A Pro
authors: [mohab]
tags: [codemods, tutorial, advanced]
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Writing Codemods Like a Pro

Writing codemods using imperative methods (like JSCodeShift for JS/TS) can be extremely powerful for code transformation. However, such codemods can be difficult to create, especially by new codemod authors.

In this article, we'll take what we've learned about ASTs and building simple codemods to tackle a real-world problem.

<!--truncate-->

 ---


## Prerequisites

- **Understanding ASTs -** This article requires a basic understanding of how to inspect, traverse, and manipulate ASTs. If you need a refresher, check out [our guide on understanding ASTs](#).
- **Basic Understanding of Codemods -** This article requires a basic understanding of writing simple codemods. If you're unfamiliar with writing codemods, check out [our tutorial on writing your first codemod](#).


## Overview

Throughout this document, we will break down some of the thought process a codemod guru, like [Christoph Nakazawa](https://github.com/cpojer), uses to make useful codemods.

By the end of this tutorial, you will learn:

- How to write a codemod that solves a real-world problem.
- Usage of more advanced AST manipulation techniques.
- New methods and tools that make your codemod development more efficient.

Let's learn by example together!




## Scenario

Before the emergence of ES6, JS codebases heavily relied on `var` for variable declarations.

Due to the issues with `var`'s scope issues, `let` and `const` declarations were introduced to put an end to the shortcomings of `var`.

However, even after the introduction of `let` and `const`, there are still a lot of codebases that haven't been migrated to use the new declaration types. Refactoring those codebases can be a tedious process. To add, resorting to search-and-replace methods aren't applicable in this scenario, as there are edge cases (which we discuss below) that aren't possible to cover with mere find-and-replace operations.

In this example, we will take a look at the JS Codemod [`no-vars`](https://github.com/cpojer/js-codemod/blob/master/transforms/no-vars.js).

The `no-vars` codemod, has been developed to automatically refactor codebases to use `const` and `let` declarations instead of `var` wherever possible.

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

### Considering Code Variety & Edge Cases

If you are new to codemod development, you might get tricked into thinking that developing a transformation for this scenario is simpler than it is.

One might think that it is safe to simply:
- Find all variable declarations.
- Filter by variable declarations where `kind` is `var`.
- Replace found declarations with `const`.

However, this would probably lead to breaking your code in most cases.

A codemod pro adopts a thought process that is very careful with having any assumptions about code. That's why you should always consider all possible varieties and edge cases of the code pattern for your use case.


### Planning a Solution to Handle Code Variety & Edge Cases

To start handling code variety and edge cases, let's briefly note the possible cases where a `var` might be declared:
- `var` is declared and not mutated
- `var` is declared and mutated
- `var` is declared and initialized as a loop index
- `var` is declared inside a loop

If you take a closer look, you'd start noticing a lot of possible problems with simply replacing each `var` with `const`, as the use of `const` in some of the previously stated cases would break our code.

```js
for (var i = 0; i < 5; i++) { // Replacing var with const here would cause faulty code
  // ...
}
```

```js
var hello = "hello world"; // Replacing var here breaks the code as the hello variable is mutated
hello = "hello universe";
```

Simply replacing all var instances with const would break the code if:

- `var` is a loop index declaration
- `var` is a mutated variable
- `var` is declared or initialized within a loop

Therefore we should avoid replacing such declarations with const and implement fallbacks for such cases.

So, our approach should be:

- In cases where we can't replace `var` with `const`, we should replace `var` with `let` as a fallback.
- Safely transform all other occurrences of `var` into `const`.


With that in mind, now let's take a look at a step-by-step process of how the codemod pro [Christoph Nakazawa](https://github.com/cpojer) tackles those cases in his [no-vars](https://github.com/cpojer/js-codemod/blob/master/transforms/no-vars.js) transform.

---

## Developing the Codemod

To get started with writing our codemod, let's start by opening up [ASTExplorer](https://astexplorer.net/).

To follow along, set your transform setting to `jscodeshift`. Your parser setting should then automatically change to `recast`.


### Example Input

To start developing our codemod, let's consider this example input code which has an instance of:
- A non-mutated `var`
- A mutated `var`
- A `var` used for loop index declaration
- A `var` declared inside a loop

We'll use each of those cases to cover the possible edge cases that can occur; thereby, verifying if our codemod passes the possible code varieties.

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

For reference, here is our end goal. We can verify if our codemod is correct if it can transform the previous input code to the required output illustrated below.

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

With that being said, here is the transform that a pro like [Christoph Nakazawa](https://github.com/cpojer) wrote to solve this challenge.

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

<TabItem value="helper-functions" label="w/ Helper Functions" default>

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
Now let's break down the thought process of building such a transform.


#### Getting Declarations
Firstly, we get all the `var` declarations.

```js
const updatedAnything = root.find(j.VariableDeclaration).filter(
    dec => dec.value.kind === 'var'    // getting all var declarations
  )
```

#### Checking Declaration Parent

Then we check if the `var` conforms to ***any*** of those cases:
- `var` is inside a closure
- `var` is declared twice
- `var` is a function declaration that might be hoisted

If any of those cases occur, we refrain from transforming the `var` declaration to `const`. Rather, we convert it to a `let` declaration.

```js
.filter(declaration => {
    return declaration.value.declarations.every(declarator => {
      // checking if the var is inside a closure
      // or declared twice or is a function declaration that might be hoisted
      return !isTruelyVar(declaration, declarator);
    });
```

#### Checking Parent Loops
Then, for each `var` inside a loop, we check if:
- The `var` is declared as the iterable object of a For...of/in loop
- If variable is mutated inside the loop.

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
        // This transforms the declaration type to let
      declaration.value.kind = 'let';
    } else {
        // Else, var is safe to be converted to const
        // This transforms the declaration type to const
      declaration.value.kind = 'const';
    }
  }).size() !== 0;
```

In the case of (1) the variable being mutated or (2) the declarator not being initialized and the parent loop being a For...of/in loop, then we resort to replacing `var` with `let`.

Otherwise, we can safely replace `var` with `const`.

> Note here that while making this codemod, we should always consider having fallbacks for undesirable cases. The codemod developer here chose `let` as a fallback when `const` is not applicable, rather than keeping the declaration as `var`, as the use of `let` is arguably better.

#### Returning the Manipulated AST
Finally, we replace the source AST with the manipulated AST after applying our transforms.

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
- Always make sure you aren't making wishful assumptions in your codemod cases.
- Design fallbacks for undesired scenarios within your codemods.
- And most importantly, get active within the codemod community! At Intuita, we're keen on building a [community haven for all the codemod enthusiasts out there](https://github.com/intuita-inc).


### Next Steps
After taking your first steps with writing your first codemod that solves a real-world problem, now you're ready to start taking your development process to the next level.

In the upcoming tutorial, we will cover:
- Integrating declarative codemod engines like [JARVIS](https://rajasegar.github.io/jarvis/) into your workflow.
- Diversifying your codemod development toolset.
- Writing tests for your codemods.

[Let's get started! &rarr;](/blog/declarative-codemod-engines)