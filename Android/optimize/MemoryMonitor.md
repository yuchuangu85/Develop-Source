<h1 align="center">内存监控</h1>
```
adb shell dumpsys meminfo package_name|pid [-d]
```



## Dalvik Heap
The RAM used by Dalvik allocations in your app. The Pss Total includes all Zygote allocations (weighted by their sharing across processes, as described in the PSS definition above). The Private Dirty number is the actual RAM committed to only your app’s heap, composed of your own allocations and any Zygote allocation pages that have been modified since forking your app’s process from Zygote.

>Note: On newer platform versions that have the Dalvik Other section, the Pss Total and Private Dirty numbers for Dalvik Heap do not include Dalvik overhead such as the just-in-time compilation (JIT) and GC bookkeeping, whereas older versions list it all combined under Dalvik.

The Heap Alloc is the amount of memory that the Dalvik and native heap allocators keep track of for your app. This value is larger than Pss Total and Private Dirty because your process was forked from Zygote and it includes allocations that your process shares with all the others.

## .so mmap and .dex mmap
The RAM being used for mapped .so (native) and .dex (Dalvik or ART) code. The Pss Total number includes platform code shared across apps; the Private Clean is your app’s own code. Generally, the actual mapped size will be much larger—the RAM here is only what currently needs to be in RAM for code that has been executed by the app. However, the .so mmap has a large private dirty, which is due to fix-ups to the native code when it was loaded into its final address.

## .oat mmap
This is the amount of RAM used by the code image which is based off of the preloaded classes which are commonly used by multiple apps. This image is shared across all apps and is unaffected by particular apps.

## .art mmap
This is the amount of RAM used by the heap image which is based off of the preloaded classes which are commonly used by multiple apps. This image is shared across all apps and is unaffected by particular apps. Even though the ART image contains Object instances, it does not count towards your heap size.

## .Heap (only with -d flag)
This is the amount of heap memory for your app. This excludes objects in the image and large object spaces, but includes the zygote space and non-moving space.

## .LOS (only with -d flag)
This is the amount of RAM used by the ART large object space. This includes zygote large objects. Large objects are all primitive array allocations larger than 12KB.

## .GC (only with -d flag)
This is the overhead cost for garbage collection. There is not really any way to reduce this overhead.

## .JITCache (only with -d flag)
This is the amount of memory used by the JIT data and code caches. Typically, this is zero since all of the apps will be compiled at installed time.

## .Zygote (only with -d flag)
This is the amount of memory used by the zygote space. The zygote space is created during device startup and is never allocated into.

## .NonMoving (only with -d flag)
This is the amount of RAM used by the ART non-moving space. The non-moving space contains special non-movable objects such as fields and methods. You can reduce this section by using fewer fields and methods in your app.

## .IndirectRef (only with -d flag)
This is the amount of RAM used by the ART indirect reference tables. Usually this amount is small, but if it is too high, it might be possible to reduce it by reducing the number of local and global JNI references used.

## Unknown
Any RAM pages that the system could not classify into one of the other more specific items. Currently, this contains mostly native allocations, which cannot be identified by the tool when collecting this data due to Address Space Layout Randomization (ASLR). Like the Dalvik heap, the Pss Total for Unknown takes into account sharing with Zygote, and Private Dirty is unknown RAM dedicated to only your app.

## TOTAL
The total Proportional Set Size (PSS) RAM used by your process. This is the sum of all PSS fields above it. It indicates the overall memory weight of your process, which can be directly compared with other processes and the total available RAM.
The Private Dirty and Private Clean are the total allocations within your process, which are not shared with other processes. Together (especially Private Dirty), this is the amount of RAM that will be released back to the system when your process is destroyed. Dirty RAM is pages that have been modified and so must stay committed to RAM (because there is no swap); clean RAM is pages that have been mapped from a persistent file (such as code being executed) and so can be paged out if not used for a while.

## ViewRootImpl
The number of root views that are active in your process. Each root view is associated with a window, so this can help you identify memory leaks involving dialogs or other windows.

## AppContexts and Activities
The number of app Context and Activity objects that currently live in your process. This can help you to quickly identify leaked Activity objects that can’t be garbage collected due to static references on them, which is common. These objects often have many other allocations associated with them, which makes them a good way to track large memory leaks.