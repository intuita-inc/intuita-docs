---
slug: adding-codemods-to-registry
title: Adding Codemods To Intuita’s Codemod Registry
authors: [mohab]
tags: [codemods, tutorial]
# Display h2 to h5 headings
toc_min_heading_level: 2
toc_max_heading_level: 3
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';


Adding codemods to the registry allows you to get involved and contribute to the codemod community, establishing yourself as an active member of the community. If you create a codemod that solves a common problem, it can be of tremendous assistance to others who are experiencing the same difficulty, which is appreciated by tons of other developers around the world. 

Additionally, it can assist you in improving your codemods. By sharing your codemods with others, you may receive feedback that can help you improve them. Finally, adding your codemods to the registry allows seamless integration with Intuita’s platform, allowing your codemods to be easily supported and shipped to Intuita’s products that simplify your workflow tremendously as a software developer.

To reap those benefits, we’ll go over how to add your codemods to the codemod registry in this simple tutorial.

<!--truncate-->

---

## Prerequisites

- **Basic Understanding of Codemods -** This article requires a basic understanding of writing simple codemods. If you're unfamiliar with writing codemods, check out [our tutorial on writing your first codemod](#).

## How to Add Codemods to the Registry

Now let’s walk through the steps of adding your amazing new codemod to the registry. To do so, we will go through the following steps:

1. Preparing your environment
2. Understanding the codemod registry
3. Adding your codemod files
4. Creating a PR to merge to the main repository

### #1 Preparing Your Environment

To have a ready environment, you must first have a local copy of the codemod registry where you can make changes before merging those changes into the main repository.

To do this, all you need to do is:

