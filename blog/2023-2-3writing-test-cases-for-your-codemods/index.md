---
slug: writing-test-cases-for-codemods
title: Writing Test Cases for Your Codemods
authors: [mohab]
tags: [codemods, tutorial, advanced]
# Display h2 to h5 headings
toc_min_heading_level: 2
toc_max_heading_level: 3
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


In the previous tutorial, we discussed how to take your codemod skills to the next level by applying the techniques used by professionals to write a codemod like `no-vars`.

In this tutorial, we take it one step further by writing test cases that confirm that our codemods identify all essential code patterns and then perform transforms correctly and precisely. Writing tests for our codemods can significantly improve your codemod development process.

Keep reading to add this skill to your arsenal.

---

## Prerequisites

- **Understanding ASTs -** This article requires a good understanding of how to inspect, traverse, and manipulate ASTs.
- **Advanced Understanding of Codemods -** This article requires an advanced understanding of writing codemods. Before jumping into this tutorial, make sure you've gone through our [tutorial on writing codemods like a pro](/blog/writing-codemods-like-a-pro).

## Why Write Test Cases for Your Codemods?

Writing test cases for codemods is an extremely valuable practice. 

Test cases help with:
- Ensuring that the codemod is working correctly as it’s being developed.
- Having a guideline of the requirements your codemod should satisfy as you develop it.
- Catching codemod regressions as the codemod evolves in the future.
- Documenting how the codemod is supposed to work for people in your team or community contributors.
- Ensuring that the codemod works across different environments.

With that being said, let’s learn by example how to write our first codemod test case.

## Scenario

In our previous tutorial, we created a set of code patterns we keep track of to ensure that our codemod conforms to its requirements while it’s being built.

Our identified code patterns for the `no-vars` codemod included some patterns that should be detected and others that should not be detected. We categorized the identified code patterns into:
- Patterns where we transform `var` declarations into `let`.
- Patterns where we transform `var` declarations into `const`.
- Patterns where we keep `var` declarations unchanged (should not be detected).

Our goal now is to use the code patterns’ before and after snippets as a basis for our test cases.

Now let’s turn those code patterns into test cases!

## Writing Our First Codemod Test Case

Writing tests for our codemods is actually a very simple process.

Our workflow for writing the test cases will be as follows:
1. Preparing our testing environment.
2. Creating our transform and test files.
3. Writing our test cases.
4. Running our test cases.

### #1 Preparing Our Testing Environment

Usually, when you’re preparing your testing environment, you would need to:
1. Install a testing library (ex: Mocha)
2. Install an assertion library (ex: Chai)
3. Creating a testing directory
4. Adding test files for your testing directory

Alternatively, you could use our codemod registry, which already has the required libraries configured as well as a neat directory configuration for testing your codemods while eloquently integrating into the codemod platform.

We’ll continue this tutorial using the codemod registry, as it’s a simpler and more scalable approach for codemod builders.

All you need to do is:

