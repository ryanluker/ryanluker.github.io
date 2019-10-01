---
layout: post
title:  "How to unit test vscode extensions with basic mocks"
date:   2019-09-30 18:00:00 -0700
categories: software development vscode testing mocking
---

One of the major tools in any testers toolbelt is the mighty module mock.
So when I started looking into adding proper unit tests to my vscode extension,
I surprisingly had quite a difficult time finding something that would work gracefully
with vscode's test harness. Seeming I have also been tinkering with go and the [absolutely fantastic gitbook](https://quii.gitbook.io/learn-go-with-tests/) I decided to try my hand at creating a simple mocking structure for [vscode-coverage-gutters](https://github.com/ryanluker/vscode-coverage-gutters). Big warning before I dive in though, this implementation works well for this specific use case but should not be taken as vscode extension testing gospel!

Lets quickly breakdown the problem before we go into our very basic mocking implementation.
1) We need a way to mock module functionality that are used inside our extension (vscode, requests, etc).
2) We need to swap the current functionality of the module with our test expectation
3) Once we have completed a test case, we need to cleanup after ourselves to prevent test leakage.

Alright, with our requirements outlined lets dive in! The first critical piece of understanding is how we can mock modules inside the test file without directly effecting the piece of code under test. This is where having an understanding of the nodejs import [caching system](https://nodejs.org/api/modules.html#modules_caching) comes into play. Using the cache to effect extension wide changes we can do something like this for mocking the `fs.readFile`.

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
https://github.com/ryanluker/vscode-coverage-gutters/blob/v2.4.0/test/coverage-system/coverage.test.ts#L8-L50

Lets break this example up a bit so we can understand the 3 stages of the module mocking.
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
After this point any call to `fs.readFile`, in any code flow but we really only care about the functionality under test, will utilise the test function defined just above. This state will persist for the rest of test case until the teardown is called and restores the stored original functionality.

You can find more mocking examples in this style [here](https://github.com/ryanluker/vscode-coverage-gutters/pull/218/files).

conclusion
- reiterate hook
- present final thinkings / results
- call to discussion