<h1 align="center">Resources for Getting Started with Modern Android Development</h1>

[toc]

 After putting together resources for the Mobile Developer Graduate Program that we run at [DVT](http://dvt.co.za/), I realised that the content I referenced could be a great guide for getting started with Modern Android development.

It is worth noting that we have a very **hands-on approach** to training the graduates at DVT. We run workshops as well as practical projects to ensure our graduates have a solid understanding of Android development.

With the vast amount of content available online, when I started Android development I didn’t really know where to look or what I should be aware of when developing an app. Having a concise list like the one in this blog post would have been ***invaluable\*** for me when I started out. I hope you find value in it too.

Here is a list of links, code labs and reference material that would be useful for any developer that wants to get started in the world of Android development.

## Google Source for Android

* [Android Studio Tutorials](https://developer.android.com/studio)
   * [Debug your app](https://developer.android.com/studio/debug)
   * [Test your app](https://developer.android.com/studio/test)
   * [Profile your app performance](https://developer.android.com/studio/profile)
   * [Command line tools](https://developer.android.com/studio/command-line)
* [Jetpack](https://developer.android.com/jetpack)
   * [AndroidX releases](https://developer.android.com/jetpack/androidx/versions)
   * [Guide to app architecture](https://developer.android.com/jetpack/guide)
   * [Join the community](https://developer.android.com/jetpack/community)
* [Kotlin](https://developer.android.com/kotlin)
   * [Develop Android apps with Kotlin](https://developer.android.com/kotlin)
   * [Android’s Kotlin-first approach](https://developer.android.com/kotlin/first)
* [Documentation for app developers](https://developer.android.com/docs)
   * [Developer Guides](https://developer.android.com/guide)
   * [API reference](https://developer.android.com/reference)
   * [Samples](https://developer.android.com/samples)
   * [Design & Quality](https://developer.android.com/design)
* [Git repositories on android](https://android.googlesource.com/?format=HTML)
* [Android Open Source Project](https://android-review.googlesource.com/admin/repos)
* [Android Source](https://source.android.com/)

## General Programming Practices

The following general programming practices are key to getting started with your successful career in development. These practices include:

- [Source Control (Git) ](https://git-scm.com/)— Source control is a tool to manage versions of your code, it is great for writing software collaboratively.
- [Git Workflows](https://www.atlassian.com/git/tutorials/comparing-workflows) — There are many different ways in which software is managed when using source control. Popular methods include: Gitflow workflow, Centralized workflow, Forking workflow etc.
- [Continuous Integration](https://www.thoughtworks.com/continuous-integration) — Continuous integration ensures that your code is building on a server that is not your own machine. Look at using a build server like Jenkins, Buddybuild, Circle CI, Travis etc.
- [Pull Requests](https://www.atlassian.com/blog/git/written-unwritten-guide-pull-requests) — Pull requests are a great way to get very detailed feedback on the code that you have developed.
- [Agile/Scrum methodologies](https://www.scrumalliance.org/why-scrum/scrum-guide) — Most modern software development teams follow Scrum methodologies for working.
- Code Quality Tools – There are many tools that companies use to measure the quality of their code and the health of the codebase. Metrics such as number of lines of test coverage or how much technical debt the code base has, are made visible. Some tools that are frequently used: [Sonar](https://www.sonarqube.org/), [FindBugs](http://findbugs.sourceforge.net/), [Checkstyle](https://github.com/checkstyle/checkstyle) and [Android Lint](https://developer.android.com/studio/write/lint.html).

## Introduction to Android Basics

There are a bunch of websites that provide the basics of getting started with Android development. My recommendation is to follow the official documentation to understand the basics and then look to other resources (such as blogs etc) for additional information as you get into the more technical aspects of your application design (see the sections referenced later in this post).

**Some getting started resources:**

- [Android Application Fundamentals](https://developer.android.com/guide/components/fundamentals.html)
- Some of the main components in Android: [Activities](https://developer.android.com/guide/components/activities/index.html), [Fragments](https://developer.android.com/guide/components/fragments.html), [Services](https://developer.android.com/guide/components/services.html), [Broadcast Receivers](https://developer.android.com/guide/components/broadcasts.html).
- [Android Application Manifest](https://developer.android.com/guide/topics/manifest/manifest-intro.html)
- [Code lab — Build your first Android App](https://codelabs.developers.google.com/codelabs/build-your-first-android-app/index.html)

## Mastering Layouts in Android

There are a lot of different layout types in Android, from FrameLayout to RelativeLayout to ConstraintLayout. Make sure you are comfortable with these commonly used layout types: [FrameLayout](https://developer.android.com/reference/android/widget/FrameLayout.html), [RelativeLayout](https://developer.android.com/guide/topics/ui/layout/relative.html), [LinearLayout](https://developer.android.com/guide/topics/ui/layout/linear.html), [ConstraintLayout](https://developer.android.com/reference/android/support/constraint/ConstraintLayout.html), [CoordinatorLayout](https://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.html).

**Resources:**

- [Supporting different screen sizes](https://developer.android.com/training/multiscreen/screensizes.html)
- [Code lab — ConstraintLayout](https://codelabs.developers.google.com/codelabs/constraint-layout/index.html)
- [Code lab — CoordinatorLayout](https://codelabs.developers.google.com/codelabs/mdc-android/index.html)

## Build System — Working with Gradle

Working with Gradle is probably something that gets overlooked when developing Android apps. Make sure you understand the basics, even better — learn how to write your own gradle tasks!

**Resources:**

- [Gradle Documentation](https://gradle.org/)
- [Configure your Build](https://developer.android.com/studio/build/index.html)

## Networking in Android

Although most of the [Android documentation](https://developer.android.com/training/basics/network-ops/index.html) doesn’t reference Retrofit or OkHttp, these are the most commonly used libraries when it comes to networking in Android. It is also good to be familiar with the different profiling tools available in Android Studio.

**Resources:**

- [Understanding RESTful Services](https://www.tutorialspoint.com/restful/)
- [Retrofit](http://square.github.io/retrofit/) —A type-safe HTTP client for Android and Java
- [OkHttp](http://square.github.io/okhttp/) — An HTTP & HTTP/2 client for Android and Java applications
- [Network Profiler in Android](https://developer.android.com/studio/profile/network-profiler.html) — A tool in Android Studio that allows you to profile your network calls.
- [Charles Proxy](https://www.charlesproxy.com/) — Useful for intercepting network calls whilst testing.

## Architecting your Android Applications

Unfortunately, writing code and getting your app to compile isn’t the end of knowing how to write a maintainable Android app. Large scale Android applications need to follow good architectural designs in order for them to be maintainable and testable. There are many different patterns you can follow when writing an Android application. Patterns such as MVP, MVVM and Clean Architecture are commonly used. Make sure you understand the differences between the patterns as you will encounter many different patterns in the wild.

**Resources:**

- [Android Architecture Components Guide](https://developer.android.com/topic/libraries/architecture/index.html)
- My 3 part series on Android Architecture Components (part [1](https://riggaroo.co.za/android-architecture-components-looking-room-livedata-part-1/), [2](https://riggaroo.co.za/android-architecture-components-looking-viewmodels-part-2/), [3](https://riggaroo.co.za/android-architecture-components-looking-lifecycles-part-3/))
- [Introduction to Android Architecture Components Video](https://www.youtube.com/watch?v=9QrFMsihBAo)
- [Google Sample App Github Repository](https://github.com/googlesamples/android-architecture-components)
- [Code lab — Persistence](https://codelabs.developers.google.com/codelabs/android-persistence/index.html)
- [Code lab — Lifecycle Aware Components](https://codelabs.developers.google.com/codelabs/android-lifecycles/index.html)

## Testing your Android Applications

Once you have gotten the hang of creating Android apps, you will need to think of how to test them. Unit testing and UI testing are very important concepts that you need to make sure you understand. There are plenty of different tools you can use to write UI tests. Most Android developers use Espresso and JUnit to write tests but there are many other tools out there such as Robotium, Calabash, Appium etc. I would recommend using Espresso and JUnit.

**Resources:**

- [Android Testing Support Library](https://developer.android.com/topic/libraries/testing-support-library/index.html)
- [Espresso](https://developer.android.com/training/testing/espresso/basics.html)
- [JUnit](http://junit.org/junit4/)
- [Mockito](http://site.mockito.org/)
- [Code lab — Android Testing](https://codelabs.developers.google.com/codelabs/android-testing/index.html)
- [Code lab — Android Performance Testing](https://codelabs.developers.google.com/codelabs/android-perf-testing/index.html)

## Releasing your Android Apps

Great, you’ve made it this far! Now there are a few concepts you will need to cover in order to release your application:

- [Preparing your app for release](https://developer.android.com/studio/publish/preparing.html)
- [App Signing](https://developer.android.com/studio/publish/app-signing.html)
- [Versioning your App](https://developer.android.com/studio/publish/versioning.html)
- [ProGuard](https://developer.android.com/studio/build/shrink-code.html)

## **Security**

There are plenty of things that should be done in order to secure your application and make sure no one gains access to unauthorised content. Make sure you are using ProGuard (mentioned earlier). Understand what a [Man in the Middle Attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) is. Understand different encryption methods and ways in which you can securely store information inside an Android App including securing your API Tokens, certificate pinning etc.

Resources:

- [Security Tips](https://developer.android.com/training/articles/security-tips.html) on Android.
- [Certificate Pinning](https://square.github.io/okhttp/3.x/okhttp/okhttp3/CertificatePinner.html)
- [SafetyNet API](https://developer.android.com/training/safetynet/index.html)
- [Android Keystore System](https://developer.android.com/training/articles/keystore.html)

## Advanced Android Topics

Once you have covered all the basics of writing Android applications, there are a few advanced topics that you might need to cover in order to contribute to some codebases:

- [**Kotlin**](https://kotlinlang.org/) — Kotlin is the new programming language for Android, developers are actively writing their code in Kotlin. It is worth reading up about Kotlin and running through the Kotlin Koans. There is also a Kotlin [code lab](https://codelabs.developers.google.com/codelabs/build-your-first-android-app-kotlin/index.html) available.
- [**RxJava**](https://github.com/ReactiveX/RxJava) **—** RxJava is a library used for asynchronous, event-based programming. It allows you to compose operations together to do complex tasks (such as combining multiple network calls together) and can be very useful with regards to managing which thread your code is executed on. There is a great [video](https://www.youtube.com/watch?v=htIXKI5gOQU) from Jake Wharton describing how to use RxJava and the benefits of using it.
- [**Dagger**](https://google.github.io/dagger/) (Dependency Injection) — Dependency Injection is a way to manage objects and their dependencies within your application. The concept of DI is not an Android one but is available in many other frameworks as well. DI can make your code more memory efficient and promotes testability. Dagger 2 is the most popular Android DI Framework.
- [**Material Design**](https://material.io/) **—** Most Android Apps follow Google’s Material Design Guidelines. The guidelines are a way of designing your apps in a standard way which users are accustomed to.
- [**Support Libraries in Android**](https://developer.android.com/topic/libraries/support-library/index.html) **—** The support libraries in Android are very important to ensure that your app looks and behaves consistently across multiple versions of Android. There are a few different libraries that have different purposes. The linked article describes the reasoning behind the libraries.
- **Memory Leaks —** In Android, it is quite easy to create memory leaks. This can lead to erroneous behaviour in your application (random crashes). Read about memory leaks [here](https://riggaroo.co.za/fixing-memory-leaks-in-android-outofmemoryerror/). A lot of developers use [LeakCanary](https://github.com/square/leakcanary) within their apps to ensure that there aren’t any memory leaks present.

## Keeping up to date with Modern Android Development

There are a multitude of ways to keep up to date with the latest changes and developments in the Android Community. Some of the ways I find useful:

- Subscribe to the [Android Weekly](http://androidweekly.net/) newsletter
- Follow [/r/androiddev](https://www.reddit.com/r/androiddev/) on Reddit
- Follow [Android Google Developer Experts](https://developers.google.com/experts/all/technology/android) on Twitter
- Follow [Android Studio](https://twitter.com/androidstudio), [Android Dev](https://twitter.com/AndroidDev) on Twitter
- Listen to Android Development Podcasts ([Fragmented](http://fragmentedpodcast.com/), [Android Developers Backstage](http://androidbackstage.blogspot.co.za/), [The Context](https://player.fm/series/the-context-androiddev), [Android Snacks](https://player.fm/series/android-snacks))



## source

https://riggaroo.dev/resources-getting-started-android-development/

