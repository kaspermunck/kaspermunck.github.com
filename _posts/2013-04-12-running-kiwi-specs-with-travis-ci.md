---
layout: post
title: "Running Kiwi Specs with Travis-CI"
date: 2013-04-12 23:33
categories: 
- travis-ci
- kiwi
- xcodetest
- xcodebuild
---

After [Travis announced support for Objective-C](http://about.travis-ci.org/blog/introducing-mac-ios-rubymotion-testing/) projects I have tried to make it work with Kiwi since it's my preferred testing framework. It wasn't as easy as I thought, but with a little help from some [awesome](https://github.com/sgleadow/xcodetest) [tools](https://github.com/jonathanpenn/WaxSim) I managed to make it work. Here's how I did it, step-by-step.

## Installing Kiwi

I use [CocoaPods](http://cocoapods.org) to manage dependencies. First, create a new project and make sure to include unit tests.

![Create Project](/images/create-new-project.png)

Close the project and navigate to the project folder. Create a file named Podfile and copy the contents of [this file](https://github.com/kaspermunck/boiler-plates/blob/master/Kiwi/Podfile) into it. Then run `pod install` from the command line to inject Kiwi into the project. Open MyApp.xcworkspace and hit `cmd+b` to verify that the project builds and `cmd+u` to verify that the tests also run.

If you get an error like this:

    clang: error: no such file or directory: '/Users/. . .prefix.pch.dia'

when you try to run your tests, then go to the _Pods_ target and in _Pods-MyAppTests.xcconfig_ in _Target Support Files -> Pods-MyAppTests_, change the line that reads:

    FRAMEWORK_SEARCH_PATHS = $(inherited) "$(SDKROOT)/Developer/Library/Frameworks" "$(DEVELOPER_LIBRARY_DIR)/Frameworks" "$(inherited)"

to

    FRAMEWORK_SEARCH_PATHS = $(inherited) "$(SDKROOT)/Developer/Library/Frameworks" "$(DEVELOPER_LIBRARY_DIR)/Frameworks"

Try `cmd+u` again and everything should be fine. Now delete `MyAppTests.h` and replace the contents of `MyAppTests.m` with:

    SPEC_BEGIN(ViewControllerSpecs)

	// logic test
    it(@"should always succeed", ^{
        [[theValue(YES) should] equal:theValue(YES)];
    });

	SPEC_END
	
Run tests with `cmd+u` and verify that it succeeds.

## Testing From the Terminal

A build server is going to run the tests so they must be invokable from the command line. First we create a unit test scheme which we name after our test target (MyAppTests). Click on _MyApp_ next to the Stop button in the upper left corner and select Manage Schemes.

![New Scheme](/images/new-scheme.png)

Make sure that both schemes (MyApp and MyAppTests) have Shared enabled. Otherwise, they will not be exported and cannot be reached by Travis.

![Shared Schemes](/images/shared-schemes.png)

Select MyAppTests and click Edit. Under the Build tap check Test and Run for MyAppTests.

![Build Configuration](/images/build-config.png)

Done. 

Xcode has a command line utility named [xcodebuild](https://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html) with which you can build from outside of Xcode. By passing it `TEST_AFTER_BUILD=YES` we can run tests from the command line. Note that we set the sdk to iphonesimulator; if you don't do that you might wind up with signing errors. Run this from the command line:

    xcodebuild -workspace MyApp.xcworkspace -scheme MyAppTests -sdk iphonesimulator -configuration Release TEST_AFTER_BUILD=YES

The parameters pretty much explain themselves. It will output plenty of stuff, but you only need to verify that, somewhere near the bottom, it reads:

    ** BUILD SUCCEEDED **

We are not truely convinced yet, so try and break our test:

    [[theValue(YES) should] equal:theValue(NO)];
	
And hit `cmd+u` to see it break (in Xcode). After that, run it from command line again. The output should still be: 

    ** BUILD SUCCEEDED **

So, what happens when we run our tests is that a script named RunPlatformUnitTests, located in `/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/Tools/` is run. In RunPlatformUnitTests's main function it reads:

    if [ "${TEST_HOST}" != "" ]; then
        # All applications are tested the same way, by injecting a bundle.
        # The bundle needs to configure and run the tests itself somehow.

        RunTestsForApplication "${TEST_HOST}" "${TEST_BUNDLE_PATH}"
    else
        # If no TEST_HOST is specified, assume we're running the test bundle.
        
        RunTestsForBundle "${TEST_BUNDLE_PATH}"
    fi
    
And that RunTestsForApplication is the guy that gives us the warning instead of running our tests:

    RunTestsForApplication() {
    Warning ${LINENO} "Skipping tests; the iPhoneSimulator platform does not currently support application-hosted tests (TEST_HOST set)."
    }

Let's try to add `TEST_HOST=''` to the end of the build command and run it again to see if we can fool the build script. The output should now read something similar to:

    Executed 1 test, with 1 failure (1 unexpected) in 0.022 (0.023) seconds
    
It worked! But nothing comes that easy with Apple. Now create a ViewController with a nib file (don't forget to add it to the test target). Add a UIButton to the view and connect it to an IBOutlet named button. Then, in the test file, add a new spec:

	// application test
    it(@"should connect button", ^{
        ViewController *vc = [[ViewController alloc] init];
        [vc view]; // init view hierarchy
        [vc.button shouldNotBeNil]; // test IBOutlet connection
    });

Fix the other spec (revert `NO` to `YES`) and hit `cmd+u`. The tests pass. Now run the tests from the terminal again.

	/Applications/Xcode.app/Contents/Developer/Tools/RunPlatformUnitTests.include:448: error: Failed tests for architecture 'i386' (GC OFF)
	
    ** BUILD FAILED **
    
## Application vs Logic Unit Tests

Even though we managed to fool the build script by adding `TEST_HOST=''` it didn't help us much, since, as it says, all applications are tested by injecting a bundle. [According to the docs](https://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/UnitTesting/01-Unit-Test_Overview/overview.html#//apple_ref/doc/uid/TP40002143-CH2-SW1) Xcode differentiates between between application- and logic unit tests, the first being tests that need to run inside a UIKit environment, the latter tests that don't. There's more on that [here](http://www.stewgleadow.com/blog/2012/02/09/running-ocunit-and-kiwi-tests-on-the-command-line/). To get any further from here we'll use [XcodeTest](https://github.com/sgleadow/xcodetest) by [Stewart Gleadow](http://www.stewgleadow.com).

## XcodeTest

> A way of running your application unit tests from the command line.

XcodeTest depends on [WaxSim](https://github.com/jonathanpenn/WaxSim) which is _A wrapper around iPhoneSimulatorRemoteClient, to install and run apps in the iOS simulator._ Install WaxSim by first cloning the repo:

    git clone https://github.com/jonathanpenn/WaxSim.git
    
Then, `cd WaxSim` and run:

    sudo xcodebuild install DSTROOT=/ INSTALL_PATH=/usr/local/bin

Now, follow the installation guide in XcodeTest's README. Beware,  `build_and_run_unit_tests.sh` [might need to be tweeked](https://gist.github.com/kaspermunck/5375284) in order to make it work with Kiwi, since we installed it with CocoaPods and are therefore using a workspace. Run the tests again with XcodeTest:

    ./build_and_run_unit_tests.sh MyApp MyAppTests
    
You should see it build, open the simulator and run the tests (both application and logic) without any problems. For the sake of verification, break and fix your tests a few times to see that it works properly.

## Travis-CI

Now that we have all kinds of unit tests running from the command line, we are ready to move on. [Travis-CI](http://travis-ci.org) is free, hosted [continuos integration](http://en.wikipedia.org/wiki/Continuous_integration) for open source projects.

Basically, you sign up with your Github account and select which repos you want it to monitor. Everytime you push to those repos, Travis will build and notify you if anything went wrong. Best of all: [It is highly customizable](http://about.travis-ci.org/docs/user/build-configuration/) and it does a lot of stuff for automatically (for example runs `pod install` prior to all builds if a Podfile exists).

Since we now depend on WaxSim to run our tests in the Simulator (XcodeTest is integrated in our project so that's no problem), we need to configure Travis to install the dependency before attempting to build our project. For that purpose, use [this shell script](https://gist.github.com/kaspermunck/5374751) and put it in the root of your project folder. Make sure it's executable (`chmod +x`) Now, also in the root, create a file named `.travis.yml` and paste in the following

    # tell travis that this is an objective-c project
    language: objective-c
    
    # install xcodetest dependencies
    before_install: ./install_dependencies.sh
    
    # build via xcodetest
    script: ./build_and_run_unit_tests.sh MyApp MyAppTests
   
`.travis.yml` is the build config file that Travis will look for when building your project. This is where [all the magic](http://about.travis-ci.org/docs/user/build-configuration/) happens. Travis uses a [default build script](https://gist.github.com/henrikhodne/73151fccea7af3201f63), but that doesn't quite suite our needs because of the special case with application unit tests.

OK, now add, commit and push everything and go to the build page ([tracis-ci.org](http://travis-ci.org) -> My Repositories) tap your project, wait for it to build, scroll down to the bottom and verify that it says:

    =================
    Unit Tests Passed
    =================
    
That's it. It takes a little bit of configuration, but from now on, Travis is watching you. Go and try it yourself; break one of the tests, commit and push and then wait. Cool, right?

If it doesn't work, double check that:

- All the involved schemes (also the AppName scheme) are shared.
- `install_dependencies.sh` is executable.
- Your .gitignore does not ignore xcworkspaces (the default Github .gitignore for Objective-C projects does that).