1. Clone Intuita’s [codemod registry](https://github.com/intuita-inc/codemod-registry) on your local environment.
    
    `git clone https://github.com/intuita-inc/codemod-registry`
    
2. Open the codemod registry directory in your code editor of choice.
    
    ![opening vs code editor](/img/blog/writing-test-cases-for-your-codemods/1.png)
    

### #2 Creating Our Test Files

Test files contain all the code patterns that should be detected by the codemod for transformation, as well as tricky code patterns that should not be detected or transformed. The goal of the test file is to define all the patterns that, collectively, reduce the number of false positives and false negatives in our codemod.

Now that we have our local environment set up, we can begin by creating our test files. To do so, we will:

1. Navigate to the codemods/jscodeshift directory.
2. Run `npm install`.
3. Create a new folder for your codemod. In this example, we’ll call it `no-vars`. If you plan to use the codemod registry for community contribution, please refer to our article about [adding codemods to the codemod registry](/blog/adding-codemods-to-registry), where we go over the best practices to add your codemod to the registry.
    
    ![creating a new directory for our codemod](/img/blog/writing-test-cases-for-your-codemods/2.png)
    
4. In our new directory, we’ll add an `index.js` file and a `test.ts` file. The former will include our codemod code, while the latter will include our test cases.
5. Add our codemod to `index.js`. We will be using our previously written codemod `no-vars`, feel free to apply this process to a codemod you’ve written as well. At this stage, the `index.js` file will look as shown below:

```jsx
export default function transformer(file, api, options) {
  const j = api.jscodeshift;

  const root = j(file.source);

  .
  ..
  ...
  //Lots of helper functions
  ...
  ..
  .

  const updatedAnything = root
    .find(j.VariableDeclaration)
    .filter(dec => dec.value.kind === 'var')
    .filter(declaration => {
      return declaration.value.declarations.every(declarator => {
        return !isTruelyVar(declaration, declarator);
      });
    })
    .forEach(declaration => {
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
    })
    .size() !== 0;

  return updatedAnything ? root.toSource() : null;
}

```

### #3 Writing Our Test Cases

At this stage, we should write test cases that check for code patterns that should be detected, as well as those that should not be detected. 

Let’s start by writing our test cases in the `test.ts` file.

We’ll start first by adding a `describe()` function, which, you guessed it, describes what our test file (as a whole) does. In this case, we can just use `'no-vars'` as a simple description. If you’re writing test cases for migrating a codebase from one framework version to another, it would be a good idea to name the description something that describes the framework version and codemod name (ex: `'next 13 an-amazing-codemod'`).

Our file will now look like so:

```tsx
import { FileInfo } from 'jscodeshift';
import assert from 'node:assert';
import transform from '.';

describe('no-vars', function () {	
	//Will add test cases here
});
```

Now, all we have to do is add test cases for specific code patterns we identified inside our `describe()` function. We add test cases using `it()`. The `it()` function accepts 2 parameters:

1. A test case description
2. A function that specifies the test case

Let’s say for example we want to write a test case that checks if the transform we wrote changed the `var` variable declarator into `let` in case of the variable being mutated, we would end up with something like:

```tsx
import { FileInfo } from 'jscodeshift';
import assert from 'node:assert';
import transform from '.';

describe('no-vars', function () {
	it('should convert var to let if var is mutated', function () {
		//Write test case details here
	});
});
```

Now, all we have to do is write the expected input and output then check if our transform correctly converts from our input state to the other.

To do this, we should:

1. Specify an input string that contains the *‘before’* state of the code pattern.
2. Specify an output string that contains the *‘after’* state of the code pattern.
3. Specify the associated JSCodeshift codemod (`index.js` in our case).
4. Get the actual output when we run the transform on our input code pattern.
5. Check whether the actual output matches the specified expected output.

By applying those steps, we should end up with our test case as illustrated below:

```tsx
import { FileInfo } from 'jscodeshift';
import assert from 'node:assert';
import transform from '.';

describe('no-vars', function () {
	it('should convert var to let if var is mutated', function () {
		//How the code should look like before applying our transform
		const INPUT = `
		var x = 1;
		x = 2;
        `;

		const OUTPUT = `
		let x = 1;
		x = 2;
		`;
		
		//Our target transform and target input. In this case, we are using index.js and the INPUT const
		const fileInfo: FileInfo = {
			path: 'index.js',
			source: INPUT,
		};
		
		//The actual output after running our target transform on the input source
		const actualOutput = transform(fileInfo, this.buildApi('jsx'), {});
		
		//Checking if actualOutput matches the specified OUTPUT const
		assert.deepEqual(
			actualOutput?.replace(/\W/gm, ''),
			OUTPUT.replace(/\W/gm, ''),
		);
	});

	
});
```

From this point forward, you can specify as many `it()` functions as you want which cover all the code patterns you’ve identified.

:::tip
You can use advanced codemod platforms such as [Codemod Studio](https://codemod.studio) to get a list of test cases or skeletons for your Mocha test file to boost your test development process!

Using Codemod Studio can instantly give you a list of test cases such as (from our previous tutorial *“Write Codemods Like A Pro”):*

1. Test that the variable `notMutatedVar` is declared as a `const` variable and not mutated.
2. Test that the variable `mutatedVar` is declared as a `let` variable and mutated inside the `for` loop.
3. Test that the variable `anotherInsideLoopVar` is declared as a `const` variable inside the `for` loop and not mutated.
4. Test that the `for` loop with `in` keyword transforms the variable declaration of `x` to a `const` variable declaration.
5. Test that the `for` loop with `in` keyword does not modify the behavior of the loop and produces the same output as before the transformation.

which you can also export as a skeleton Mocha test file!
:::

### #4 Running Our Test Cases

Finally, to run our test cases we should:

1. Return to the codemods/jscodeshift directory.
2. Run `npm test`

Mocha should now do its magic and run all our test cases, providing a report of test statuses as shown below.

![passing mocha test cases](/img/blog/writing-test-cases-for-your-codemods/3.png)

In the case that our codemod doesn’t work as it should, our test case will now easily catch the issue and report it so you can correct any inaccuracies in your codemod.

![failing mocha test cases](/img/blog/writing-test-cases-for-your-codemods/4.png)

## Takeaways

- Use your codemod patterns as a basis to write your test cases by converting code patterns into test cases using the before and after code states.
- Regularly run your test cases during codemod development to ensure that your codemods satisfy the end results you expect to see in your codebase.
- Use your test cases as your main reference during development and update your test cases if any new edge cases arise. This allows you to confidently rely on your test results during development.
- Break up code patterns over multiple `it()` test cases. This allows others to easily understand and improve on your codemods in the future.
- Always use understandable test case descriptions and include comments that describe what specific test cases do.