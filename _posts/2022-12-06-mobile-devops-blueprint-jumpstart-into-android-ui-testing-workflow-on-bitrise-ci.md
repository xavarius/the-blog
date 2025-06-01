---
layout: default
title: "Mobile DevOps Blueprint: Jumpstart into Android UI testing workflow on Bitrise CI"
date: 2022-12-06
---

![Illustration of a child jumping on a trampoline](/assets/1*i5a9u-Ec_NYsMcKm1e8MYw.jpeg)
*Image source: [Creative Commons Attribution](https://vectorportal.com/pt/vector/crian%C3%A7a-pulando-em-um-trampolim/35471)*

### Mobile DevOps Blueprint: Jumpstart into Android UI testing workflow on Bitrise CI

#### Just run those tests in minutes

#### The Challange
I want to run UI tests on Bitrise CI **asap**. I have a sample test suite
or I’m migrating from another CI. Perhaps I want to customize the workflow a bit to run custom annotation that scopes the test suite or sets device config, etc.

#### **Who would benefit?**
DevOps, Automation Engineer, Android Dev, Tech Lead.

#### Assumptions
I’m assuming you’re a beginner to intermediate Bitrise user, or you’re
a skilled professional looking for a shortcut to just run your test suite asap. What I’m providing here is supposed to be a blueprint for you to jumpstart running your tests on Bitrise CI.

This article will cover **the Mobile DevOps** side of how to kick off UI testing on Bitrise for Android. Code included. I’ll also share a couple
of good practices for UI automation configuration.

I’m assuming you want a *quasi-infrastructure-as-a-code* approach, meaning you are going to use *bitrise.yaml* instead of Bitrise web console.
In fact, please use *bitrise.yaml* + version control system to maintain control over changes and learn more.

I’m assuming you have configured the auth mechanism between your repository and Bitrise, meaning you can clone your repo while running Bitrise builds.

I’m assuming you know where to find the most recent *bitrise.yaml* version. BTW See Workflow Editor -> *bitrise.yaml* tab and download it.

The Firebase Device Lab will be used to run the automation suite.
The default account is linked with your Bitrise instance — it’s recommended to provide your own account.

If you would like to learn in-depth Bitrise concepts and how to navigate and use it in an optimized way, I would recommend you read [my article on being effective with Bitrise CI](https://proandroiddev.com/be-effective-with-bitrise-ci-for-android-lessons-i-learned-the-hard-way-5a85e45a33dc). These two articles will share a few tips for sure.

#### The Blueprint
```yaml
  runTests:
    steps:
      - git-clone: { }
      - cache-pull@2: { }
      - install-missing-android-tools@3: { }
      - android-sdk-update@1: { }
      - android-build-for-ui-testing@0:
          inputs:
            - variant: debug
            - module: app
      - virtual-device-testing-for-android@1:
          is_skippable: true
          inputs:
            - test_type: instrumentation
            - test_devices: "Pixel2,28,en_US,portrait"
            - inst_test_targets: "annotation com.example.annotation.SMOKE"
            - inst_use_orchestrator: 'true'
            - test_timeout: 3600
      - custom-test-results-export@0.1:
          inputs:
            - test_name: "*"
            - search_pattern: "*"
      - cache-push@2: { }
```
Just basic configs, nothing to see here, really. *runTest* steps live alongside other steps.

```yaml
  runTests: #name of the workflow
    steps:
      - git-clone: { } # clones repo, more config may be required, out of scope
      - cache-pull@2: { } # gets prev cached artifacts, faster build
      - install-missing-android-tools@3: { } # update
      - android-sdk-update@1: { } # update
# this
# is
# where
# testing
# happens
      - custom-test-results-export@0.1: # Test reports. You want them.
          inputs:
            - test_name: "*"
            - search_pattern: "*"
      - cache-push@2: { } # push artifacts
# Optionally you may like Slack notification
```
As said before, no magic happens here. Just housekeeping like cloning
the repository and being sure things are up to date.
Keep an eye on reports - those are useful to track what’s exactly wrong with your test including screenshots, logs, state, execution time, etc.

Ok, let’s explain the UI tests running part.
```yaml
runTests:
    steps:
      # Test builds
      - android-build-for-ui-testing@0:
          inputs:
            - variant: debug
            - module: app
      # Testing
      - virtual-device-testing-for-android@1:
          is_skippable: true
          inputs:
            - test_type: instrumentation
            - test_devices: "Pixel2,28,en_US,portrait"
            - inst_test_targets: "annotation com.example.annotation.SMOKE"
            - inst_use_orchestrator: 'true'
            - test_timeout: 3600
```

#### How test builds are being prepared?
```yaml
# Test builds creation
      - android-build-for-ui-testing@0:
          inputs:
            - variant: debug
            - module: app
```
No magic here again. **Both** your APK and testing APK will be built as a part of these steps. Bitrise will figure out the paths to APKs itself.
If you want to see the guide — here is [the official one](https://www.bitrise.io/integrations/steps/android-build-for-ui-testing).

#### The UI testing steps
```yaml
- virtual-device-testing-for-android@1:
          is_skippable: true
          inputs:
            - test_type: instrumentation
            - test_devices: "Pixel2,28,en_US,portrait" #example
            - inst_test_targets: "annotation com.example.annotation.SMOKE" #OPTIONAL
            - inst_use_orchestrator: 'true'
            - test_timeout: 900
```
Step by step
```yaml
is_skippable: true # not required
```
You may want to have a green build even though the UI test failed because some of the tests are flaky and brittle. Maybe you just care about configuration (for now) or maybe you’re running an experiment. It’s useful. Use it whenever necessary.
```yaml
- test_type: instrumentation # required, other: game loop, robo
```
In most cases, Android UI tests are instrumentation tests. Ask Automation Engineer if that’s the truth for you.
```yaml
# required, schema deviceID,version,language,orientation
- test_devices: "Pixel2,28,en_US,portrait"
```
You have to provide at least one device, you can provide many.
Separated by comma.

You can find config details and a list of devices on [the official Bitrise Github step codebase](https://github.com/bitrise-steplib/steps-virtual-device-testing-for-android/blob/master/step.yml).
```yaml
- inst_test_targets: "annotation com.example.annotation.SMOKE" # optional
```
You can define test targets — if not provided, **every** test in *androidTest* package will be triggered. You’re required to provide a **fully qualified** class or a package.

You can also configure your test targets via annotations. Annotations give you the opportunity to scope out what test to run by grouping them. It’s up to you what, how, and when to group which gives you the opportunity to adapt to the current context (release, feature branch or smoke testing?) — it's all up to you and can be done both manually and dynamically.
A short and sweet article on annotation is found [here](https://evanfang.medium.com/run-specific-android-espresso-tests-by-creating-custom-annotations-using-kotlin-and-command-line-6a90e8728e3b).
```yaml
- inst_use_orchestrator: 'true' # optional, default false
```
This listing defines if tests are using Android Test Orchestrator or not. Default: false. Ask your Automation Engineer.
```yaml
- test_timeout: 900 # optional, default 900=15m
```
A question: How much time must pass before **a test** is canceled?
Default 900 = 15m. I would assume it should be much quicker for an efficient automation suite.

But [there](https://github.com/bitrise-steplib/steps-virtual-device-testing-for-android/blob/master/step.yml) is obviously much more, like
```yaml
- num_flaky_test_attempts: 0
```
And more, and more, and more.

You can compare what I have shown you with the official documentation from Github. In this case for [the UI testing step](https://github.com/bitrise-steplib/steps-virtual-device-testing-for-android/blob/master/step.yml). It’s way better than website documentation.

We’re finally at the end. Now, you can use provided a blueprint and run your test suite in minutes via Bitrise. Happy coding!

I hope you like this piece. As you can see I love a fast feedback loop —
if you have any objections, comments, or questions — please drop a comment or DM message.

If you want to reach out, I’m based in Wrocław, Poland.
Here are my [LinkedIn](https://pl.linkedin.com/in/maciejmalak) and [Twitter](https://twitter.com/monkeydevspl).

