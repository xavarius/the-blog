---
layout: default
title: "Be effective with Bitrise CI for Android — lessons I learned the hard way."
date: 2020-05-05
---

### **Be effective with Bitrise CI for Android — the lessons I learned the hard way.**
![Featured image](/assets/1*TQOKYOUKm5mDiYlF7cKO1A.png)

I won’t elaborate here on how important and crucial for any software development-oriented team the [continuous integration](https://www.thoughtworks.com/continuous-integration) (CI) practise is.  
I’m pretty sure we can all agree on how CI tools support our day to day effectiveness. How they **might** save dozens of hours spent on non-essential tasks. Yet, it’s common to present CI tools as a hassle; slow, bulky,  
and unreliable pipelines bloated with chaotic events instead of fast, maintainable feedback loop configured to support both product quality  
and team flexibility.

As the title implies, our CI process was far from optimal. We learned what “slow and chaotic” means the hard way. Below, you will find an overview  
of each issue that slowed us down, with full explanation of what the solution was (including code and external links), as well as honest **results** measured by minutes.

In this article, you will find discussion surrounding architecture, *flavour agnostic* unit testing, Gradle usage as well as keeping your logs and artefacts deployment in order. Additionally, at the end of the article, several tips  
and tricks beyond optimisation will be included.  
It’s not a step-by-step tutorial. We gathered results that work for us,  
and **you** have to **think them through.** If those solutions make sense to you then, and only then, apply them to your environment.

### The landscape
In order to fully understand **why** we provided a particular optimisation  
it is crucial to understand how our landscape looked at the time.

There is [git flow approach](https://nvie.com/posts/a-successful-git-branching-model/) in place, which usually means multiple *feature* branches exist at the same time in a remote repository. There is at least one *pull request* per story. Each pull request needs to go through an integration *process* meaning the newest commit in a pull request triggers a fresh CI build. That’s being done in order to ensure the newest change won’t introduce any flaws. Yep, automation and unit test suites test each *software incrementation*. Software Engineers in Test (SET) writes automation tests  
as “a part of“ the feature in some cases.

We are supporting multiple modules as a part of our architecture.  
Let’s assume it is a [clean-ish architecture](https://five.agency/android-architecture-part-2-clean-architecture/) with *domain*, *data* and *app* layers packed into separate modules. Each of the modules has its own unit tests suite — between dozens to few hundreds of them per module. We have to support multiple *flavours* and they differ greatly. Each flavour has  
a separate set of automation and unit test, although most of them are shared.

When it comes to infrastructure, there is a separate Bitrise workflow for every build type. Also, a separate one for each of: *feature* development,  
*automation* efforts, *release* (*tags*) activities and after merging feature  
to the develop. Seeing how many distinct configs we have, there is a need  
to run multiple builds every day. We can’t and won’t have “infinite” amount  
of concurrent jobs, so time devoted to each build is very important to us.  
It’s also important because we **value sh*t done the right way**.

The basic measurement that **will prove effectiveness here is build time** — both *entire* build time or a particular step time (such as unit tests step or deploy step).

### Improvements
#### Unit testing
The most commonly used feedback loop is unit tests suite, in particular  
if you’re supporting multiple flavours for Android app and you want to be sure that none of the changes would break any of the flavours.  
Unit tests are supposed to be a fast and reliable feedback loop, which can be automated at the CI level. So, we used docs and tutorials to set them up for  
all of the flavours. After few changes to CI, we ended up with 30 minutes long unit test step for 3 flavours.  
Yes, you read it properly: 30 minutes for 3 flavours.

Ok, let’s fix that.

After a little bit of research it occurred to us that we used two separate steps for unit tests. [Android unit test step for Bitrise](https://devcenter.bitrise.io/testing/android-run-a-unit-test/) was running *app* module unit tests. [Gradle Unit Test](https://www.bitrise.io/integrations/steps/gradle-unit-test) step was just running *.gradlew test* task.

![Total time for each step in minutes.](/assets/1*ZDxhzjlV0QFs2JDvktxfuA.png)
_Total time for each step in minutes._

What’s wrong with gradle unit test step in our case? According to the Gradle documentation:

![Gradle CLI documentation screenshot](/assets/1*6z7dHF2MnY1nAVjGlLGntQ.png)
_Source: [https://docs.gradle.org/current/userguide/command_line_interface.html](https://docs.gradle.org/current/userguide/command_line_interface.html)_

In simple terms, *./gradlew test* [triggers **dozens** of different *test* *tasks*](https://medium.com/android-testing-daily/running-your-tests-on-the-command-line-with-gradle-bcba78244487) from every module. In our case, it triggered both *debug* and *release* related tests  
for every *subproject (*module*)*. That’s too much redundancy; consider the final result of *./gradlew test* command:

*(amount of flavours) x (amount of supported envs) x (amount of modules)*

But the amount of tasks triggered is not all we can improve here.  
I already mentioned we have several modules. Since it’s *cleanish architecture,* it consists of *app*, *domain*, *data* and *api* modules.  
It’s easy to see that some of those modules are *flavour agnostic — *domain, data and api layers can and should be treated as libraries. Those are *external dependencies* that *could* be used via any JVM compatible code.  
Do we need to run those tests separately for each flavour?  
Of course we don’t! Where does it lead us?

#### **Flavour agnostic unit tests**
Split *flavour dependent* and *flavour agnostic* unit tests. Gain greater control over how your application is tested. Use [Gradle Unit Test step](https://www.bitrise.io/integrations/steps/gradle-unit-test) in your [Bitrise.yml](https://devcenter.bitrise.io/bitrise-cli/basics-of-bitrise-yml/) to run targeted *flavour agnostic* unit tests, like this:

{% gist xavarius/6132a1ed16bdd944d3cbe0e9cf4ca2b3 %}
_My gist lets you configure module dependent unit testing. Source: [https://gist.github.com/xavarius/6132a1ed16bdd944d3cbe0e9cf4ca2b3](https://gist.github.com/xavarius/6132a1ed16bdd944d3cbe0e9cf4ca2b3)_

Using *unit_test_task* attribute enables you to configure a particular task to be run. Basically, any gradle task. You can obviously chain gradle commands, but I want **granularity** here. Additionally, the usage of *title* attribute keeps build logs in order and enables you to track each step separately.

![The result of applying the above mentioned recommendations.](/assets/1*SQSkq3ZylChkaKHJxCGHTw.png)
_The result of applying the above mentioned recommendations. Cleaning up resources gave us unit tests result in seconds instead of minutes._

#### Flavour dependent unit tests
The second recommendation relates to [Android unit test step for Bitrise](https://devcenter.bitrise.io/testing/android-run-a-unit-test/)  
and how *flavour dependent* unit tests are managed. In most cases,  
I would recommend you to run only what you need. But I came to conclusion that ‘run only what you need’ could be counterintuitive in our case.

It’s **really easy** to break one of the flavours by introducing changes to only one of them. That’s why we ended up with running unit tests for every flavour in every build. In addition, the above mentioned set of *flavour agnostic* tests is triggered. What does it mean when it comes to Bitrise CI setup?

{% gist xavarius/d14bb74261ac9cc40f134b20e66f5542 %}
_Targeted unit tests per flavour. Source: [https://gist.github.com/xavarius/d14bb74261ac9cc40f134b20e66f5542](https://gist.github.com/xavarius/d14bb74261ac9cc40f134b20e66f5542)_

The above snippet runs unit tests for the *app* module for a particular flavour injected as an [*environment variable*](https://devcenter.bitrise.io/builds/env-vars-secret-env-vars/) and a particular build variant.  
So, if CI builds only one flavour at time, this snippet is supposed to be triggered three times, once for each flavour. If all of the flavours are built simultaneously, then each flavour should run its own unit tests in order to avoid redundancy and save a few minutes from build. Notice that, before *_UnitTestsPerFlavour step, UnitTests_Flavour_Agnostic_Modules* step is triggered. It runs flavour agnostic tests, so *domain*, *data and feature* modules unit tests. Either way, all unit tests are always validated.

Alternatively to the above setup, you can use the following setup to hardcode which flavour’s unit tests should be run:

{% gist xavarius/4699179027cce14007257d7b17984d55 %}

That way we’re all covered, no matter if we’re building all the flavours, or just one. Remember, the lesson here is *flavour dependent and agnostic* unit tests should be triggered **once.** There is no redundancy, but there is full coverage. Every software increment is **safe**.

#### Results
We started with around **30 minutes** per build.

![Total time for unit testing then.](/assets/1*ZDxhzjlV0QFs2JDvktxfuA.png)
_Total time for unit testing then._

And finished up with the below results when running one flavour.  
**Down to ~5/6 minutes per build.**

![Total time for unit testing now.](/assets/1*-ah-XIlWh18Ac0-6z7JEXA.jpeg)
_Total time for unit testing now._

And also **down to ~3 minutes per build** when running all the flavours  
at once, which means each flavour is responsible for its own unit tests finally. Yes, that’s a separate config in order to optimise build time even further.

![Total time when running all of the flavours.](/assets/1*eFTwp0OYl-5lIMLpKgx3GA.png)
_Total time when running all of the flavours. Build time for those unit tests per one flavour._

### Artefacts Deployment
Review your *Deploy to Bitrise.io* step. According to the documentation [[1]](https://devcenter.bitrise.io/testing/test-reports/) [[2]](https://www.bitrise.io/integrations/steps/deploy-to-bitrise-io) [[3]](https://devcenter.bitrise.io/testing/device-testing-for-android/) for the following steps test reports are deployed automatically:

- Xcode Test for iOS
- Android Unit Test
- iOS Device Testing
- Virtual Device Testing for Android

As noted in the documentation, by default Android unit and UI tests are deployed to Bitrise directory and are provided via the *Test reports* tab. They are easily accessible — but the question is — are they really necessary?

We have robust unit tests. They fail rarely in CI because the entire team writes and runs them frequently. On the other hand, it’s easy to check Bitrise for which logs failed.

#### Deciding what to deploy
We already changed from [Android unit test step for Bitrise](https://devcenter.bitrise.io/testing/android-run-a-unit-test/) to [Gradle Unit Test](https://www.bitrise.io/integrations/steps/gradle-unit-test) step which does not deploy unit tests reports automatically. And we want it that way. What about the rest of the artefacts? For *automation* builds we’ve decided not to deploy any APKs. They are not needed.

We also already know that [Virtual Device Testing for Android](https://www.bitrise.io/integrations/steps/virtual-device-testing-for-android) step deploys UI tests results into the *Test Reports* directory. We decided that for all of the builds we are going to move or remove *Deploy to Bitrise.io* step completely as an experiment.  
Also, *Deploy to Bitrise.io* step is always triggered before unit tests but after APK creation. That way, only application (*uatRelease* APK for example) and the UI tests report are deployed.

Initially *deploy to Bitrise.io* **step took from 2.1 to 3.2 minutes**.

![Initially 3.2 min was total time per this step.](/assets/1*rPm9k8yro36t8KsQr9f1oQ.png)
_Initially 3.2 min was total time per this step._

**After the changes it’s 0 minutes for some builds. It is ~8 seconds for most of them.**

![That’s how quick it could be!](/assets/1*QkK3s1i0SLgx5Zo1NBNw0A.png)
_That’s how quick it could be!_

![Oh yeah!](/assets/1*kFe06gXR1oM1b5rbmSR9BQ.jpeg)
_Oh yeah! Source: [https://knowyourmeme.com/photos/988454-we-did-it-reddit](https://knowyourmeme.com/photos/988454-we-did-it-reddit)_

### Automation workflow
One of the low hanging fruits was to change what is being done  
as a part of a particular workflow, since they all have different goals.  
As I mentioned, we have *feature*, *automation*, *develop* and *release* workflow.  
In our case, initially, all of the mentioned workflows had basically  
the same setup. Why is this wrong? Because, as we said, workflows simply have different **responsibilities**.

#### **Understanding the differences in workflows**
I have already mentioned the *automation* workflow. It’s because it is special compared to other workflows. The only responsibility automation workflow has is to support Software Engineers in Test in writing and securing automation test suite. That simple conclusion means we can trim  
several steps from it; in our case, APK and other artefacts creation  
and deployment. We were also able to get rid of custom scripts we had there for the release app or “runtime” resources optimisations steps and beyond.

#### Results
By doing this, the *automation* build is a fast feedback loop for the SETs.  
**It takes around 10 minutes less than other builds.**  
I believe it’s a huge win for the SETs team.

### Investigating tools configuration
Here is a quick and simple story as an example. Our builds produced *uatDebug* and *uatRelease* APKs. UAT stands for ‘[user acceptance testing](https://en.wikipedia.org/wiki/Acceptance_testing#User_acceptance_testing)’ and it’s also a name of one of our environment s— environment with almost *production* setup but more over *development* data — and simply used for testing purposes. So, producing those two build sounds about right, doesn’t it? I started asking questions anyway. We were sure we need *uatRelease* for testing purposes. It makes sense since testing production ready app (*release*) using development data (*uat*) is one of the best practices. But why do we need *uatDebug* then?

#### **Trimming unused resources**
The sole reason was a misconfiguration of the [Charles proxy](https://www.charlesproxy.com/) tool, which led testers to not being able to use proxy tools while testing *uatRelease* build variant. Famous [*network_security_config*](https://developer.android.com/training/articles/security-config) file had been added to the project but it wasn’t working, since the build variant has to be *debuggable.* The quick fix was to add *android:debuggable* attribute to all *uat* builds. And since we’re not testing *uat* builds using any public channels — it’s secure enough.

#### Results
A simple configuration fix to the existing toolset **brought an 8 minutes time reduction to each build and fixed SETs headache.**

### All numbers together
In summary

- Unit tests time **down from 30 minutes to 3~6 minutes.** Depends on build type.
- Automation build cut off by **another 10 minutes** through removing a few unnecessary steps.
- Artefacts deployment reduced **from 2.1~3.2 minutes into 8 seconds!**
- Fix to Charles configs gave us another **8 minutes** — due *uatDebug* build removal.

We were able to shorten builds by between **48 minutes and 34 minutes  
per each build**. That was a huge win and relief as you can imagine!

We obviously made some rookie mistakes. But the most important part  
is to learn from them. We were able to adapt quickly and we’re providing other small improvements since then. It can’t happen on a daily basis because we also need to deliver *business value* to our clients — but with an appropriate plan in place, I’m sure you can do even more.

### Tips and tricks beyond optimisations
Bitrise and its plugins’ documentation is quite limited. You will need to deep dive into the plugins code if you want to understand the platform fully. Plugins code is mostly open source — you can find links inside plugin documentation. In particular, review the [*main.go*](https://github.com/bitrise-steplib/steps-gradle-unit-test/blob/master/main.go) file if you’re looking for attributes and parameters which could customise the build.

Use [Bitrise CLI](https://app.bitrise.io/cli) in your terminal in order to test configuration locally. It will save you a lot of time.

Have as granular CI steps as possible. Use *title* attribute extensively. Greater readability — greater control over time. Solid foundations are the first step for future optimisation.

Do what we haven’t done yet — introduce tools to measure build metrics automatically.

Leverage version control since Bitrise is *similar* to [infrastructure as a code](https://en.wikipedia.org/wiki/Infrastructure_as_code).

That’s a separate story but in Tigerspike, we **optimised APK size by 13%** during our internal hackathon day. You should be aware of best practises for Android app configuration. Get rid or optimise resources, configuration and APK size. These kinds of things are also impacting your build time: git pull, compilation and build time, tests, deploy time — these are some of many examples.

Listen. Observe. Experiment. Formulate a plan and adopt only what’s needed for your team. Good luck!

![Donald Duck says Thank you!](/assets/1*BZQiSFYjEQF1yKdcpJuWXA.png)
_Thanks! Source: [http://123emoji.com/donald-duck-stickers-2-9606/](http://123emoji.com/donald-duck-stickers-2-9606/)_

I hope you like this piece. As you can see I love a fast feedback loop —  
if you have any objections, comments or questions — please drop a comment or DM message.

The article showcases what we have done for one of our projects in [Tigerspike](https://tigerspike.com/). We’re [hiring](https://tigerspike.com/join-us/) — please mention my name! ;)

If you want to reach me out, I’m based in Wrocław, Poland.  
I’m also visiting London from time to time.  
Here is my [LinkedIn](https://pl.linkedin.com/in/maciejmalak) and [Twitter](https://twitter.com/monkeydevspl).