1. Fork Intuita’s [codemod registry](https://github.com/intuita-inc/codemod-registry).
    
    ![forking codemod registry](/img/blog/adding-codemods-to-registry/1.jpeg)
    
    ![forking codemod registry](/img/blog/adding-codemods-to-registry/2.jpeg)
    
2. Clone your forked repository by using: `git clone https://github.com/<your-username>/codemod-registry` or using GitHub Desktop GUI.
3. Open the codemod registry directory in your code editor of choice.
    
    ![opening codemod registry in vs code](/img/blog/adding-codemods-to-registry/3.png)
    

4. Then, you’ll need to create a new branch where you will do your changes (adding your codemod to the registry).
    
    To do this, you can create and switch to a new branch using `git checkout -b <branch-name>`.
    
    Alternatively, you can create a branch using GitHub Desktop (GUI) by clicking on the current `main` branch and creating a new branch. Remember to name your branch with a name that’s descriptive of the codemod you’re adding.
    
    ![creating a branch of the codemod registry](/img/blog/adding-codemods-to-registry/4.jpeg)
    
    ![creating a branch of the codemod registry](/img/blog/adding-codemods-to-registry/5.jpeg)
    
    ![creating a branch of the codemod registry](/img/blog/adding-codemods-to-registry/6.png)
    

### #2 Understanding the Codemod Registry

Now that you have a local environment set up, you’re ready to add your codemod files to the registry.

Let’s first take a look at the codemod registry structure.

![codemod registry directory structure](/img/blog/adding-codemods-to-registry/7.jpeg)

You’ll find that the registry has 2 main directories:

1. Codemods
2. Sets

**Codemods:**

The codemods directory will include all the codemod files used available in the registry, categorized by codemod engine > framework/library > version > codemod name.

For example, a codemod named `replace-use-router` to upgrade `Next.js` to `v13` will be in the following directory:

`/codemods/jscodeshift/next/13/replace-use-router/`.

**Sets:**

Sets are a group of related codemods. Usually, sets are used as a group of codemods that collaboratively migrate a codebase from one dependency version to another.

In Intuita’s codemod registry, sets are also used to seamlessly integrate your codemods into the registry with other products available on Intuita’s platform. For example, if you add a codemod to an existing or new set, your codemod will be eloquently integrated into Intuita’s VS Code Extension under an existing or new codemod set.

![intutia vs code extension integration with codemod registry](/img/blog/adding-codemods-to-registry/8.png)

### #3 Adding Codemod Files

Now that we have our environment set up, we can begin by creating our codemod files. As of now, the codemod registry supports the JSCodeshift engine due to its popularity. If you want us to support more codemod engines, feel free to [submit an issue](https://github.com/intuita-inc/codemod-registry/issues/new) on the registry with your request.

For this tutorial, we will follow the steps of adding a JSCodeshift codemod. To do so, we will:

1. Navigate to the `/codemods/jscodeshift` directory.
2. Run `npm install`.
3. Add the codemod directory. Let’s say, for example, we’re adding a codemod called `isiterable-to-iscollection` that upgrades `Immutable JS`  to `v4`. In this case, you should look for (if the set directory exists) or create a directory that’s structured `/codemods/jscodeshift/immutable/4/isterable-to-iscollection/`. As you can see, we follow the convention of `/jscodeshift/library/version/codemod-name` whether we’re creating a new codemod or adding a codemod to an existing set.
    
    ![adding your codemod directory to the codemod registry](/img/blog/adding-codemods-to-registry/9.png)
    

4. In your new directory, you’ll add an `index.ts` file and a `config.json` file (also preferably a test file, [read more on testing your codemods here](/blog/writing-test-cases-for-codemods). The `index.ts` file will include the codemod you’ve written while the `config.json` file will include metadata about your codemod.
5. Add our codemod to `index.ts`. In this example, we’ll be using an existing codemod we’ve written for Immutable JS. At this step, you can add any codemod you’ve written.
    
    ![adding your codemod files in index.ts](/img/blog/adding-codemods-to-registry/10.png)
    

6. Now, let’s add some simple metadata about our codemod in the `config.json` file. In this file, enter the metadata relevant and corresponding to the codemod you’re adding to the registry, including the codemod name, engine, required dependency version, etc.
    
    ![adding codemod metadata in config.json](/img/blog/adding-codemods-to-registry/11.png)
    

7. Finally, you should just add your new codemod to a new/existing codemod set. To do this, navigate to the `/sets` directory, then look for an existing (if applicable) or create a new codemod set. If you’re adding a new codemod set to the registry, create a directory with the following convention: `framework_version`. In our example, we added a codemod for Immutable JS v4, then we should expect to add our codemod to the `immutable_4` set. Now we should simply just add our new codemod directory into the set’s `config.json` file as shown below.
    
    ![adding your codemod in the codemod sets to config.json](/img/blog/adding-codemods-to-registry/12.jpeg)
    

### #3 Merging Your Codemod Into The Registry

Now your amazing new codemod is fully prepared to be merged into the registry.

To merge your codemod, we’ll start by committing all the changes in your new branch using `git add -A && git commit -m "<your message>"`. Then, we publish the branch using `git push --set-upstream origin <branch-name>`.

Now, if you open your forked repository, you'll see your new amazing codemod added to your repository.

![your new codemod's branch](/img/blog/adding-codemods-to-registry/13.jpeg)

All you need to do now is simply open a pull request by clicking *“Compare & pull request”.*

![opening a pull request for your new codemod](/img/blog/adding-codemods-to-registry/14.jpeg)

And finally, add a description of your added codemod and create the PR.

![creating the pull request to add your codemod to the registry](/img/blog/adding-codemods-to-registry/15.jpeg)

Once you create your PR, it will be shortly reviewed before getting merged into the codemod registry to be integrated into our platform and used by codemod enthusiasts around the world!

## Final Words

Intuita's codemod registry is an excellent resource for developers looking to automate code updates. By adding your codemods to the registry, you can help other developers, gain recognition, and improve your codemods. If you have created a useful codemod, we encourage you to share it with the community by adding it to Intuita's registry.