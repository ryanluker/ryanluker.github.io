---
layout: default
title: "Testing Visual Studio Code Extensions"
date: 2018-06-26 22:00:00 -0700
categories: typescript extension testing
---

Testing is a key focus of mine whenever building software, doubly so when people depend on it! Integration testing is a valuable part of the testing pyramid and one that I found difficulty in finding examples to use in [vscode-coverage-gutters](https://github.com/ryanluker/vscode-coverage-gutters). In the next few code snippets I outline some of the integration tests I wrote during development. I also dive into what confidence they provide me, in terms of not breaking current features with future work, along with some of the critical api's I used.

This first example is rather simple (really just a warmup for us). It tests for the extension being properly started by the testing harness. This allows for me to confirm that the extension activation step has completed successfully.

<script src="https://gist.github.com/ryanluker/a9ac95c0d477aafd1b0906edcf3a0680.js"></script>

It uses the [getExtension](https://code.visualstudio.com/docs/extensionAPI/vscode-api#extensions.getExtension) function on the vscode api to fetch information about the installed extension. I then simply assert that it is active and ready to accept commands.

The next example is much more complex and provides a lot of coverage to many areas of the extension such as the file loader, coverage generation, lcov parser, etc. The goal of the test is to confirm that when a file is opened in the IDE (this file must also have coverage in the [example project](https://github.com/ryanluker/vscode-coverage-gutters/tree/master/example/node)) we check that the code coverage lines have been calculated with the expected values.

<script src="https://gist.github.com/ryanluker/636d39686f65495a63e2cc88002603b3.js"></script>

Many of the steps in this test are for setting up the IDE in a way that allows for the proper running of the `displayCoverage` command. You can see on line 3 we get the exported api provided by vscode-coverage-gutters that will allow us to later fetch the lines of coverage. Next in lines 4-6 we setup the IDE to have the test document open and ready to have coverage displayed.

Finally, you can see we assert that the coverage received from the exported api includes the expect coverage values. A critical piece of this test is the [`getCachedLines`](https://github.com/ryanluker/vscode-coverage-gutters/blob/v2.0.0/src/exportsapi.ts) which stores the last rendered coverage and exports the cache for use in tests or by other extension developers. Usually I would not depend on a cache like this for testing purposes but currently there is no way to inspect what decorations have been displayed to the IDE via the vscode api [[1]](https://github.com/Microsoft/vscode/issues/48364).

A similar example but for the xml-based coverage can be found below as well.

<script src="https://gist.github.com/ryanluker/c10df3c6e24b823c3aa8578a2978e51f.js"></script>

You can find all of the example test snippets above in [`extension.test.ts`](https://github.com/ryanluker/vscode-coverage-gutters/blob/v2.0.0/test/extension.test.ts), hopefully they provide useful examples to allow others to test their vscode extensions.

If you wish to chat about this blog post, you can find the link below!
[`Github Discussion`](https://github.com/ryanluker/ryanluker.github.io/issues/1)