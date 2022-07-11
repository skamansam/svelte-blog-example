# create-svelte

Everything you need to build a Svelte project, powered by [`create-svelte`](https://github.com/sveltejs/kit/tree/master/packages/create-svelte).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```bash
# create a new project in the current directory
npm init svelte

# create a new project in my-app
npm init svelte my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.


---
comments: true
view: post
layout: post
author: skamansam
title: Creating a Blog in Svelte
description: We are going to go step by step and see how to create a markdown-based blog using SvelteKit. (up-to-date as of 07-11-22)
excerpt: 
categories:
  - programming
  - Svelte
  - Blog
tags:
  - blog
  - how to
  - Rude Boy Solutions
created_at: 2022-07-11 09:50:16
---

<nav class="toc">

## On This Page

</nav>

> In this series, I will be using `pnpm` instead of `npm` as my package manager.

## Project Setup
```
# make sure you are using the right version:
# install nvm
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
# make sure you are using the new/updated NVM:
. ~/.bashrc
# install node latest (18.5.0) at the time of thsi writing
nvm install node
# install pnpm:
npm install -g pnpm
# Step 1: skaffold with sveltekit:
pnpm create svelte blog-example
# select "demo app", "typescript", "eslint", "prettier", "playwright"
cd blog-example
pnpm i
make sure we save the current version of node so we can get back to it
echo `node --version` > .nvmrc
# save with git
git init && git add -A && git commit -m "Initial commit"
# open the sample app
pnpm run dev --open
# open up your favorite editor (mine is the insiders version of vscode)
code-insiders .
```

### add markdown support
```
pnpx svelte-add@latest mdsvex
pnpm i
```
reload your editor to make sure the intellisense picks up all the changes: ctrl+shift+p, "reload window"

see the changes that have been made
```
git commit -am "add mdsvex support"
```

### setup deployments to static hosting (ie.e github-pages)
edit svelte.config and change 

install the static adapter
```sh
pnpm i @sveltejs/adapter-static
```

```js
import adapter from '@sveltejs/adapter-auto';
```
to 
```js
import adapter from '@sveltejs/adapter-static';
```
test whether we can build, then preview the build:
```
pnpm build
pnpm preview
git commit -am "configure static adapter"
```

maybe run the lighthouse analysis on it and watch the fireworks!


### Add some markdown content!

```bash
mkdir -p src/routes/posts
touch src/routes/posts/sample-entry.md
```
Then write a basic entry. we are going to use this to test whether our markdown plugins work.

```
<!-- src/routes/posts/sample-entry.md -->

# Blog Entry #1!

This is a sample blog entry that will highlight the capabilities of our blog and serve as a markdown reference.
```

build, test, and commit.

Since we want to get some parity with `markdown-it` so we can just use our existing blog entries, we will copy the 
`markdown-it` demo from https://markdown-it.github.io/

build, test, then commit
```
pnpm build
pnpm preview
```

OOPS! build failed.

```
[vite-plugin-svelte-kit] Cannot read properties of undefined (reading 'paths')
error during build:
TypeError: Cannot read properties of undefined (reading 'paths')
    at create_builder (file:///home/sam/workspace/blog-example/node_modules/.pnpm/@sveltejs+kit@1.0.0-next.368_svelte@3.49.0+vite@2.9.14/node_modules/@sveltejs/kit/dist/chunks/index2.js:52:48)
    at adapt (file:///home/sam/workspace/blog-example/node_modules/.pnpm/@sveltejs+kit@1.0.0-next.368_svelte@3.49.0+vite@2.9.14/node_modules/@sveltejs/kit/dist/chunks/index2.js:204:18)
    at Object.closeBundle (file:///home/sam/workspace/blog-example/node_modules/.pnpm/@sveltejs+kit@1.0.0-next.368_svelte@3.49.0+vite@2.9.14/node_modules/@sveltejs/kit/dist/vite.js:3099:11)
    at async Promise.all (index 0)
    at async /home/sam/workspace/blog-example/node_modules/.pnpm/rollup@2.76.0/node_modules/rollup/dist/shared/rollup.js:23693:13
    at async catchUnfinishedHookActions (/home/sam/workspace/blog-example/node_modules/.pnpm/rollup@2.76.0/node_modules/rollup/dist/shared/rollup.js:23174:20)
    at async rollupInternal (/home/sam/workspace/blog-example/node_modules/.pnpm/rollup@2.76.0/node_modules/rollup/dist/shared/rollup.js:23682:5)
    at async doBuild (/home/sam/workspace/blog-example/node_modules/.pnpm/vite@2.9.14/node_modules/vite/dist/node/chunks/dep-c9998dc6.js:41710:24)
    at async build (/home/sam/workspace/blog-example/node_modules/.pnpm/vite@2.9.14/node_modules/vite/dist/node/chunks/dep-c9998dc6.js:41552:16)
    at async CAC.<anonymous> (/home/sam/workspace/blog-example/node_modules/.pnpm/vite@2.9.14/node_modules/vite/dist/node/cli.js:738:9)
 ELIFECYCLE  Command failed with exit code 1.

```

it appears something is not being processed correctly with the markdown-it content. Problem detection strategy: divide and conquer. Remove by halves and see where the problem is. Removing everything from the plugins section works. the problem must be there. put back half of that. add back all to footnotes section. removed abbreviations and custom containers causes fail so it must be in the footnotes or definition lists section. removing definition lists causes it to succeed so the problem must be there.

attempt 1: replace curly brackets with svelte-syntax: { => {'{'}. still fails
attempt 2: comment out first section. succeeds. what is here that fails?
attempt 3: commnt out the braketed section. passes. wth?