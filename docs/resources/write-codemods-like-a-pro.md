---
sidebar_position: 1
---

# Write a Codemod Like a Pro


So you're working on a project, and for some reason, you need to upgrade/change a dependency. But by doing so, you realize that you have a lot of rework and refactoring to do in your codebase.

It feels like you've been hit by a truck. How can you go over this huge codebase and re-read, inspect, analyze, and rewrite all this code; and all just for a simple dependency change?

Let alone that this will most probably happen again when another dependency usage changes sometime in the near future. 😈

Huge inconvenience! Certainly the amazing coders out there won't leave us in the open without a solution to this, right?


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

And you need to replace every ***hello*** &rarr; ***world***.
</div>

That would be quite problematic as you see. First of all, you're not sure which hellos should be replaced, and which shouldn't. And even if you did, simply finding and replacing will lead to replacing parts of the program that shouldn't have been replaced. Causing so much hassle and eventually leading to manual work.

So what's the solution? 😞


## Enter, Codemods! 🥳

Before you start chugging gallons of coffee ☕ and bashing on your keyboard ⌨️, wasting all those previous hours, you hear about codemods. Suddenly the light comes back into your life. 🌟

Codemods - shorthand for code modifications - are cool programs that can help you automate all the tedious work of modifying your code to do all the needed changes to re-structure it for a dependency change. Codemods easily allow your current codebases to be *transformed* automatically and migrate from one structure to another that allows it to work after needed dependency changes.

Codemods leverage (very) mature analysis and structuring techniques to build new cutting-edge workflow efficiency scripts.

Codemods work by creating ASTs (Abstract Syntax Trees) of your code, manipulating/re-structuring those trees, and re-writing your code using those new trees. 🌲

<p align="left" class="inverted-grey"><img  width="400" src="/img/twilio-ast-tokenization.gif" alt="tokenization" /></p>


Well, this sounds a bit overwhealming, but all of this is just a fancy way of saying that codemods go over your code, tokenizes it (or splits it into small chunks 🍪), and then uses those tiny pieces to create a tree that resembles how your code looks like in a complete structural point of view, and finally uses that tree to manipulate the code & doing all the needed fixes then creating new code off of that tree. Leaving you with refactored & working code in no time, Voila! 🪄 

<p align="left" class="inverted-grey"><img  width="400" src="/img/twilio-ast-parsing.gif" alt="tokenization" /></p>



## Sold? Now, How Do You Write Your First Codemod? ⌨️

*tutorial for writing codemods here, mainly rely on [codeshift's tutorial](https://www.codeshiftcommunity.com/docs/your-first-codemod/)*


## Share Your Codemods With The Amazing Community 👩🏽‍💻🌎

*insert small part here about sharing codemods with [our community](http://localhost:3000/docs/contributing/our-community)*


