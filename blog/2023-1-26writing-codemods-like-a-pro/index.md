---
slug: writing-codemods-like-a-pro
title: Writing Codemods Like A Pro
authors: [mohab]
tags: [codemods, tutorial, advanced]
# Display h2 to h5 headings
toc_min_heading_level: 2
toc_max_heading_level: 3
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Writing Codemods Like a Pro


Writing codemods using imperative methods (like JSCodeShift for JS/TS) can be extremely powerful for code transformation. However, such codemods can be difficult to create, especially for new codemod authors.

In this article, we'll use what we've learned about ASTs and simple codemod development to tackle a real-world problem.


<!--truncate-->

---

## Prerequisites

- **Understanding ASTs -** This article requires a basic understanding of how to inspect,
traverse, and manipulate ASTs.
- **Basic Understanding of Codemods -** This article requires a basic understanding of writing simple codemods. If you're unfamiliar with writing codemods, check out [our tutorial on writing your first codemod](#).

## Overview

Throughout this document, we will break down some of the thought process a codemod guru, like [Christoph Nakazawa](https://github.com/cpojer), uses to make useful codemods.

By the end of this tutorial, you will learn:

- How to write a codemod that solves a real-world problem.
- Usage of more advanced AST manipulation techniques.
- New methods and tools that make your codemod development more efficient.

Let's learn by example together!

## Problem

Before the emergence of ES6, JS codebases heavily relied on `var` for variable declarations.

Due to the issues with `var`'s scope issues, `let` and `const` declarations were introduced to put an end to the shortcomings of `var`.

However, even after the introduction of `let` and `const`, there are still a lot of codebases that haven't been migrated to use the new declaration types. Refactoring those codebases can be a tedious process. To add, resorting to search-and-replace methods aren't applicable in this scenario, as there are edge cases (which we discuss below) that aren't possible to cover with mere find-and-replace operations.

In this example, we will take a look at the codemod `[no-vars](https://github.com/cpojer/js-codemod/blob/master/transforms/no-vars.js)`.

The `no-vars` codemod has been developed to automatically refactor codebases to use `const` and `let` declarations instead of `var` wherever possible.


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

### Planning Our Codemod

If you are new to codemod development, you might get tricked into thinking that developing a transformation for this scenario is simpler than it is.

One might think that it is safe to simply:

- Find all variable declarations.
- Filter by variable declarations where `kind` is `var`.
- Replace found declarations with `const`.

However, this would probably lead to breaking your code in most cases.

Let’s consider this sample code snippet which covers some of the possible cases of how a `var` might be declared.

```jsx
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

As we can see, this code covers the following cases:

- `var` is declared and not mutated
- `var` is declared and mutated
- `var` is declared and initialized as a loop index
- `var` is declared inside a loop

Such cases can also be referred to as **patterns**.

Now, take a moment to try to find out which cases would break the code if we were to transform `var` into `const`.

Let’s see if you could point them all out. Here’s a brief summary of different occurring patterns and the corresponding safe transform we can apply for each one:

1. `var` is a loop index declaration
2. `var` is a mutated variable
3. `var` is in a loop and mutated
4. Global or local non-mutated `var`
5. `var` is declared twice
6. `var` is hoisted
7. `var` is declared in a loop and referenced inside a closure




<!-- table goes here-->


#### #1 `var` is a loop index declaration

<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
// red start
for (var i = 0; i < 5; i++)
// red end
```

</div>

<div class="flex-column">

```jsx  title="After Change"
// green start
for (let i = 0; i < 5; i++)
// green end
```

</div>
</div>





#### #2 `var` is a mutated variable

<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
// red start
var x = 1;
// red end
x=1;
```

</div>

<div class="flex-column">

```jsx  title="After Change"
// green start
let x = 1;
// green end
x = 2;
```

</div>
</div>




#### #3 `var`  is in a loop and mutated

<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
// red start
for (var i = 0; i < 5; i++) {
  var x = "foo";
  // red end
  x = “bar”;
}
```

</div>

<div class="flex-column">

```jsx  title="After Change"
// green start
for (let i = 0; i < 5; i++) {
  let x = "foo";
  // green end
  x = “bar”;
}
```

</div>
</div>




#### #4 Global or local non-mutated `var`

<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
// highlight-next-line
var x = “foo”;
```

</div>

<div class="flex-column">

```jsx  title="After Change"
// highlight-next-line
const x = “foo”; // No change
```

</div>
</div>



#### #5 `var` is declared twice

<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
// highlight-next-line
var x;
// highlight-next-line
var x;
```

</div>

<div class="flex-column">

```jsx  title="After Change"
// highlight-next-line
var x; // No change
// highlight-next-line
var x; // No change
```

</div>
</div>





#### #6 `var` is hoisted

<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
x = 5;
// highlight-next-line
var x;
```

</div>

<div class="flex-column">

```jsx  title="After Change"
x = 5;
// highlight-next-line
var x; // No change
```

</div>
</div>





#### #7 `var` is declared in a loop and referenced inside a closure

<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
for (var i = 0; i<5; i++){
  // highlight-next-line
 var a = "hello";
  function myFunction() {
  a = "world";
  return a;
 }
}
```

</div>

<div class="flex-column">

```jsx  title="After Change"
for (let i = 0; i<5; i++){
  // highlight-next-line
 var a = "hello"; // No change
  function myFunction() {
  a = "world";
  return a;
 }
}
```

</div>
</div>




<!-- table ends here -->





Now that we have a concrete list of possible patterns and their corresponding suitable actions, let's prepare a test case to validate if the codemod we write successfully satisfies our plan.

### Before State

```jsx
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

### After State

```jsx
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

We can verify if our codemod is correct if it can transform the previous “Before” state of the code to the “After” state illustrated above.

With this plan in mind, let's take a look at a step-by-step process of how the codemod pro [Christoph Nakazawa](https://github.com/cpojer) puts it into action in his [no-vars](https://github.com/cpojer/js-codemod/blob/master/transforms/no-vars.js) transform.

---

## Developing the Codemod

Now that we’ve done the prep work in the previous section, we can confidently start writing our codemod.

Our workflow for writing the codemod will be as follows:

1. Detect code patterns
2. Transform patterns

To get started with writing our codemod, let's start by opening up [ASTExplorer](https://astexplorer.net/).

To follow along, set your transform setting to `jscodeshift`. Your parser setting should then automatically change to `recast`.

Now that your environment is set up, let's start following our workflow.

### #1 Detect Code Patterns

To detect our target code patterns we will:

1. **Find** all nodes that conform to a broad rule which encompasses all code patterns. This includes both, patterns that should and should not be detected.
2. Then, we extract (**filter**) the nodes we want to modify.

1.1 **Finding Nodes**

In our case, we can start first by finding all variable declarations. To do this, we can insert a simple `var` declaration snippet into ASTExplorer, and use the tree explorer to find the structure name for variable declarations.

![finding var declaration in ast explorer](/img/blog/writing-codemods-like-a-pro/1.png)

Now we know that we can find all variable declarations by finding `j.VariableDeclaration` instances as shown below.

```jsx
const updatedAnything = root.find(j.VariableDeclaration)
```

:::tip
It’s good practice to leverage tools like AST Explorer and TS AST Viewer within your workflow to efficiently analyze ASTs of your identified code patterns.
:::

**1.2 Extract The Nodes To Be Modified**

Now that we’ve captured all variable declarations, we can now start filtering them based on the patterns we’ve identified while [planning our codemod](#planning-our-codemod).

In the previous step, we targeted all variable declarations, which include `var`, `let`, and `const` declarations. So, let’s first start filtering for `var` declarations only.

We do this by using JSCodeshift’s `filter` method as shown below:

```jsx
const updatedAnything = root.find(j.VariableDeclaration).filter(
    // highlight-next-line
    dec => dec.value.kind === 'var'    // getting all var declarations
  )
```

Now that we’re targeting only `var` declarations, let’s rule out all `var` declarations that we cannot transform into `let` or `const`.

To do so, we will call a second filter which calls the custom helper function `isTruelyVar`. This filter checks for every `var` if it conforms to ***any*** of such cases:

- `var` is declared in a loop and referenced inside a closure
- `var` is declared twice
- `var` is hoisted

If any of those cases occur, we refrain from transforming the `var` declaration at all. Rather, we fall back to a `var` declaration.

```jsx
.filter(declaration => {
    return declaration.value.declarations.every(declarator => {
      // checking if the var is inside a closure
      // or declared twice or is a function declaration that might be hoisted
      // highlight-next-line
      return !isTruelyVar(declaration, declarator);
    });
```

After ruling out all non-transformable `var` occurrences, we can now apply the final filter to determine whether the remaining `var` occurrences will be transformed into `let` or `const`.

To do so, for each `var` inside a loop, we check if:

- The `var` is declared as the iterable object of a For...of/in loop
- If a variable is mutated inside the loop.

```jsx
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
    )
```

This filter allows us to pinpoint 2 possible cases:

1. The variable is mutated
2. The variable is not initialized and the parent loop is a For...of/in loop

### #2 Transforming the Nodes

With the 2 possible cases that we’ve identified, now we can determine whether we will transform the `var` into either a `let` or `const` declaration

In the case of the occurrence of either case (1) or (2), we resort to replacing `var` with `let`.

Otherwise, we can safely replace `var` with `const`.

```jsx
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
        // highlight-next-line
      declaration.value.kind = 'let';
    }else {
        // Else, var is safe to be converted to const
        // highlight-next-line
      declaration.value.kind = 'const';
    }
  }).size() !== 0;
// highlight-next-line
return updatedAnything ? root.toSource() :null; //replacing the source AST with the manipulated AST after applying our transforms
```

:::note
Note here that while writing codemods, we should always consider having 
fallbacks for undesirable cases. The codemod developer here chose `let` as a fallback when `const` is not applicable, rather than keeping the declaration as `var`, as the use of `let` is arguably better.
:::



Finally ending up with the following transform:

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


---

## Wrapping Up

After applying this transform, we successfully get our desired code output.

```jsx
const notMutatedVar = "definitely not mutated";
let mutatedVar = "yep, i'm mutated";

for (let i = 0; i < 5; i++) {
  mutatedVar = "foo";
const anotherInsideLoopVar = "should i be changed?";
}

for (const xin text) {
  text += x + " ";
}
```

### Takeaways

- Do a code search and methodically find and capture as many possible code patterns as possible.
- Create a test file using the captured code patterns. Use the code patterns as a reference for writing your test cases, which include both, patterns that should and should not be detected.
- Write and test your codemods with the test file you created before.

### Next Steps

After taking your first steps with writing your first codemod that solves a real-world problem, now you're ready to start taking your development process to the next level.

In the upcoming tutorial, we will cover:

- Integrating declarative codemod engines, like [Intuita’s Codemod Studio](https://codemod.studio), into your workflow.
- Writing tests for your codemods [(Read More →)](/blog/writing-test-cases-for-codemods)
- Adding your codemods to Intuita’s codemod registry [(Read More →)](/blog/writing-codemods-like-a-pro)
- Get active within the codemod community! [Join Intuita’s Slack](https://join.slack.com/t/intuita-inc/shared_invite/zt-1bjj5exxi-95yPfWi71HcO2p_sS5L2wA) community gathering leading codemod champions around the world.