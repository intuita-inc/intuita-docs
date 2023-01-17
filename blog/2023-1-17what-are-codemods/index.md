---
slug: what-are-codemods
title: What Are Codemods? - Deciphered
authors: [mohab]
tags: [codemods, tutorial, beginner]
image: ./what-are-codemods-header.png
---

![Docusaurus Plushie](./what-are-codemods-header.png)

# What Are Codemods? - Deciphered


So you're working on a project, and for some reason, you need to upgrade/change a dependency. But by doing so, you realize that you have a lot of rework and refactoring to do in your codebase.

It feels like you've been hit by a truck. How can you go over this huge codebase and re-read, inspect, analyze, and rewrite all this code; and all just for a simple dependency change?

Let alone that this will most probably happen again when another dependency usage changes sometime in the near future. 😈

Huge inconvenience! Certainly the amazing coders out there won't leave us in the open without a solution to this, right? Fear no more.

<!--truncate-->


## I'll Find & Replace, What's the Issue? 😎 (Hint: A Lot!)

You might like living on the edge and thought about going wild with a find & replace to do all the needed changes in this scenario, but then few replaces in and found that you're either already breaking the code or trying to take care of so many special cases that it's infeasible. (This is our "we told you so moment").

<p align="center"><img src="/img/i-told-you-so.gif" alt="i told you so meme" /></p>

<div align="center">

Imagine if you have a piece of code like this in your program:

<p class="max-width-300" align="left">

```jsx
let hello = "world";

function sayhello(){
    console.log("hello world");
}
```

</p>

And in every part of your program, the variable ***hello*** needs to be changed &rarr; ***world***.

</div>

That would be quite problematic as you see. First of all, you're not sure which hellos should be replaced, and which shouldn't. And even if you did, simply finding and replacing will lead to replacing parts of the program that shouldn't have been replaced. Causing so much hassle and eventually leading to manual work.

So what's the solution? 😞


## Enter, Codemods! 🥳

Before you start chugging gallons of coffee ☕ and bashing on your keyboard ⌨️, wasting all those previous hours, you hear about codemods. Suddenly the light comes back into your life. 🌟

Codemods - shorthand for code modifications - are cool programs that can help you automate all the tedious work of modifying your code to do all the needed changes to re-structure it for a dependency change. Codemods easily allow your current codebases to be *transformed* automatically and migrate from one structure to another that allows it to work after needed dependency changes.

Codemods leverage (very) mature analysis and structuring techniques to build new cutting-edge workflow efficiency scripts.

Codemods work by creating ASTs (Abstract Syntax Trees) of your code, manipulating/re-structuring those trees, and re-writing your code using those new trees. 🌲
<p align="left" class="inverted-grey"><img src="https://assets.cdn.prod.twilio.com/images/-vnpXSyzxyPpPK3TqYdV1pedY3MnsEnyyGUH1WlTd0O29Cl.original.png" alt="tokenization" /></p>


Well, this sounds a bit overwhealming, but all of this is just a fancy way of saying that codemods go over your code then:
1. Tokenizes it (or splits it into small chunks 🍪)
<p align="left" class="inverted-grey"><img width="400" src="/img/twilio-ast-tokenization.gif" alt="tokenization" /></p>
2. Uses those tiny pieces to create a tree that resembles how your code looks like in a complete structural point of view.
<p align="left" class="inverted-grey"><img width="400" src="/img/twilio-ast-parsing.gif" alt="parsing" /></p>
3. Finally uses that tree to manipulate the code & doing all the needed fixes then creating new code off of that tree.
<p align="left" class="inverted-grey"><img width="400" src="/img/twilio-ast-code-generation.gif" alt="code generation" /></p>

Leaving you with refactored & working code in no time, Voila! 🪄 

<p class="max-width-400">

```json
{
 "type": "Program",
 "start": 0,
 "end": 14,
 "body": [
   {
     "type": "ExpressionStatement",
     "start": 0,
     "end": 14,
     "expression": {
       "type": "CallExpression",
       "start": 0,
       "end": 13,
       "callee": {
         "type": "Identifier",
         "start": 0,
         "end": 7,
         "name": "isPanda"
       },
       "arguments": [
         {
           "type": "Literal",
           "start": 8,
           "end": 12,
           "value": "🐼",
           "raw": "'🐼'"
         }
       ]
     }
   }
 ],
 "sourceType": "module"
}
```

</p>





All in all, codemods use tokenize your code, built ASTs with those tokens, then manipulates those ASTs based on given rules, and gives you a migrated new fresh working code. ✨


<div class="flex-row">

<div class="flex-column">

```jsx  title="Before Change"
// red line
let hello = "world";

function hello(){
    console.log("hello world");
}
```

</div>

<div class="flex-column">

```jsx  title="After Change"
// green line
let world = "world";

function hello(){
    console.log("hello world");
}

```

</div>
</div>

## Sold? Now, How Do You Write Your First Codemod? ⌨️

*check our tutorial for writing your first codemod*


## Share Your Codemods With The Amazing Community 👩🏽‍💻🌎

*check our tutorial for integrating your codemod into intuita*

