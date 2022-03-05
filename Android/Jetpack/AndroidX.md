<h1 align="center">AndroidX</h1>

[toc]

## 官方地址

* [Jetpack](https://developer.android.com/jetpack)
* [Developer Guides](https://developer.android.com/guide)
* [AndroidX Version](https://developer.android.com/jetpack/androidx/versions)
* [Google Maven repositery](https://maven.google.com/web/index.html)

## 代码下载

### 代码地址：

* [androidx/androidx: Development environment for Android Jetpack extension libraries under the androidx namespace. Synchronized with Android Jetpack's primary development branch on AOSP. (github.com)](https://github.com/androidx/androidx)

* [refs/heads/androidx-main - platform/frameworks/support - Git at Google (googlesource.com)](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-main)



### Checking Out the Code

**NOTE: You will need to use Linux or Mac OS. Building under Windows is not currently supported.**

1. Install `repo` (Repo is a tool that makes it easier to work with Git in the context of Android. For more information about Repo, see the [Repo Command Reference](https://source.android.com/setup/develop/repo))

```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

1. Configure Git with your real name and email address.

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

1. Create a directory for your checkout (it can be any name)

```
mkdir androidx-main
cd androidx-main
```

1. Use `repo` command to initialize the repository.

```
repo init -u https://android.googlesource.com/platform/manifest -b androidx-main --partial-clone --clone-filter=blob:limit=10M
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b androidx-main --partial-clone --clone-filter=blob:limit=10M
```

注意：可以将 https://android.googlesource.com/platform/manifest 换成：https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest

1. Now your repository is set to pull only what you need for building and running AndroidX libraries. Download the code (and grab a coffee while we pull down the files):

```
repo sync -j8 -c
```

You will use this command to sync your checkout in the future - it’s similar to `git fetch`

### Using Android Studio

To open the project with the specific version of Android Studio recommended for developing:

```
cd path/to/checkout/frameworks/support/
// run androidx
ANDROIDX_PROJECTS=MAIN ./gradlew studio
// run compose
ANDROIDX_PROJECTS=COMPOSE ./gradlew studio
```

and accept the license agreement when prompted. Now you're ready to edit, run, and test!

You can also the following sets of projects: `ALL`, `MAIN`, `COMPOSE`, or `FLAN`

If you get “Unregistered VCS root detected” click “Add root” to enable git integration for Android Studio.

If you see any warnings (red underlines) run `Build > Clean Project`.

### Builds

#### Full Build (Optional)

You can do most of your work from Android Studio, however you can also build the full AndroidX library from command line:

```
cd path/to/checkout/frameworks/support/
./gradlew createArchive
```

#### Testing modified AndroidX Libraries to in your App

You can build maven artifacts locally, and test them directly in your app:

```
./gradlew createArchive
```

And put the following at the top of your ‘repositories’ property in your **project** `build.gradle` file:

```
maven { url '/path/to/checkout/out/androidx/build/support_repo/' }
```

## Android Jetpack Components

|Foundation|Architecture|Behavior|UI|
|:---|:---|:---|:---|
|Android KTX|Data Binding|CameraX|Animation & transitions|
|AppCompat|Lifecycles|Media & playback|Emoji|
|Car|LiveData|Notifications|Fragment|
|Benchmark|Navigation|Permissions|Layout|
|Multidex|Paging|Preferences|Palette|
|Security|Room|Sharing|ViewPager2|
|Test|ViewModel|Slices|WebView|
|Tv|WorkManager|||
|Wear OS by Google||||



## AndroidX releases

> Androidx的各个library版本

https://developer.android.com/jetpack/androidx/versions



## Artifact Mappings

> Android support vs Androidx

https://developer.android.com/jetpack/androidx/migrate/artifact-mappings



## Jetpack源码解析以及优秀博客

### ViewPager2

* [ViewPager初始化源码解析](https://www.jianshu.com/p/76ca6f7d6b1a)
* [ViewPager滑动原理解析](https://www.jianshu.com/p/5d3ae197325e)
* [ViewPager方法改造实现无限循环](https://www.jianshu.com/p/ed9cedcaaaba)



### RecyclerView

* [GravitySnapHelper](https://github.com/rubensousa/GravitySnapHelper)
* [RecyclerViewItemAnimators](https://github.com/gabrielemariotti/RecyclerViewItemAnimators)
* [BaseRecyclerViewAdapterHelper](https://github.com/CymChad/BaseRecyclerViewAdapterHelper)
* [ViewPagerLayoutManager](https://github.com/leochuan/ViewPagerLayoutManager)

   

### Hilt

* [Dependency Injection on Android with Hilt](https://medium.com/androiddevelopers/dependency-injection-on-android-with-hilt-67b6031e62d)
* [Dependency injection with Hilt](https://developer.android.com/training/dependency-injection/hilt-android#setup)
* [All about Hilt](https://proandroiddev.com/all-about-hilt-a-dependency-injection-framework-869b9c2bcb09)







