---
layout: post
title: "How to unit test vscode extensions with basic mocks"
date: 2019-09-30 18:00:00 -0700
categories: vscode testing mocking
---

One of the major tools in any testers toolbelt is the mighty module mock. So when I started looking into adding proper unit tests to my vscode extension, I surprisingly had quite a difficult time finding something that would work gracefully with vscode's test harness. Seeming I have also been tinkering with go and the absolutely fantastic gitbook [learn-go-with-tests](https://quii.gitbook.io/learn-go-with-tests/), I decided to try my hand at creating a simple mocking structure for [vscode-coverage-gutters](https://github.com/ryanluker/vscode-coverage-gutters). This was in contrast to using something established like [sinon](https://sinonjs.org). Big warning before I dive in though, this implementation works well for this specific use case but should not be taken as vscode extension testing gospel!

Let's quickly breakdown the problem before we go into our very basic mocking implementation.
1) We need a way to mock module functionality that are used inside our extension (vscode, requests, etc).
2) We need to swap the current functionality of the module with our test expectation
3) Once we have completed a test case, we need to clean up after ourselves to prevent test leakage.

Alright, with our requirements outlined let's dive in! The first critical piece of understanding is how we can mock modules inside the test case without directly affecting the piece of code under test. This is where having an understanding of the nodejs import [caching system](https://nodejs.org/api/modules.html#modules_caching) comes into play. Using the cache ,to effect extension wide changes, we can do something like this for mocking the `fs.readFile`.

```ts
import * as assert from "assert";
import * as fs from "fs";

// Original functions
const readFile = fs.readFile;

teardown(function() {
    (fs as any).readFile = readFile;
});

test("#load: Should reject when readFile returns an error @unit", function(done) {
    const readFile = function(path: string, cb) {
        assert.equal(path, "pathtofile");
        const error: NodeJS.ErrnoException = new Error("could not read from fs");
        return cb(error, Buffer.from(""));
    };
    (fs as any).readFile = readFile;

    const coverage = new Coverage(
        fakeConfig,
    );
});
```

Let's break this example up a bit so we can understand the 3 stages of the module mocking.
```ts
// Original functions
const readFile = fs.readFile;

teardown(function() {
    (fs as any).readFile = readFile;
});
```
In the first stage we store the original function so that we can "restore" this module to an original state later. You can see this restoring action in the teardown snippet, which runs after each test case has been completed.

Next we see us changing the functionality of the `readFile`.
```ts
const readFile = function(path: string, cb) {
    assert.equal(path, "pathtofile");
    const error: NodeJS.ErrnoException = new Error("could not read from fs");
    return cb(error, Buffer.from(""));
};
(fs as any).readFile = readFile;
```
After this point any call to `fs.readFile` in any code flow, we really only care about the functionality under test though, will utilise the test function defined just above. This state will persist for the rest of test case until the teardown is called and restores the stored original functionality.

You can find this and more mocking examples [here](https://github.com/ryanluker/vscode-coverage-gutters/pull/218/files).

Wrapping up, we see that using the node module caching allows us to "swap" out functionality of specific modules for mock flows that help us verify code under test. This clean approach of storing the original functionality, modifying the functionality with a mock and finally restoring the module to an original state, feels like quick a succinct pattern. Also it allows us to accomplish our main goal of testing exactly the conditional nuance we want in each test case.

I love to discuss technical implementations so drop a comment in the [github discussion](https://github.com/ryanluker/ryanluker.github.io/issues/3) if you have any advice or questions!