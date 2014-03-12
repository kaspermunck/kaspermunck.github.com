---
layout: post
title: "Fix 'clang: unknown argument' error after updating to Xcode 5.1"
date: 2014-03-12 13:37
comments: true
categories:
- clang
- unknown argument
- xcode 5.1
---

### Error

Updating Xcode to version 5.1 seems to have an impact on certain ruby gems. In particular for me, when trying to install the `json` gem, the process exits with 1 and the following error (among others):

    compiling generator.c
    linking shared-object json/ext/generator.bundle
    clang: error: unknown argument: '-multiply_definedsuppress' [-Wunused-command-line-argument-hard-error-in-future]
    clang: note: this will be a hard error (cannot be downgraded to a warning) in the future

The message from clang is quite specific in that it states the error commited (`unknown argument`), which warning (`-Wunused-command-line-argument-hard-error-in-future`) and a note ('this will be a hard error...'). It is certain from the information given that this has something to do with clang and very likely something with warnings being treated as error.

### Reason

The reason for this sudden error is found in the [Xcode Release Notes](https://developer.apple.com/library/ios/releasenotes/DeveloperTools/RN-Xcode/Introduction/Introduction.html):

> The Apple LLVM compiler in Xcode 5.1 treats unrecognized command-line options as errors. This issue has been seen when building both Python native extensions and Ruby Gems, where some invalid compiler options are currently specified.

It seems that the newer version of the llvm compiler shipping with Xcode 5.1 is a little more restrictive when it comes to warnings. Furthermore it says that:

> Projects using invalid compiler options will need to be changed to remove those options.

That is, developers should not expect this change to be reverted in the future.

### Fix

While the real error lays in the particular gem, there is a temporary solution, which is to downgrade the error to a warning. This can be done by telling the compiler to not treat unused-command-line-arguments as errors:

    ARCHFLAGS=-Wno-error=unused-command-line-argument-hard-error-in-future gem install GemName

As explicitly noted in the release notes, this is a temporary fix. It does, however, solve the immediate problem (not being able to install certain gems), and is sufficient untill the maintainers of the gems update their projects to not violate the treat-warnings-as-errors constraint.
