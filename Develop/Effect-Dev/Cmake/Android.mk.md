# Android.mk

## Secrets of Android.mk
* [Intro to Android.mk](#Intro to Android.mk)
* [Simple example](#Simple example)
* [NDK Usage](#NDK Usage)
* [Defining Modules](#Defining Modules)
* [Simple APK](#Simple APK)
* [APK Dependent on static .jar file](#APK Dependent on static .jar file)
* [APK signed with the platform key](#APK signed with the platform key)
* [APK that signed with vendor key](#APK that signed with vendor key)
* [Prebuilt APK](#Prebuilt APK)
* [Adding a Static Java Library](#Adding a Static Java Library)
* [Android.mk variables](#Android.mk variables)


## Introduction to Android.mk

This document describes the syntax of Android.mk build file written to describe your C and C++ source files to the Android
NDK. To understand what follows, it is assumed that you have read the docs/OVERVIEW.html file that explains their role and
usage.

An Android.mk file is written to describe your sources to the build system. More specifically:

- The file is really a tiny GNU Makefile fragment that will be parsed one or more times by the build system. As such, you
  should try to minimize the variables you declare there and do not assume that anything is not defined during parsing.

- The file syntax is designed to allow you to group your sources into 'modules'. A module is one of the following:

    - a static library
    - a shared library

  Only shared libraries will be installed/copied to your application package. Static libraries can be used to generate
  shared libraries though.

  You can define one or more modules in each Android.mk file, and you can use the same source file in several modules.

- The build system handles many details for you. For example, you don't need to list header files or explicit dependencies between
  generated files in your Android.mk. The NDK build system will compute these automatically for you.

  This also means that, when updating to newer releases of the NDK, you should be able to benefit from new toolchain/platform support
  without having to touch your Android.mk files.

Note that the syntax is *very* close to the one used in Android.mk files distributed with the full open-source Android platform sources. While
the build system implementation that uses them is different, this is an intentional design decision made to allow reuse of 'external' libraries'
source code easier for application developers.


## Simple example

Before describing the syntax in details, let's consider the simple "hello JNI" example, i.e. the files under:

    apps/hello-jni/project

Here, we can see:

  - The 'src' directory containing the Java sources for the sample Android project.

  - The 'jni' directory containing the native source for the sample, i.e. 'jni/hello-jni.c'

    This source file implements a simple shared library that implements a native method that returns a string to the
    VM application.

  - The 'jni/Android.mk' file that describes the shared library to the NDK build system. Its content is:


   LOCAL_PATH := $(call my-dir)

   include $(CLEAR_VARS)

   LOCAL_MODULE    := hello-jni
   LOCAL_SRC_FILES := hello-jni.c

   include $(BUILD_SHARED_LIBRARY)



Now, let's explain these lines:

  LOCAL_PATH := $(call my-dir)

An Android.mk file must begin with the definition of the LOCAL_PATH variable.
It is used to locate source files in the development tree. In this example, the macro function 'my-dir', provided by the build system, is used to return
the path of the current directory (i.e. the directory containing the Android.mk file itself).

  include $(CLEAR_VARS)

The CLEAR_VARS variable is provided by the build system and points to a special GNU Makefile that will clear many LOCAL_XXX variables for you
(e.g. LOCAL_MODULE, LOCAL_SRC_FILES, LOCAL_STATIC_LIBRARIES, etc...), with the exception of LOCAL_PATH. This is needed because all build
control files are parsed in a single GNU Make execution context where all variables are global.

  LOCAL_MODULE := hello-jni

The LOCAL_MODULE variable must be defined to identify each module you describe in your Android.mk. The name must be *unique* and not contain
any spaces. Note that the build system will automatically add proper prefix and suffix to the corresponding generated file. In other words,
a shared library module named 'foo' will generate 'libfoo.so'.


IMPORTANT NOTE:
If you name your module 'libfoo', the build system will not add another 'lib' prefix and will generate libfoo.so as well.
This is to support Android.mk files that originate from the Android platform sources, would you need to use these.


  LOCAL_SRC_FILES := hello-jni.c

The LOCAL_SRC_FILES variables must contain a list of C and/or C++ source files that will be built and assembled into a module. Note that you should
not list header and included files here, because the build system will compute dependencies automatically for you; just list the source files
that will be passed directly to a compiler, and you should be good.

Note that the default extension for C++ source files is '.cpp'. It is however possible to specify a different one by defining the variable
LOCAL_CPP_EXTENSION. Don't forget the initial dot (i.e. '.cxx' will work, but not 'cxx').

  include $(BUILD_SHARED_LIBRARY)

The BUILD_SHARED_LIBRARY is a variable provided by the build system that points to a GNU Makefile script that is in charge of collecting all the
information you defined in LOCAL_XXX variables since the latest 'include $(CLEAR_VARS)' and determine what to build, and how to do it
exactly. There is also BUILD_STATIC_LIBRARY to generate a static library.

There are more complex examples in the samples directories, with commented Android.mk files that you can look at.


Reference:

This is the list of variables you should either rely on or define in an Android.mk. You can define other variables for your own usage, but
the NDK build system reserves the following variable names:

- names that begin with LOCAL_  (e.g. LOCAL_MODULE)
- names that begin with PRIVATE_, NDK_ or APP_  (used internally)
- lower-case names (used internally, e.g. 'my-dir')

If you need to define your own convenience variables in an Android.mk file, we recommend using the MY_ prefix, for a trivial example:


    MY_SOURCES := foo.c
    ifneq ($(MY_CONFIG_BAR),)
      MY_SOURCES += bar.c
    endif

    LOCAL_SRC_FILES += $(MY_SOURCES)



So, here we go:


NDK-provided variables:

These GNU Make variables are defined by the build system before your Android.mk file is parsed. Note that under certain circumstances
the NDK might parse your Android.mk several times, each with different definition for some of these variables.

CLEAR_VARS
    Points to a build script that undefines nearly all LOCAL_XXX variables listed in the "Module-description" section below. You must include
    the script before starting a new module, e.g.:

      include $(CLEAR_VARS)

BUILD_SHARED_LIBRARY
    Points to a build script that collects all the information about the module you provided in LOCAL_XXX variables and determines how to build
    a target shared library from the sources you listed. Note that you must have LOCAL_MODULE and LOCAL_SRC_FILES defined, at a minimum before
    including this file. Example usage:

      include $(BUILD_SHARED_LIBRARY)

    note that this will generate a file named lib$(LOCAL_MODULE).so

BUILD_STATIC_LIBRARY
    A variant of BUILD_SHARED_LIBRARY that is used to build a target static library instead. Static libraries are not copied into your
    project/packages but can be used to build shared libraries (see LOCAL_STATIC_LIBRARIES and LOCAL_WHOLE_STATIC_LIBRARIES described below).
    Example usage:

      include $(BUILD_STATIC_LIBRARY)

    Note that this will generate a file named lib$(LOCAL_MODULE).a

PREBUILT_SHARED_LIBRARY
    Points to a build script used to specify a prebuilt shared library.
    Unlike BUILD_SHARED_LIBRARY and BUILD_STATIC_LIBRARY, the value of LOCAL_SRC_FILES must be a single path to a prebuilt shared
    library (e.g. foo/libfoo.so), instead of a source file.

    You can reference the prebuilt library in another module using the LOCAL_PREBUILTS variable (see docs/PREBUILTS.html for more
    information).

PREBUILT_STATIC_LIBRARY
    This is the same as PREBUILT_SHARED_LIBRARY, but for a static library file instead. See docs/PREBUILTS.html for more.

TARGET_ARCH
    Name of the target CPU architecture as it is specified by the full Android open-source build. This is 'arm' for any ARM-compatible
    build, independent of the CPU architecture revision.

TARGET_PLATFORM
    Name of the target Android platform when this Android.mk is parsed.
    For example, 'android-3' correspond to Android 1.5 system images. For a complete list of platform names and corresponding Android system
    images, read docs/STABLE-APIS.html.

TARGET_ARCH_ABI
    Name of the target CPU+ABI when this Android.mk is parsed.
    Two values are supported at the moment:

       armeabi For ARMv5TE

       armeabi-v7a

    NOTE: Up to Android NDK 1.6_r1, this variable was simply defined as 'arm'. However, the value has been redefined to better
          match what is used internally by the Android platform.

    For more details about architecture ABIs and corresponding compatibility issues, please read docs/CPU-ARCH-ABIS.html

    Other target ABIs will be introduced in future releases of the NDK and will have a different name. Note that all ARM-based ABIs will
    have 'TARGET_ARCH' defined to 'arm', but may have different
    'TARGET_ARCH_ABI'

TARGET_ABI
    The concatenation of target platform and ABI, it really is defined as $(TARGET_PLATFORM)-$(TARGET_ARCH_ABI) and is useful when you want
    to test against a specific target system image for a real device.

    By default, this will be 'android-3-armeabi'

    (Up to Android NDK 1.6_r1, this used to be 'android-3-arm' by default)


## NDK-provided function macros:

The following are GNU Make 'function' macros, and must be evaluated by using '$(call <function>)'. They return textual information.

my-dir
    Returns the path of the last included Makefile, which typically is the current Android.mk's directory. This is useful to define
    LOCAL_PATH at the start of your Android.mk as with:

        LOCAL_PATH := $(call my-dir)

    IMPORTANT NOTE: Due to the way GNU Make works, this really returns the path of the *last* *included* *Makefile* during the parsing of
    build scripts. Do not call my-dir after including another file.

    For example, consider the following example:

        LOCAL_PATH := $(call my-dir)

        ... declare one module

        include $(LOCAL_PATH)/foo/Android.mk

        LOCAL_PATH := $(call my-dir)

        ... declare another module

    The problem here is that the second call to 'my-dir' will define LOCAL_PATH to $PATH/foo instead of $PATH, due to the include that
    was performed before that.

    For this reason, it's better to put additional includes after everything else in an Android.mk, as in:

        LOCAL_PATH := $(call my-dir)

        ... declare one module

        LOCAL_PATH := $(call my-dir)

        ... declare another module

        # extra includes at the end of the Android.mk
        include $(LOCAL_PATH)/foo/Android.mk

    If this is not convenient, save the value of the first my-dir call into another variable, for example:

        MY_LOCAL_PATH := $(call my-dir)

        LOCAL_PATH := $(MY_LOCAL_PATH)

        ... declare one module

        include $(LOCAL_PATH)/foo/Android.mk

        LOCAL_PATH := $(MY_LOCAL_PATH)

        ... declare another module


all-subdir-makefiles
    Returns a list of Android.mk located in all sub-directories of the current 'my-dir' path. For example, consider the following
    hierarchy:

        sources/foo/Android.mk
        sources/foo/lib1/Android.mk
        sources/foo/lib2/Android.mk

    If sources/foo/Android.mk contains the single line:

        include $(call all-subdir-makefiles)

    Then it will include automatically sources/foo/lib1/Android.mk and sources/foo/lib2/Android.mk

    This function can be used to provide deep-nested source directory hierarchies to the build system. Note that by default, the NDK
    will only look for files in sources/*/Android.mk

this-makefile
    Returns the path of the current Makefile (i.e. where the function is called).

parent-makefile
    Returns the path of the parent Makefile in the inclusion tree, i.e. the path of the Makefile that included the current one.

grand-parent-makefile
    Guess what...

import-module
    A function that allows you to find and include the Android.mk of another module by name. A typical example is:

      $(call import-module,<name>)

    And this will look for the module tagged <name> in the list of directories referenced by your NDK_MODULE_PATH environment
    variable, and include its Android.mk automatically for you.

    Read docs/IMPORT-MODULE.html for more details.


## Module-description variables:

The following variables are used to describe your module to the build system. You should define some of them between an 'include $(CLEAR_VARS)'
and an 'include $(BUILD_XXXXX)'. As written previously, $(CLEAR_VARS) is a script that will undefine/clear all of these variables, unless explicitly
noted in their description.

LOCAL_PATH
    This variable is used to give the path of the current file.
    You MUST define it at the start of your Android.mk, which can be done with:

      LOCAL_PATH := $(call my-dir)

    This variable is *not* cleared by $(CLEAR_VARS) so only one definition per Android.mk is needed (in case you define several
    modules in a single file).

LOCAL_MODULE
    This is the name of your module. It must be unique among all module names, and shall not contain any space. You MUST define
    it before including any $(BUILD_XXXX) script.

    By default, the module name determines the name of generated files,
    e.g. lib<foo>.so for a shared library module named <foo>. However you should only refer to other modules with their 'normal'
    name (e.g. <foo>) in your NDK build files (either Android.mk or Application.mk)

    You can override this default with LOCAL_MODULE_FILENAME (see below)

LOCAL_MODULE_FILENAME
    This variable is optional, and allows you to redefine the name of generated files. By default, module <foo> will always generate a
    static library named lib<foo>.a or a shared library named lib<foo>.so, which are standard Unix conventions.

    You can override this by defining LOCAL_MODULE_FILENAME, For example:

        LOCAL_MODULE := foo-version-1
        LOCAL_MODULE_FILENAME := libfoo

    NOTE: You should not put a path or file extension in your LOCAL_MODULE_FILENAME, these will be handled automatically by the
    build system.

LOCAL_SRC_FILES
    This is a list of source files that will be built for your module.
    Only list the files that will be passed to a compiler, since the build system automatically computes dependencies for you.

    Note that source files names are all relative to LOCAL_PATH and you can use path components, e.g.:

      LOCAL_SRC_FILES := foo.c \
                         toto/bar.c

    NOTE: Always use Unix-style forward slashes (/) in build files.  Windows-style back-slashes will not be handled properly.

LOCAL_CPP_EXTENSION
    This is an optional variable that can be defined to indicate
    the file extension of C++ source files. The default is '.cpp' but you can change it. For example:

        LOCAL_CPP_EXTENSION := .cxx

    Since NDK r7, you can list several extensions in this variable, as in:

        LOCAL_CPP_EXTENSION := .cxx .cpp .cc

LOCAL_CPP_FEATURES
    This is an optional variable that can be defined to indicate
    that your code relies on specific C++ features. To indicate that
    your code uses RTTI (RunTime Type Information), use the following:

        LOCAL_CPP_FEATURES := rtti

    To indicate that your code uses C++ exceptions, use:

        LOCAL_CPP_FEATURES := exceptions

    You can also use both of them with (order is not important):

        LOCAL_CPP_FEATURES := rtti exceptions

    The effect of this variable is to enable the right compiler/linker
    flags when building your modules from sources. For prebuilt binaries,
    this also helps declare which features the binary relies on to ensure
    the final link works correctly.

    It is recommended to use this variable instead of enabling -frtti and
    -fexceptions directly in your LOCAL_CPPFLAGS definition.

LOCAL_C_INCLUDES
    An optional list of paths, relative to the NDK *root* directory,
    which will be appended to the include search path when compiling all sources (C, C++ and Assembly). For example:

        LOCAL_C_INCLUDES := sources/foo

    Or even:

        LOCAL_C_INCLUDES := $(LOCAL_PATH)/../foo

    These are placed before any corresponding inclusion flag in
    LOCAL_CFLAGS / LOCAL_CPPFLAGS

    The LOCAL_C_INCLUDES path are also used automatically when launching native debugging with ndk-gdb.


LOCAL_CFLAGS
    An optional set of compiler flags that will be passed when building C *and* C++ source files.

    This can be useful to specify additional macro definitions or compile options.

    IMPORTANT: Try not to change the optimization/debugging level in your Android.mk, this can be handled automatically for
               you by specifying the appropriate information in your Application.mk, and will let the NDK generate
               useful data files used during debugging.

    NOTE: In android-ndk-1.5_r1, the corresponding flags only applied to C source files, not C++ ones. This has been corrected to
          match the full Android build system behaviour. (You can use LOCAL_CPPFLAGS to specify flags for C++ sources only now).

    It is possible to specify additional include paths with
    LOCAL_CFLAGS += -I<path>, however, it is better to use LOCAL_C_INCLUDES
    for this, since the paths will then also be used during native debugging with ndk-gdb.


LOCAL_CXXFLAGS
    An alias for LOCAL_CPPFLAGS. Note that use of this flag is obsolete as it may disappear in future releases of the NDK.

LOCAL_CPPFLAGS
    An optional set of compiler flags that will be passed when building C++ source files *only*. They will appear after the LOCAL_CFLAGS
    on the compiler's command-line.

    NOTE: In android-ndk-1.5_r1, the corresponding flags applied to both C and C++ sources. This has been corrected to match the
          full Android build system. (You can use LOCAL_CFLAGS to specify flags for both C and C++ sources now).

LOCAL_STATIC_LIBRARIES
    The list of static libraries modules (built with BUILD_STATIC_LIBRARY) that should be linked to this module. This only makes sense in
    shared library modules.

LOCAL_SHARED_LIBRARIES
    The list of shared libraries *modules* this module depends on at runtime.
    This is necessary at link time and to embed the corresponding information in the generated file.

LOCAL_WHOLE_STATIC_LIBRARIES
    A variant of LOCAL_STATIC_LIBRARIES used to express that the corresponding library module should be used as "whole archives" to the linker. See the
    GNU linker's documentation for the --whole-archive flag.

    This is generally useful when there are circular dependencies between several static libraries. Note that when used to build a shared library,
    this will force all object files from your whole static libraries to be added to the final binary. This is not true when generating executables
    though.

LOCAL_LDLIBS
    The list of additional linker flags to be used when building your module. This is useful to pass the name of specific system libraries
    with the "-l" prefix. For example, the following will tell the linker to generate a module that links to /system/lib/libz.so at load time:

      LOCAL_LDLIBS := -lz

    See docs/STABLE-APIS.html for the list of exposed system libraries you can linked against with this NDK release.

LOCAL_ALLOW_UNDEFINED_SYMBOLS
    By default, any undefined reference encountered when trying to build a shared library will result in an "undefined symbol" error. This is a
    great help to catch bugs in your source code.

    However, if for some reason you need to disable this check, set this variable to 'true'. Note that the corresponding shared library may fail
    to load at runtime.

LOCAL_ARM_MODE
    By default, ARM target binaries will be generated in 'thumb' mode, where each instruction are 16-bit wide. You can define this variable to 'arm'
    if you want to force the generation of the module's object files in 'arm' (32-bit instructions) mode. E.g.:

      LOCAL_ARM_MODE := arm

    Note that you can also instruct the build system to only build specific sources in ARM mode by appending an '.arm' suffix to its source file
    name. For example, with:

       LOCAL_SRC_FILES := foo.c bar.c.arm

    Tells the build system to always compile 'bar.c' in ARM mode, and to build foo.c according to the value of LOCAL_ARM_MODE.

    NOTE: Setting APP_OPTIM to 'debug' in your Application.mk will also force the generation of ARM binaries as well. This is due to bugs in the
          toolchain debugger that don't deal too well with thumb code.

LOCAL_ARM_NEON
    Defining this variable to 'true' allows the use of ARM Advanced SIMD (a.k.a. NEON) GCC intrinsics in your C and C++ sources, as well as
    NEON instructions in Assembly files.

    You should only define it when targeting the 'armeabi-v7a' ABI that corresponds to the ARMv7 instruction set. Note that not all ARMv7
    based CPUs support the NEON instruction set extensions and that you should perform runtime detection to be able to use this code at runtime
    safely. To learn more about this, please read the documentation at docs/CPU-ARM-NEON.html and docs/CPU-FEATURES.html.

    Alternatively, you can also specify that only specific source files may be compiled with NEON support by using the '.neon' suffix, as in:

        LOCAL_SRC_FILES = foo.c.neon bar.c zoo.c.arm.neon

    In this example, 'foo.c' will be compiled in thumb+neon mode, 'bar.c' will be compiled in 'thumb' mode, and 'zoo.c' will be
    compiled in 'arm+neon' mode.

    Note that the '.neon' suffix must appear after the '.arm' suffix if you use both (i.e. foo.c.arm.neon works, but not foo.c.neon.arm !)

LOCAL_DISABLE_NO_EXECUTE
    Android NDK r4 added support for the "NX bit" security feature.
    It is enabled by default, but you can disable it if you *really* need to by setting this variable to 'true'.

    NOTE: This feature does not modify the ABI and is only enabled on kernels targeting ARMv6+ CPU devices. Machine code generated
          with this feature enabled will run unmodified on devices running earlier CPU architectures.

    For more information, see:

        http://en.wikipedia.org/wiki/NX_bit
        http://www.gentoo.org/proj/en/hardened/gnu-stack.xml

LOCAL_DISABLE_RELRO
    By default, NDK compiled code is built with read-only relocations and GOT protection.  This instructs the runtime linker to mark
    certain regions of memory as being read-only after relocation, making certain security exploits (such as GOT overwrites) harder
    to perform.

    It is enabled by default, but you can disable it if you *really* need to by setting this variable to 'true'.

    NOTE: These protections are only effective on newer Android devices ("Jelly Bean" and beyond). The code will still run on older
          versions (albeit without memory protections).

    For more information, see:

        http://isisblogs.poly.edu/2011/06/01/relro-relocation-read-only/
        http://www.akkadia.org/drepper/nonselsec.pdf (section 6)

LOCAL_EXPORT_CFLAGS
    Define this variable to record a set of C/C++ compiler flags that will be added to the LOCAL_CFLAGS definition of any other module that uses
    this one with LOCAL_STATIC_LIBRARIES or LOCAL_SHARED_LIBRARIES.

    For example, consider the module 'foo' with the following definition:

        include $(CLEAR_VARS)
        LOCAL_MODULE := foo
        LOCAL_SRC_FILES := foo/foo.c
        LOCAL_EXPORT_CFLAGS := -DFOO=1
        include $(BUILD_STATIC_LIBRARY)

    And another module, named 'bar' that depends on it as:

        include $(CLEAR_VARS)
        LOCAL_MODULE := bar
        LOCAL_SRC_FILES := bar.c
        LOCAL_CFLAGS := -DBAR=2
        LOCAL_STATIC_LIBRARIES := foo
        include $(BUILD_SHARED_LIBRARY)

    Then, the flags '-DFOO=1 -DBAR=2' will be passed to the compiler when building bar.c

    Exported flags are prepended to your module's LOCAL_CFLAGS so you can easily override them. They are also transitive: if 'zoo' depends on
    'bar' which depends on 'foo', then 'zoo' will also inherit all flags exported by 'foo'.

    Finally, exported flags are *not* used when building the module that exports them. In the above example, -DFOO=1 would not be passed to the
    compiler when building foo/foo.c.

LOCAL_EXPORT_CPPFLAGS
    Same as LOCAL_EXPORT_CFLAGS, but for C++ flags only.

LOCAL_EXPORT_C_INCLUDES
    Same as LOCAL_EXPORT_CFLAGS, but for C include paths.
    This can be useful if 'bar.c' wants to include headers that are provided by module 'foo'.

LOCAL_EXPORT_LDLIBS
    Same as LOCAL_EXPORT_CFLAGS, but for linker flags. Note that the imported linker flags will be appended to your module's LOCAL_LDLIBS
    though, due to the way Unix linkers work.

    This is typically useful when module 'foo' is a static library and has code that depends on a system library. LOCAL_EXPORT_LDLIBS can then be
    used to export the dependency. For example:

        include $(CLEAR_VARS)
        LOCAL_MODULE := foo
        LOCAL_SRC_FILES := foo/foo.c
        LOCAL_EXPORT_LDLIBS := -llog
        include $(BUILD_STATIC_LIBRARY)

        include $(CLEAR_VARS)
        LOCAL_MODULE := bar
        LOCAL_SRC_FILES := bar.c
        LOCAL_STATIC_LIBRARIES := foo
        include $(BUILD_SHARED_LIBRARY)

    There, libbar.so will be built with a -llog at the end of the linker command to indicate that it depends on the system logging library,
    because it depends on 'foo'.

LOCAL_SHORT_COMMANDS
    Set this variable to 'true' when your module has a very high number of sources and/or dependent static or shared libraries. This forces the
    build system to use an intermediate list file, and use it with the library archiver or static linker with the @$(listfile) syntax.

    This can be useful on Windows, where the command-line only accepts a maximum of 8191 characters, which can be too small for complex
    projects.

    This also impacts the compilation of individual source files, placing nearly all compiler flags inside list files too.

    Note that any other value than 'true' will revert to the default behaviour. You can also define APP_SHORT_COMMANDS in your
    Application.mk to force this behaviour for all modules in your project.

    NOTE: We do not recommend enabling this feature by default, since it makes the build slower.

LOCAL_FILTER_ASM
    Define this variable to a shell command that will be used to filter the assembly files from, or generated from, your LOCAL_SRC_FILES.

    When it is defined, the following happens:

      - Any C or C++ source file is generated into a temporary assembly file (instead of being compiled into an object file).

      - Any temporary assembly file, and any assembly file listed in LOCAL_SRC_FILES is sent through the LOCAL_FILTER_ASM command
        to generate _another_ temporary assembly file.

      - These filtered assembly files are compiled into object file.

    In other words, If you have:

      LOCAL_SRC_FILES  := foo.c bar.S
      LOCAL_FILTER_ASM := myasmfilter


    foo.c --1--> $OBJS_DIR/foo.S.original --2--> $OBJS_DIR/foo.S --3--> $OBJS_DIR/foo.o
    bar.S                                 --2--> $OBJS_DIR/bar.S --3--> $OBJS_DIR/bar.o


    Were "1" corresponds to the compiler, "2" to the filter, and "3" to the assembler. The filter must be a standalone shell command that takes the
    name of the input file as its first argument, and the name of the output file as the second one, as in:

        myasmfilter $OBJS_DIR/foo.S.original $OBJS_DIR/foo.S
        myasmfilter bar.S $OBJS_DIR/bar.S


NDK_TOOLCHAIN_VERSION
    Define this variable to either 4.4.3 or 4.6 to select version of GCC compiler.  4.6 is the default


The Android Build Cookbook offers code snippets to help you quickly implement some common build tasks. For additional instruction, please see the other build documents in this section.
## Building a simple APK
  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)
   
  # Build all java files in the java subdirectory
  LOCAL_SRC_FILES := $(call all-subdir-java-files)
   
  # Name of the APK to build
  LOCAL_PACKAGE_NAME := LocalPackage
   
  # Tell it to build an APK
  include $(BUILD_PACKAGE)

## Building a APK that depends on a static .jar file
  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)
   
  # List of static libraries to include in the package
  LOCAL_STATIC_JAVA_LIBRARIES := static-library
   
  # Build all java files in the java subdirectory
  LOCAL_SRC_FILES := $(call all-subdir-java-files)
   
  # Name of the APK to build
  LOCAL_PACKAGE_NAME := LocalPackage
   
  # Tell it to build an APK
  include $(BUILD_PACKAGE)

## Building a APK that should be signed with the platform key
  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)
   
  # Build all java files in the java subdirectory
  LOCAL_SRC_FILES := $(call all-subdir-java-files)
   
  # Name of the APK to build
  LOCAL_PACKAGE_NAME := LocalPackage
   
  LOCAL_CERTIFICATE := platform
   
  # Tell it to build an APK
  include $(BUILD_PACKAGE)

## Building a APK that should be signed with a specific vendor key
  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)
   
  # Build all java files in the java subdirectory
  LOCAL_SRC_FILES := $(call all-subdir-java-files)
   
  # Name of the APK to build
  LOCAL_PACKAGE_NAME := LocalPackage
   
  LOCAL_CERTIFICATE := vendor/example/certs/app
   
  # Tell it to build an APK
  include $(BUILD_PACKAGE)

## Adding a prebuilt APK
  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)
   
  # Module name should match apk name to be installed.
  LOCAL_MODULE := LocalModuleName
  LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
  LOCAL_MODULE_CLASS := APPS
  LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
   
  include $(BUILD_PREBUILT)

## Adding a Static Java Library
  LOCAL_PATH := $(call my-dir)
  include $(CLEAR_VARS)
   
  # Build all java files in the java subdirectory
  LOCAL_SRC_FILES := $(call all-subdir-java-files)
   
  # Any libraries that this library depends on
  LOCAL_JAVA_LIBRARIES := android.test.runner
   
  # The name of the jar file to create
  LOCAL_MODULE := sample
   
  # Build a static jar file.
  include $(BUILD_STATIC_JAVA_LIBRARY)

## Android.mk Variables
These are the variables that you'll commonly see in Android.mk files, listed alphabetically. First, a note on the variable naming: 
* LOCAL_ - These variables are set per-module. They are cleared by the include $(CLEAR_VARS) line, so you can rely on them being empty after including that file. Most of the variables you'll use in most modules are LOCAL_ variables.
* PRIVATE_ - These variables are make-target-specific variables. That means they're only usable within the commands for that module. It also means that they're unlikely to change behind your back from modules that are included after yours. This link to the make documentation describes more about target-specific variables.
* HOST_ and TARGET_ - These contain the directories and definitions that are specific to either the host or the target builds. Do not set variables that start with HOST_ or TARGET_ in your makefiles.
* BUILD_ and CLEAR_VARS - These contain the names of well-defined template makefiles to include. Some examples are CLEAR_VARS and BUILD_HOST_PACKAGE.
* Any other name is fair-game for you to use in your Android.mk. However, remember that this is a non-recursive build system, so it is possible that your variable will be changed by another Android.mk included later, and be different when the commands for your rule / module are executed.

| Android.mk variables |  Description |
|----|----|
| LOCAL_AAPT_FLAGS |  |
| LOCAL_ACP_UNAVAILABLE |  |
| LOCAL_ADDITIONAL_JAVA_DIR |  |
| LOCAL_AIDL_INCLUDES |  |
| LOCAL_ALLOW_UNDEFINED_SYMBOLS |  |
| LOCAL_ARM_MODE |  |
| LOCAL_ASFLAGS |  |
| LOCAL_ASSET_DIR |  |
| LOCAL_ASSET_FILES | In Android.mk files that include $(BUILD_PACKAGE) set this to the set of files you want built into your app. Usually:LOCAL_ASSET_FILES += $(call find-subdir-assets) |
| LOCAL_BUILT_MODULE_STEM |  |
| LOCAL_C_INCLUDES | Additional directories to instruct the C/C++ compilers to look for header files in. These paths are rooted at the top of the tree. Use LOCAL_PATH if you have subdirectories of your own that you want in the include paths. For example:LOCAL_C_INCLUDES += extlibs/zlib-1.2.3LOCAL_C_INCLUDES += $(LOCAL_PATH)/srcYou should not add subdirectories of include to LOCAL_C_INCLUDES, instead you should reference those files in the #includestatement with their subdirectories. For example:#include <utils/KeyedVector.h>not #include <KeyedVector.h> |
| LOCAL_CC | If you want to use a different C compiler for this module, set LOCAL_CC to the path to the compiler. If LOCAL_CC is blank, the appropriate default compiler is used. |
| LOCAL_CERTIFICATE |  |
| LOCAL_CFLAGS | If you have additional flags to pass into the C or C++ compiler, add them here. For example:LOCAL_CFLAGS += -DLIBUTILS_NATIVE=1 |
| LOCAL_CLASSPATH |  |
| LOCAL_COMPRESS_MODULE_SYMBOLS | |
| LOCAL_COPY_HEADERS | The set of files to copy to the install include tree. You must also supply LOCAL_COPY_HEADERS_TO.This is going away because copying headers messes up the error messages, and may lead to people editing those headers instead of the correct ones. It also makes it easier to do bad layering in the system, which we want to avoid. We also aren't doing a C/C++ SDK, so there is no ultimate requirement to copy any headers. |
| LOCAL_COPY_HEADERS_TO | The directory within "include" to copy the headers listed in LOCAL_COPY_HEADERS to.This is going away because copying headers messes up the error messages, and may lead to people editing those headers instead of the correct ones. It also makes it easier to do bad layering in the system, which we want to avoid. We also aren't doing a C/C++ SDK, so there is no ultimate requirement to copy any headers. |
| LOCAL_CPP_EXTENSION | If your C++ files end in something other than ".cpp", you can specify the custom extension here. For example:LOCAL_CPP_EXTENSION := .ccNote that all C++ files for a given module must have the same extension; it is not currently possible to mix different extensions. |
| LOCAL_CPPFLAGS | If you have additional flags to pass into only the C++ compiler, add them here. For example:LOCAL_CPPFLAGS += -ffriend-injectionLOCAL_CPPFLAGS is guaranteed to be after LOCAL_CFLAGS on the compile line, so you can use it to override flags listed inLOCAL_CFLAGS |
| LOCAL_CXX | If you want to use a different C++ compiler for this module, set LOCAL_CXX to the path to the compiler. If LOCAL_CXX is blank, the appropriate default compiler is used. |
| LOCAL_DX_FLAGS |  |
| LOCAL_EXPORT_PACKAGE_RESOURCES |  |
| LOCAL_FORCE_STATIC_EXECUTABLE | If your executable should be linked statically, set LOCAL_FORCE_STATIC_EXECUTABLE:=true. There is a very short list of libraries that we have in static form (currently only libc). This is really only used for executables in /sbin on the root filesystem. |
| LOCAL_GENERATED_SOURCES | Files that you add to LOCAL_GENERATED_SOURCES will be automatically generated and then linked in when your module is built. See the Custom Tools template makefile for an example. |
| LOCAL_INSTRUMENTATION_FOR |  |
| LOCAL_INSTRUMENTATION_FOR_PACKAGE_NAME |  |
| LOCAL_INTERMEDIATE_SOURCES |  |
| LOCAL_INTERMEDIATE_TARGETS |  |
| LOCAL_IS_HOST_MODULE |  |
| LOCAL_JAR_MANIFEST |  |
| LOCAL_JARJAR_RULES |  |
| LOCAL_JAVA_LIBRARIES | When linking Java apps and libraries, LOCAL_JAVA_LIBRARIES specifies which sets of java classes to include. Currently there are two of these: core and framework. In most cases, it will look like this:LOCAL_JAVA_LIBRARIES := core frameworkNote that setting LOCAL_JAVA_LIBRARIES is not necessary (and is not allowed) when building an APK with "include $(BUILD_PACKAGE)". The appropriate libraries will be included automatically. |
| LOCAL_JAVA_RESOURCE_DIRS |  |
| LOCAL_JAVA_RESOURCE_FILES |  |
| LOCAL_JNI_SHARED_LIBRARIES |  |
| LOCAL_LDFLAGS | You can pass additional flags to the linker by setting LOCAL_LDFLAGS. Keep in mind that the order of parameters is very important to ld, so test whatever you do on all platforms. |
| LOCAL_LDLIBS | LOCAL_LDLIBS allows you to specify additional libraries that are not part of the build for your executable or library. Specify the libraries you want in -lxxx format; they're passed directly to the link line. However, keep in mind that there will be no dependency generated for these libraries. It's most useful in simulator builds where you want to use a library preinstalled on the host. The linker (ld) is a particularly fussy beast, so it's sometimes necessary to pass other flags here if you're doing something sneaky. Some examples:LOCAL_LDLIBS += -lcurses -lpthreadLOCAL_LDLIBS += -Wl,-z,origin |
| LOCAL_MODULE | LOCAL_MODULE is the name of what's supposed to be generated from your Android.mk. For exmample, for libkjs, the LOCAL_MODULE is "libkjs" (the build system adds the appropriate suffix -- .so .dylib .dll). For app modules, use LOCAL_PACKAGE_NAME instead of LOCAL_MODULE.  |
| LOCAL_MODULE_PATH | Instructs the build system to put the module somewhere other than what's normal for its type. If you override this, make sure you also set LOCAL_UNSTRIPPED_PATH if it's an executable or a shared library so the unstripped binary has somewhere to go. An error will occur if you forget to.See Putting modules elsewhere for more. |
| LOCAL_MODULE_STEM |  |
| LOCAL_MODULE_TAGS | Set LOCAL_MODULE_TAGS to any number of whitespace-separated tags. This variable controls what build flavors the package gets included in. For example:user: include this in user/userdebug buildseng: include this in eng buildstests: the target is a testing target and makes it available for testsoptional: don't include this |
| LOCAL_NO_DEFAULT_COMPILER_FLAGS |  |
| LOCAL_NO_EMMA_COMPILE |  |
| LOCAL_NO_EMMA_INSTRUMENT |  |
| LOCAL_NO_STANDARD_LIBRARIES |  |
| LOCAL_OVERRIDES_PACKAGES |  |
| LOCAL_PACKAGE_NAME | LOCAL_PACKAGE_NAME is the name of an app. For example, Dialer, Contacts, etc.  |
| LOCAL_POST_PROCESS_COMMAND | For host executables, you can specify a command to run on the module after it's been linked. You might have to go through some contortions to get variables right because of early or late variable evaluation:module := $(HOST_OUT_EXECUTABLES)/$(LOCAL_MODULE)LOCAL_POST_PROCESS_COMMAND := /Developer/Tools/Rez -d __DARWIN__ -t APPL\       -d __WXMAC__ -o $(module) Carbon.r |
| LOCAL_PREBUILT_EXECUTABLES | When including $(BUILD_PREBUILT) or $(BUILD_HOST_PREBUILT), set these to executables that you want copied. They're located automatically into the right bin directory. |
| LOCAL_PREBUILT_JAVA_LIBRARIES |  |
| LOCAL_PREBUILT_LIBS | When including $(BUILD_PREBUILT) or $(BUILD_HOST_PREBUILT), set these to libraries that you want copied. They're located automatically into the right lib directory. |
| LOCAL_PREBUILT_OBJ_FILES |  |
| LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES |  |
| LOCAL_PRELINK_MODULE |  |
| LOCAL_REQUIRED_MODULES | Set LOCAL_REQUIRED_MODULES to any number of whitespace-separated module names, like "libblah" or "Email". If this module is installed, all of the modules that it requires will be installed as well. This can be used to, e.g., ensure that necessary shared libraries or providers are installed when a given app is installed. |
| LOCAL_RESOURCE_DIR |  |
| LOCAL_SDK_VERSION |  |
| LOCAL_SHARED_LIBRARIES | These are the libraries you directly link against. You don't need to pass transitively included libraries. Specify the name without the suffix:LOCAL_SHARED_LIBRARIES := \    libutils \    libui \    libaudio \    libexpat \    libsgl |
| LOCAL_SRC_FILES | The build system looks at LOCAL_SRC_FILES to know what source files to compile -- .cpp .c .y .l .java. For lex and yacc files, it knows how to correctly do the intermediate .h and .c/.cpp files automatically. If the files are in a subdirectory of the one containing the Android.mk, prefix them with the directory name:LOCAL_SRC_FILES := \    file1.cpp \    dir/file2.cpp |
| LOCAL_STATIC_JAVA_LIBRARIES |  |
| LOCAL_STATIC_LIBRARIES | These are the static libraries that you want to include in your module. Mostly, we use shared libraries, but there are a couple of places, like executables in sbin and host executables where we use static libraries instead.LOCAL_STATIC_LIBRARIES := \    libutils \    libtinyxml |
| LOCAL_UNINSTALLABLE_MODULE |  |
| LOCAL_UNSTRIPPED_PATH | Instructs the build system to put the unstripped version of the module somewhere other than what's normal for its type. Usually, you override this because you overrode LOCAL_MODULE_PATH for an executable or a shared library. If you overrode LOCAL_MODULE_PATH, but not LOCAL_UNSTRIPPED_PATH, an error will occur.See Putting modules elsewhere for more. |
| LOCAL_WHOLE_STATIC_LIBRARIES | These are the static libraries that you want to include in your module without allowing the linker to remove dead code from them. This is mostly useful if you want to add a static library to a shared library and have the static library's content exposed from the shared library.LOCAL_WHOLE_STATIC_LIBRARIES := \    libsqlite3_android |
| LOCAL_YACCFLAGS | Any flags to pass to invocations of yacc for your module. A known limitation here is that the flags will be the same for all invocations of YACC for your module. This can be fixed. If you ever need it to be, just ask.LOCAL_YACCFLAGS := -p kjsyy |
| OVERRIDE_BUILT_MODULE_PATH |  |

↑ Go to top
Except as noted, this content is licensed under Creative Commons Attribution 2.5.
Gathered from Android Open Source Project Documentation. 
If you would like to contribute, fork https://github.com/kimocode/android.mk and submit a pull request.