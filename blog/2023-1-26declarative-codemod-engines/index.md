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


## Prerequisites

- **Understanding ASTs -** This article requires a good understanding of how to inspect, traverse, and manipulate ASTs. If you need a refresher, check out [our guide on understanding ASTs](#).
- **Advanced Understanding of Codemods -** This article requires an advanced understanding of writing codemods. Before jumping into this tutorial, make sure you've went through our [tutorial on writing codemods like a pro](#).


## Overview

Throughout this document, dive deep into speeding up your codemod development process through integrating declarative codemod engines into your development workflow.

Ultimately, our goal is to leverage declarative codemod engines to make our codemod development process much more efficient.

By the end of this tutorial, you will learn:

- How to use declarative codemod engines (like JARVIS).
- When to (and when NOT to) use declarative codemod engines.
- New methods and tools that make your codemod development more efficient.

Let's learn by example together!




