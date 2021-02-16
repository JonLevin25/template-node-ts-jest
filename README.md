# Fork Changes from original [Jon]
* `launch.json`
  * Didn't work on windows, seems like assums mac/linux?
  * Used this launch.json instead: \([master][new-launch-json-master] | [actual commit taken][new-launch-json-commit]\)
* `tsconfig.json`
  * Changed target to `es2017` + include `"lib": ["es2017"]`
  * Removed `strict` + `noImplicitAny` restrictions
* `.gitignore`
  * Used [Github node GitIgnore](https://github.com/github/gitignore/blob/master/Node.gitignore)
* `Readme`
  * Added this section


<!-- Markdown Links -->

[new-launch-json-master]: https://github.com/Lemoncode/jest-vs-code-debugging-example/blob/master/custom-solution-jest-config-file/01-implemented/.vscode/launch.json

[new-launch-json-commit]: https://github.com/Lemoncode/jest-vs-code-debugging-example/blob/1f6a82eb773bc96e8f75709cb41d4da8cc8858ec/custom-solution-jest-config-file/01-implemented/.vscode/launch.json


<br />
<br />
<br />
<br />


# Sample Project using ts-jest

The goal in this project is to create a TypeScript project that can do all of the following:

* Compile code as an es5 library that can be published as a Node module with typings.
* Using `jest` and `ts-jest` for testing
* Provides proper stack traces for failed tests
* Provide accurate code coverage metrics
* Can be debugged using the Node debugger with proper source maps

## Basic Setup

### Module and Dependencies

I start by initialing this as an `npm` project.

```sh
$ yarn init .
```

Then, I install `typescript`, `jest`, `ts-jest` and `@types/jest` as dependencies:

```sh
$ yarn add -D typescript jest ts-jest @types/jest
```

At the time of this writing, that means `typescript@2.7.1`, `jest@22.1.4` and `ts-jest@22.0.2`.

### TypeScript

Next, we initialize this as a TypeScript project using:

```sh
$ npx tsc --init .
```

I want my TypeScript generated code to be stored in `./lib` and I want declarations generated.
So, I configure `outDir` in `tsconfig.json` to be `./lib`.

### Files

My `.gitignore` is then configured to be:

```sh
/node_modules
/lib
```

...while my `.npmignore` is just:

```sh
/node_modules
```

For the same reason, I remove the default value for `files` in `tsconfig.json` and replace it with:

```json
    "exclude": ["node_modules", "lib"]
```

### Source

To start, I create a `src/index.ts` that contains a simple function:

```typescript
export function sampleFunction(x: string): string {
    return x + x;
}
```

I also add a simple `jest` test. I prefer to keep my tests in a completely separate
location, so I'll put all my tests in `__tests__`. So I create the following test case
in `__tests__/base.spec.ts`:

```typescript
import { sampleFunction } from "../src";

describe("This is a simple test", () => {
    test("Check the sampleFunction function", () => {
        expect(sampleFunction("hello")).toEqual("hellohello");
    });
});
```

### Configurating Jest

At this point, I'd like to run that test. But first I need to create a `jest.config.js`
file for all my `jest` settings. This has to take into account the fact that I'm using
`ts-jest` and the fact that my tests are stored in `__tests__`. So the resulting file
looks like this:

```js
module.exports = {
    transform: {
        "^.+\\.tsx?$": "ts-jest",
    },
    testRegex: "(/__tests__/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$",
    moduleFileExtensions: ["ts", "tsx", "js", "jsx", "json", "node"],
};
```

### Scripts

I then add the following scripts to `package.json`:

```json
  "scripts": {
    "compile": "tsc",
    "test": "jest"
  }
```

At this point, if I run `yarn test`, I get exactly what I was hoping for:

```
 PASS  __tests__/base.spec.ts
  This is a simple test
    ✓ Check the sampleFunction function (3ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
```

## Code Coverage

### Configuration

To enable code coverage, I update my `jest.config.js` file to:

```js
module.exports = {
    transform: {
        "^.+\\.tsx?$": "ts-jest",
    },
    testRegex: "(/__tests__/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$",
    moduleFileExtensions: ["ts", "tsx", "js", "jsx", "json", "node"],
    collectCoverage: true,
};
```

I'll also want to update my `.gitignore` and `.npmignore` files to avoid version
controlling or publishing the `coverage` directory generated by `jest`.

### Code Organization

At this point, I'm going to start introducing sub-modules in my project. So I'll add a
`src/core` and a `src/utils` module just so make things sligtly more realistic. Then I'll
export the contents of both of these so that `src/index.ts` looks like this:

```typescript
export * from "./core";
export * from "./utils";
```

These then import specific files containing various types and functions. Initially, I'll
create a very simple set of types for representing extremely simple expressions with only
literals and the binary operations `+`, `-`, `*` and `/`. Then I can write a few tests
like these:

```typescript
import { evaluate, Expression } from "../src";

describe("Simple expression tests", () => {
    test("Check literal value", () => {
        expect(evaluate({ type: "literal", value: 5 })).toBeCloseTo(5);
    });
    test("Check addition", () => {
        let expr: Expression = {
            type: "binary",
            operator: "+",
            left: {
                type: "literal",
                value: 5,
            },
            right: {
                type: "literal",
                value: 10,
            },
        };
        expect(evaluate(expr)).toBeCloseTo(15);
    });
});
```

So far so good. But note that if I actually run these tests, I get these results:

```
 PASS  __tests__/base.spec.ts
  Simple expression tests
    ✓ Check literal value (4ms)
    ✓ Check addition

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        2.048s
Ran all test suites.
---------------|----------|----------|----------|----------|----------------|
File           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
---------------|----------|----------|----------|----------|----------------|
All files      |    66.67 |     37.5 |       50 |    66.67 |                |
 src           |      100 |      100 |      100 |      100 |                |
  index.ts     |      100 |      100 |      100 |      100 |                |
 src/core      |    61.54 |     37.5 |      100 |    61.54 |                |
  functions.ts |    54.55 |     37.5 |      100 |    54.55 | 14,16,18,20,25 |
  index.ts     |      100 |      100 |      100 |      100 |                |
 src/utils     |    66.67 |      100 |        0 |    66.67 |                |
  checks.ts    |       50 |      100 |        0 |       50 |              2 |
  index.ts     |      100 |      100 |      100 |      100 |                |
---------------|----------|----------|----------|----------|----------------|
```

Note the lack of code coverage. Adding a few more test cases along with some
`/* istanbul ignore ... */` comments to let `istanbul` know what it can safely
ignore, we get to:

```
 PASS  __tests__/base.spec.ts
  Simple expression tests
    ✓ Check literal value (3ms)
    ✓ Check addition
    ✓ Check subtraction
    ✓ Check multiplication (1ms)
    ✓ Check division

Test Suites: 1 passed, 1 total
Tests:       5 passed, 5 total
Snapshots:   0 total
Time:        1.353s
Ran all test suites.
---------------|----------|----------|----------|----------|----------------|
File           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
---------------|----------|----------|----------|----------|----------------|
All files      |      100 |      100 |      100 |      100 |                |
 src           |      100 |      100 |      100 |      100 |                |
  index.ts     |      100 |      100 |      100 |      100 |                |
 src/core      |      100 |      100 |      100 |      100 |                |
  functions.ts |      100 |      100 |      100 |      100 |                |
  index.ts     |      100 |      100 |      100 |      100 |                |
 src/utils     |      100 |      100 |      100 |      100 |                |
  checks.ts    |      100 |      100 |      100 |      100 |                |
  index.ts     |      100 |      100 |      100 |      100 |                |
---------------|----------|----------|----------|----------|----------------|
```

Now, if we change a test to make it fail, we get something like this:

```
● Simple expression tests › Check division

    expect(received).toBeCloseTo(expected, precision)

    Expected value to be close to (with 2-digit precision):
      1
    Received:
      2

      19 |     test("Check division", () => {
      20 |         let expr = bin("/", 10, 5);
    > 21 |         expect(evaluate(expr)).toBeCloseTo(1);
      22 |     });
      23 | });
      24 |

      at Object.<anonymous> (__tests__/base.spec.ts:21:32)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 4 passed, 5 total
Snapshots:   0 total
Time:        1.535s
Ran all test suites.
---------------|----------|----------|----------|----------|----------------|
File           |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
---------------|----------|----------|----------|----------|----------------|
All files      |      100 |      100 |      100 |      100 |                |
 src           |      100 |      100 |      100 |      100 |                |
  index.ts     |      100 |      100 |      100 |      100 |                |
 src/core      |      100 |      100 |      100 |      100 |                |
  functions.ts |      100 |      100 |      100 |      100 |                |
  index.ts     |      100 |      100 |      100 |      100 |                |
 src/utils     |      100 |      100 |      100 |      100 |                |
  checks.ts    |      100 |      100 |      100 |      100 |                |
  index.ts     |      100 |      100 |      100 |      100 |                |
---------------|----------|----------|----------|----------|----------------|
```

Note that the stack track is correct. It points to the problem **in the TypeScript
code**.

## Compilation

Recall that we added a `compile` script to our `package.json`. We can compile the
code with `yarn compile`. Doing so, we see that the `lib` directory is populated
with two subdirectories, `src` and `__tests__`.

However, if we look in those directories, we will find that they only include
the generated Javascript code. They do not include type definitions. In order
to generate type definitions (`.d.ts` files) so that other TypeScript users can
benefit from all the type information we've added to our code, we have to set
the `declaration` field in our `tsconfig.json` file to be `true`.

Also note that in order for others to use this package as an NPM module, you need
to set the `main` field in `package.json` to `lib/src/index.js`. Furthermore, in
order for others to be able to access the types in this module, we also need to
set the `typings` field in `package.json` to `lib/src/index.d.ts`. In other words,

```json
    "main": "lib/src/index.js",
    "typings": "lib/src/index.d.ts",
```

If properly configured, we can then launch a `node` session and import our new package:

```sh
$ node
> var me = require(".")
undefined
> me
{ evaluate: [Function: evaluate],
  assertNever: [Function: assertNever] }
>
```

Now be sure to update your `jest.config.js` to include the following setting or `jest`
will start matching the code in the `lib/__tests__` directory:

```json
    testPathIgnorePatterns: ["/lib/", "/node_modules/"],
```

## Debugging

Finally, we come to debugging. I'm using Visual Studio Code, so I'll demonstrate how to
get debugging working there. Some of this information may very well translate to other
IDEs.

In VSCode, we can go to the debugging sidebar. Initially, next to the "play" button
will be the words "No Configuration". Clicking on that brings up a pull-down menu
with an option "Add Confiuration...".

As much as I love TypeScript, debugging is really its Achilles Heel. It isn't that you
cannot debug, it is that it is just difficult to get working. If you select "Add Configuration..." and then "Node.js", you'll see several preconfigurations including
one for `mocha`. But there isn't one for `jest`. So you'll have to create your
own `.vscode/launch.json` file. Fortunately, the `jest` page suggestions you create
a `.vscode/launch.json` file that looks like this:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Jest Tests",
            "type": "node",
            "request": "launch",
            "runtimeArgs": ["--inspect-brk", "${workspaceRoot}/node_modules/.bin/jest", "--runInBand"],
            "console": "integratedTerminal",
            "internalConsoleOptions": "neverOpen"
        }
    ]
}
```

I was pleasantly surprised to find that I could not only run my tests and get code coverage
as usual, but also set breakpoints in both the tests (_i.e.,_ in `__tests__/base.spec.ts`)
as well as in the code (_e.g.,_ `src/core/functions.ts`) and the debugger will find them.

Note that I tested all this on Node 8.x. I've seen issues with debugging using Node 6.x so
if you are having trouble there, you might consider upgrading (or let, if you manage to fix
it, submit a PR for this README explaining the fix).

