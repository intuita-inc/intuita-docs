---
slug: what-are-codemods
title: What Are Codemods, Deciphered
authors: [mohab]
tags: [codemods, tutorial, beginner]
image: ./what-are-codemods-header.png
---

![What Are Codemods](./what-are-codemods-header.png)

Imagine you're upgrading a dependency for a project; however, this upgrade needs unfathomable modifications and refactoring?

In this article we go over:
- How codebase migrations can be problematic?
- What are codemods and how do they solve this issue?
- How do codemods work?
- Types of codemods.
- References for getting started with your first codemod!

<!--truncate-->

---


So you're working on a project, and for some reason, you need to upgrade or change a dependency. But by doing so, you realize that you have a lot of rework and refactoring to do in your codebase.

It feels like you've been hit by a truck. How can you go over this huge codebase and re-read, inspect, analyze, and rewrite all this code, all for a simple dependency change?

Let alone that this will most likely happen again when another dependency's usage changes sometime in the near future. 😈

Huge inconvenience! Certainly, the amazing coders out there won't leave us in the open without a solution to this, right?

## I'll Find & Replace, What's the Issue? 😎 (Hint: A Lot!)

You might like living on the edge and think about going wild with a find and replace to do all the needed changes in this scenario, but then a few replaces in and you find that you're either already breaking the code or trying to take care of so many special cases that it's infeasible. (This is our "we told you so” moment.)

<p align="center"><img src="/img/i-told-you-so.gif" alt="i told you so meme" /></p>

Imagine if you have a piece of code like this in your program:

```jsx
let hello = "world";

function sayhello(){
    console.log("hello world");
}

```

And you need to replace every ***hello*** → ***world*** .

That would be quite problematic, as you see. First of all, you're not sure which “hello” occurrences should be replaced and which shouldn't. And even if you did, simply finding and replacing will lead to replacing parts of the program that shouldn't have been replaced. Causing so much hassle and eventually leading to manual work.

So what's the solution? 😞


## Enter, Codemods! 🥳

Before you start chugging gallons of coffee ☕ and bashing on your keyboard ⌨️, wasting all those precious hours, you hear about codemods. Suddenly, the light comes back into your life. 🌟

Codemods, short for “code modifications," are cool programs that automate the tedious work of re-structuring your code for something like a dependency change by automatically and easily converting and migrating your codebases.

Codemods leverage (very) mature analysis and structuring techniques to build new cutting-edge workflow efficiency scripts.

Codemods work by creating ASTs (Abstract Syntax Trees) of your code, manipulating/re-structuring those trees, and re-writing your code using those new trees. 🌲 ASTs are simply an abstract representation of the code’s syntax, which were created so that compilers could work.

![Illustration of how codemods work](/img/blog/what-are-codemods/1.png)


Well, this sounds a bit overwhelming, but all of this is just a fancy way of saying that codemods go over your code then:
1. Tokenizes it (or splits it into small chunks 🍪)
2. Uses those tiny pieces to create a tree that resembles how your code looks like from a structural point of view.
3. Finally, uses that tree to manipulate the code and do all the needed fixes then creates new code off of that tree.

![animation of how codemods transform code](/img/blog/what-are-codemods/2.gif)

## Methods of Codemods Generation

There are 3 methods of codemod generation you can go about.

1. Imperative-Based Codemods
2. Declarative/Pattern-Based Codemods
3. LLM-Based (Large Language Models) Codemods

### Imperative-Based Codemod Engines

This type of codemod generation is very powerful and enables the codemod author to tailor the codemod for many types of complex code transformations. However, imperative-based codemod generation requires a deep understanding of ASTs and their manipulation techniques.

Imperative-based codemod engines work by using explicitly written rules for selecting nodes in an AST and transforming them; thereby changing the AST from one version to another.

An example of this method is using [ASTExplorer](https://astexplorer.net/) to manually inspect the AST, select nodes, filter nodes, modify them, and create a code transformation.

### Declarative/Pattern-Based Codemod Engines

These types of codemod engines are much easier to use compared to imperative-based engines. Declarative-based engines rely mainly on input/output states of code that allow the declarative engine to find specific patterns of code transformation between both states.

Declarative-based codemod engines hide much of the underlying implementation details, allowing a much more human-understandable method of codemod generation. However, declarative codemod engines still rely on ASTs for code transformations and are limited to supporting changes only when atomic transformations are already built.

An example of a declarative codemod engine is [JARVIS](https://rajasegar.github.io/jarvis/).

### LLM-Based (Large Language Models) Codemod Engines

LLM-based codemod engines allow codemod generation to be a data-driven process of providing examples of before and after code snippets.

These codemod engines skip the use of ASTs to develop transformations, and rather work by a direct code-to-code transformation.

Such models, however, are rarely suitable for complex transformations and are instead used for simple transformations. In addition, human review is needed to ensure that the code transformation is accurate.

## Learn How to Write Your First Codemod

Check out our guide here to ***[learn how to write your first codemod](#)***!

## Share Your Codemods With The Amazing Community 👩🏽‍💻🌎

Check out our guide here about ***[integrating your codemod into Intuita](/blog/adding-codemods-to-registry)***!