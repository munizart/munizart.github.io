---
layout: "post"
title: "How to add types to javascript with typescript"
description: "A small tutorial that explains how to use typescript compilers to check types in javascript"
tags:
 - "tutorial"
 - "typescript"
 - "type systems"
 - "javascript"
 - "programming"
 - "static typechecking javascript"
---

### What is this post about?
Do you ever wanted to have a look at what typescript can do for your
codebase without having to switch it all to `.ts`?

If so, this post is for you!
This tutorial will show how you can use typescript's compiler to check your
javascript code and how to use global type definitions to describe your types.

[TL;DR](#tldr)

### Let's do it!
To start, on a new folder, we create a `package.json` file:
```json
{
  "name": "js-types-tutorial",
  "version": "1.0.0",
  "main": "src/index.js",
  "license": "MIT"
}
```
And add your dependencies using your package manager (in this case `yarn`):
```sh
yarn add -D typescript
```

Create a new `src/index.js`:
```js
let myVar = 3
myVar = "string"
```

The intention here is to get the following error when checking:
```sh
src/index.js:2:1 - error TS2322: Type '"string"' is not assignable to type 'number'.
```

To do so we update our `package.json` file:
```json
{
  "name": "types",
  "version": "1.0.0",
  "main": "src/index.js",
  "license": "MIT",
  "devDependencies": {
    "typescript": "^3.7.5"
  },
  "scripts": {
    "check": "tsc -p jsconfig.json"
  }
}

```
Note that we've add `scripts.check` and installing typescript added `devDependencies.typescript`.
The `check` script invokes `tsc`, the *T*ype*S*crip *C*ompiler and tell it to look for the `-p`roject configuration at `jsconfig.json`.

Which leeds us to: `jsconfig.json`
```json
{
  "include": [
    "src"
  ],
  "compilerOptions": {
    "checkJs": true,
    "noEmit": true
  }
}
```

A list of things going on here:
  - `include`: tells `tsc` where the code is
  - `compilerOptions.checkJs`: tells `tsc` to look for `.js` too, instead of only `.ts`
  - `compilerOptions.noEmit`: This one is trickier. It means that the compiler will parse the files, do the syntatic and the semantic analisys and halt without write any compiled files.

Now you should be able to run `npm run check` and see this output:

```sh
> types@1.0.0 check
> tsc -p jsconfig.json

src/index.js:2:1 - error TS2322: Type '"string"' is not assignable to type 'number'.

2 myVar = "string"
  ~~~~~

Found 1 error.
```

Bingo! We got type-checking for `.js` files!

So far, so good. But what happens with we add dependencies?

To test our setup, let's try to solve a small problem.
Imagine we want to create a cli tool that takes one argument, read the `process.stdin` and print the number of lines that contained that one argument. (which would be equivalent to `grep -e myWord my-file.txt | wc -l`)

Let's create a file to handle the filesystem, I called it `src/stdin.js`:
```js
import * as fs from 'fs'

export function readStdIn () {
  return fs.readFileSync('/dev/stdin').toString()
}
```
And editing our `src/index.js`:
```js
import { filter, includes, length, pipe, split } from 'ramda'
import { readStdIn } from './stdin'

if (process.argv.length < 3) {
  console.error('please pass in the word to count')
  process.exit(-1)
}

const inputFile = readStdIn()
const argument = process.argv[2]

pipe(split('\n'), filter(includes(argument)), length, console.log)(inputFile)
```

Checking the project again:
```sh
 npm run check
> tsc -p jsconfig.json

src/index.js:1:55 - error TS2307: Cannot find module 'ramda'.

1 import { filter, includes, length, pipe, split } from 'ramda'
                                                        ~~~~~~~

src/index.js:4:5 - error TS2580: Cannot find name 'process'. Do you need to install type definitions for node? Try `npm i @types/node`.

4 if (process.argv.length < 3) {
      ~~~~~~~

src/index.js:6:3 - error TS2580: Cannot find name 'process'. Do you need to install type definitions for node? Try `npm i @types/node`.

6   process.exit(-1)
    ~~~~~~~

src/index.js:10:18 - error TS2580: Cannot find name 'process'. Do you need to install type definitions for node? Try `npm i @types/node`.

10 const argument = process.argv[2]
                    ~~~~~~~

src/stdin.js:1:21 - error TS2307: Cannot find module 'fs'.

1 import * as fs from 'fs'
                      ~~~~

Found 5 errors.
```

Wow, that was a lot.
First thing, the errors says we should add type definitions for node, let's try it:
```
yarn add -D @types/node
```
And try `check` again:

```sh
 npm run check
> tsc -p jsconfig.json

src/index.js:1:55 - error TS2307: Cannot find module 'ramda'.

1 import { filter, includes, length, pipe, split } from 'ramda'
                                                        ~~~~~~~

Found 1 error.
```

Way better, huh? Now we add ramda
```
yarn add ramda
```

But if we run `check` again we'll run into ~25 errors like this:
```
node_modules/ramda/src/view.js:22:12 - error TS2304: Cannot find name 'Lens'.

22  * @param {Lens} lens
```

What is missing? type definitions! Install it using `yarn add -D @types/ramda` and `check` will succeds.

### Testing the cli

So we wrote and type-checkd our cli, lets use it.
```sh
cat jsconfig.json | node . true # we expect see 1 as output as jsconfig.json has one line that include "true".

index.js:1
(function (exports, require, module, __filename, __dirname) { import { filter, includes, length, pipe, split } from 'ramda'
                                                              ^^^^^^

SyntaxError: Unexpected token import
    at createScript (vm.js:80:10)
    at Object.runInThisContext (vm.js:139:10)
    at Module._compile (module.js:616:28)
    at Object.Module._extensions..js (module.js:663:10)
    at Module.load (module.js:565:32)
    at tryModuleLoad (module.js:505:12)
    at Function.Module._load (module.js:497:3)
    at Function.Module.runMain (module.js:693:10)
    at startup (bootstrap_node.js:191:16)
    at bootstrap_node.js:612:3
```

Ops, it seems we are using unsupported syntax for javascript. We can solve it either by using node v13 or by transpiling.

Luckily `tsc` is a transpiler and we can easily configure it to transpile our `.js` files:

`jsconfig.json`
```json
{
  "include": [
    "src"
  ],
  "compilerOptions": {
    "checkJs": true,
    "declaration": true,
    "noEmit": false,
    "outDir": "dist",
    "module": "commonJS"
  }
}
```

To better describe what `tsc` will be doing you can rename `check` script to `build` or `compile`.
We also update the `main` key of `package.json` to `"dist/index.js"`

```sh

 npm run compile # OK
> tsc -p jsconfig.json

 cat jsconfig.json | node . true # expected 2
2

 tree dist
dist
├── index.d.ts
├── index.js
├── stdin.d.ts
└── stdin.js

0 directories, 4 files

```

As you can see, `tsc` created four files under `dist` dir. One transpiled `.js` for each `.js` in our `src` and two `.d.ts` files within generated type definitions.

`dist/stdin.d.ts`
```ts
export function readStdIn(): string;

```

### TL;DR

To wrap up what we've learned

1. we create a new javascript project
2. used `jsconfig.json` to configure our project
3. imported `typescript` and created a npm script that uses `tsc -p jsconfig.json`
4. imported `@types/node`, `ramda` `@types/ramda`
5. created a small cli and compiled it using that npm script

I hope this tutorial was helpfull. You can reach me out at [@munizart](https://github.com/munizart) and [@hurricanecart](https://twitter.com/hurricanecart).

Thank you for reading.
