<h1 align="center"> Java Feature</h1>

[toc]

## 一、[JDK Source](https://github.com/zxiaofan/JDK)
Jdk source includ 1.6/1.7/1.8/1.9/10/11/12/13/14.

## 二、[java-almanac](https://github.com/marchof/java-almanac)
Collection of information about the history of Java primarily from a technical point of view.

Websit:  [https://javaalmanac.io/](https://javaalmanac.io/)


## 三、[agp-java-support](https://github.com/JakeWharton/agp-java-support)
Tracking your ability to use new Java language features and APIs in an Android app

This repository tracks your ability to use new Java language features in an Android app.

| AGP / Java    | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 |
|---------------|---|---|---|----|----|----|----|----|
| 3.4.2         | ✅ | ✅ | ❌ | ❌  | ❌  | ❌  | ❌  | ❌  |
| 3.5.3         | ✅ | ✅ | ❌ | ❌  | ❌  | ❌  | ❌  | ❌  |
| 3.6.1         | ✅ | ✅ | ❌ | ❌  | ❌  | ❌  | ❌  | ❌  |
| 4.0.0-beta03  | ✅ | ✅ | ❌ | ❌  | ❌  | ❌  | ❌  | ❌  |
| 4.1.0-alpha03 | ✅ | ✅ | ❌ | ❌  | ❌  | ❌  | ❌  | ❌  |

Depressing! While there's no problem using JDK 14 to compile your app with source/target level 8,
once you change the source/target level to 9 or higher the way `javac` compiles your code changes.
Prior to Java 9, Java's APIs were exposed through something called `rt.jar` which was put on the
"bootclasspath" of `javac`. Android used the same mechanism with its `android.jar`. When set to 9
or higher, `javac` uses the new module system which requires an entirely new mechanism of exposing
the Android APIs.

So yeah. Not great.

Here's D8/R8's support of Java versions though:

| AGP / Java    | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 |
|---------------|---|---|---|----|----|----|----|----|
| 3.4.2         | ✅ | ✅ | ✅ | ✅  | ✅  | ✅  | ❌  | ❌  |
| 3.5.3         | ✅ | ✅ | ✅ | ✅  | ✅  | ✅  | ✅  | ❌  |
| 3.6.1         | ✅ | ✅ | ✅ | ✅  | ✅  | ❌  | ❌  | ❌  |
| 4.0.0-beta03  | ✅ | ✅ | ✅ | ✅  | ✅  | ❌  | ❌  | ❌  |
| 4.1.0-alpha03 | ✅ | ✅ | ✅ | ✅  | ✅  | ❌  | ❌  | ❌  |


Notes:
 - Just because a Java language version is supported does not mean all tools will handle it
   properly. For example, annotation processors and bytecode transformers may need updated.
 - Lint is not run on the test projects because is consumes an inordinate amount of memory and seems
   to have JVM metaspace leaks making automated testing difficult. Anecdotally, it seems to work
   fine on the latest stable, beta, and alpha for the working Java versions.
 - All projects are built with Gradle 6.3 RC4 which is required for running on Java 14. Older Gradle
   versions can probably be used with lower Java versions but are not tracked here.


### Java 7

------

 * [Binary literals](projects/java7/src/main/java/com/jakewharton/javaversions/java7/BinaryLiterals.java)
 * [Diamond operator](projects/java7/src/main/java/com/jakewharton/javaversions/java7/DiamondOperator.java)
 * [Multi-catch](projects/java7/src/main/java/com/jakewharton/javaversions/java7/Mutlicatch.java)
 * [Save varargs](projects/java7/src/main/java/com/jakewharton/javaversions/java7/SafeVarargsOnMethod.java)
 * [Switch on string](projects/java7/src/main/java/com/jakewharton/javaversions/java7/SwitchOnString.java)
 * [Try-with-resources](projects/java7/src/main/java/com/jakewharton/javaversions/java7/TryWithResources.java)
 * [Underscore literals](projects/java7/src/main/java/com/jakewharton/javaversions/java7/UnderscoreLiterals.java)


### Java 8

------

 * [Default methods in interfaces](projects/java8/src/main/java/com/jakewharton/javaversions/java8/InterfaceDefaultMethod.java)
 * [Static methods in interfaces](projects/java8/src/main/java/com/jakewharton/javaversions/java8/InterfaceStaticMethod.java)
 * [Lambdas](projects/java8/src/main/java/com/jakewharton/javaversions/java8/Lambda.java)
 * [Method references](projects/java8/src/main/java/com/jakewharton/javaversions/java8/MethodReference.java)
 * [Repeated annotations](projects/java8/src/main/java/com/jakewharton/javaversions/java8/RepeatedAnnotation.java)
 * [Type annotations](projects/java8/src/main/java/com/jakewharton/javaversions/java8/TypeAnnotation.java)


### Java 9

------

❌ AGP usage blocked by https://issuetracker.google.com/issues/139013660.

 * [Anonymous diamond operator](projects/java9/src/main/java/com/jakewharton/javaversions/java9/AnonymousDiamond.java)
 * [Try-with-resources on effectively-final variables](projects/java9/src/main/java/com/jakewharton/javaversions/java9/EffectivelyFinalTryWithResources.java)
 * [Private methods in interfaces](projects/java9/src/main/java/com/jakewharton/javaversions/java9/PrivateInterfaceMethods.java)
 * [Save varargs on private methods](projects/java9/src/main/java/com/jakewharton/javaversions/java9/SafeVarargsOnPrivate.java)


### Java 10

-------

❌ AGP usage blocked by https://issuetracker.google.com/issues/139013660.

 * [Local variable type inference](projects/java10/src/main/java/com/jakewharton/javaversions/java10/LocalVariableTypeInference.java)


### Java 11

-------

❌ AGP usage blocked by https://issuetracker.google.com/issues/139013660.

 * [Lambda parameter type inference](projects/java11/src/main/java/com/jakewharton/javaversions/java11/LambdaParameterTypeInference.java)


### Java 12

-------

❌ AGP usage blocked by https://issuetracker.google.com/issues/139013660.

❌ D8 usage blocked by https://issuetracker.google.com/issues/141587937.

 * (preview) [Switch expressions](projects/java12-with-preview/src/main/java/com/jakewharton/javaversions/java12/SwitchExpression.java)


### Java 13

-------

❌ AGP usage blocked by https://issuetracker.google.com/issues/139013660.

❌ D8 usage blocked by https://issuetracker.google.com/issues/141587937.

 * (preview) [Switch expressions](projects/java13-with-preview/src/main/java/com/jakewharton/javaversions/java13/SwitchExpression.java)
 * (preview) [Text blocks](projects/java13-with-preview/src/main/java/com/jakewharton/javaversions/java13/TextBlocks.java)


### Java 14

-------

❌ AGP usage blocked by https://issuetracker.google.com/issues/139013660.

❌ D8 usage blocked by https://issuetracker.google.com/issues/141587937.

 * [Switch expressions](projects/java14/src/main/java/com/jakewharton/javaversions/java14/SwitchExpression.java)
 * (preview) [Instanceof pattern matching](projects/java14-with-preview/src/main/java/com/jakewharton/javaversions/java14/InstanceOfPatternMatching.java)
 * (preview) [Text blocks](projects/java14-with-preview/src/main/java/com/jakewharton/javaversions/java14/TextBlocks.java)
 * ❌ (preview) Records (requires runtime API support)



## 四、[java-diff-utils](https://github.com/java-diff-utils/java-diff-utils)

Diff Utils library is an OpenSource library for performing the comparison / diff operations between texts or some kind of data: computing diffs, applying patches, generating unified diffs or parsing them, generating diff output for easy future displaying (like side-by-side view) and so on. [https://java-diff-utils.github.io/jav…](https://java-diff-utils.github.io/java-diff-utils/)

## 五、**[ jdk-api-diff](https://github.com/AdoptOpenJDK/jdk-api-diff)**

Creates a report of all API changes two JDK versions