---
sidebar_position: 1
---

import ThemedImage from "@theme/ThemedImage";
import useBaseUrl from "@docusaurus/useBaseUrl";
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Intro

## Awesome codemods delivered to you!


Intuita is a codemod manager 🤖 that makes migrations, dependency upgrades & large refactorings faster & easier for codebases of any size.

Intuita's extension is open source & supports the best codemod engines out there, such as Facebook's JSCodeShift, Uber's Piranha, & soon ML-powered engines. If your favorite codemod engine is not supported yet, please create an issue for us to prioritize.

The codemods for upgrading the following frameworks/libraries have already been added to the extension. If you want to upgrade a dependency in the near future & it's not listed here, please create an issue for us to prioritize.

- Nextjs vx → v13
- Material UI v4 → v5
- React Router vx → v4 & v4 → v6
- ImmutableJS x3 -> v4
- ⚠️ This extension is in Public Beta! Get in touch if you are planning to use this extension for large upgrades & you have some feature requests!

## How It Works 🪄

1. Open package.json in your project & applicable upgrade codemods automatically show up for you.

```jsx
"@mdx-js/react": "^1.6.22",
"clsx": "^1.2.1",
// highlight-next-line
"next": "12.3.1",
"prism-react-renderer": "^1.3.5",
"react": "^17.0.2",
"react-dom": "^17.0.2"

```

2. Click on "execute codemods" or run the codemods from the VS Code's Command menu & let Intuita transform your code automatically.
<ThemedImage
  alt="Touca server signup page"
  sources={{
    light: useBaseUrl("/img/intro/execute-codemods.png"),
    dark: useBaseUrl("/img/intro/execute-codemods.png")
  }}
/>


3. Quickly review, tweak, & apply the proposed changes one-by-one or in batches.

<Tabs>
<TabItem value="before-changes" label="Before Changes" default>

```jsx  title="Before Change"
"@mdx-js/react": "^1.6.22",
// red start
"clsx": "^1.2.1",
"next": "12.3.1",
"prism-react-renderer": "^1.3.5",
// red end
"react": "^17.0.2",
"react-dom": "^17.0.2"
```

</TabItem>

<TabItem value="after-changes" label="After Changes" default>

```jsx  title="After Change"
"@mdx-js/react": "^1.6.22",
// green start
"clsx": "^1.2.1",
"next": "12.3.1",
"prism-react-renderer": "^1.3.5",
// green end
"react": "^17.0.2",
"react-dom": "^17.0.2"
```

</TabItem>

</Tabs>

> If you like videos better, here is a quick YouTube tutorial.



## Why Intuita?

### Codemod Consumers 🧑🏾‍💻👩🏻‍💻


- ❌ Without Intuita ❌ if you want to upgrade a dependency with some API & breaking changes, you need to:
  - Search online for upgrade guides & if you are lucky, find some codemods (likely not following any quality guidelines or without proper tests).
  - Run those codemods in CLI one by one (~60 codemods for Material UI v5 upgrade) without knowing which one is actually applicable to your codebase.
  - End up with large PRs & many reviewers because you couldn't easily distribute the changes into meaningful commits.
  - And finally, if many teams are impacted, you need to create tasks, manually find the best reviewer & follow up on those tasks until you get the change reviewed & merged.

- ✅ With Intuita ✅ a set of quality-checked community-led codemods will be pushed to you right in the package.json (for JS/TS) so you don't need to search for them.
  - You run all the codemods with one click.
  - At an intermediary step before making the code change, you can tweak them if needed, batch them as you wish & push your changes for review.
  - [Enterprise feature] for large teams, we will be building integrations with Jira/Slack/Github to automatically assign tasks & send reminders to teams, leadership dashboard for tracking & coordinating the campaign.




### Codemod Builders 🥷🏼

- ❌ Without Intuita ❌ you go the codemod journey alone! You ask yourself:
  - What are the best codemod engines out there for the type of transformation i want to build?
  - What's the easiest way to build codemods? Any tool out there? How about creating test cases?
  - Has anyone out there built the exact codemod I want to build?
  - How can my codemod be discovered & used by thousands of other developers out there who could benefit from it sooner or later?

- ✅ With Intuita ✅ you are surrounded by a community of codemod experts
  - We build tools, processes & guidelines on how to write high-quality codemods & associated tests & pick the right engine (imperative, declarative, ML based, or pattern based) for each use case.
  - We will be the distribution channel for your codemods. You write a useful codemod & along with other codemods created by the community, we will ship it to tens of thousands of developers out there within a few mins of integrating them into Intuita.
  - If you, just like us, are also a codemod nerd, let's get in touch! we are growing our team with awesome engineers who are passionate about automating boring tasks for fellow developers.


### Engineering Leads 👩🏾‍💼👨🏻‍💼
- ❌ Without Intuita ❌ leaders do not have visibility into the status of large migration/upgrade campaigns. You ask yourself:
  - How long would this upgrade take? How many developers are impacted? Are we on track to hit our business deadlines?
- ✅ With Intuita ✅ [enterprise features] you can easily track progress & coordinate migration campaigns across many teams.
  - Team leads, please contact us to learn more about enterprise features such as Jira/Github/Slack integrations, automated task & notification delivery.


