---
slug: declarative-codemod-engines
title: Declarative Codemod Engines
authors: [mohab]
tags: [codemods, tutorial, advanced]
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Declarative Codemod Engines

In the previous tutorial, we showed you how to start diving deep into advanced codemod development techniques.

The previous example focused writing codemods from scratch using [ASTExplorer](https://astexplorer.net/), which is an imperative codemod engine. 

In this tutorial, you will step up your game by using declarative codemod engines (like [JARVIS](https://rajasegar.github.io/jarvis/)) to make your process much more efficient.

<!--truncate-->

--- 

## Overview

Declarative codemod engines 

By the end of this tutorial you will learn:

- How to start tackling codemod development like an expert.
- How to write a codemod that solves a real-world problem.
- Usage of more advanced AST manipulation techniques.
- About new methods and tools that make your codemod development more efficient.

Lets learn by example together!

:::tip

*We suggest you follow this tutorial using [ASTExplorer](https://astexplorer.net/) w/ JSCodeShift.*

:::

---

## Prerequisites

### Understanding ASTs
This tutorial requires understanding of Abstract Syntax Trees (ASTs). You should be able to inspect, traverse, & manipulate ASTs. If you're not familiar with ASTs, make sure to check out [our guide on understanding ASTs first](#).

### Writing simple codemods
Before getting into this tutorial, make sure you're familiar with writing simple codemods. If you're not, make sure to go over [our tutorial for writing your first codemod](#), first.

---

