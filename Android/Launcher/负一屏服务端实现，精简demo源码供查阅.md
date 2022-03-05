# 负一屏服务端实现，精简demo源码供查阅
前期两篇文章介绍了下在launcher上集成负一屏的方案，这个方案脱离了launcher本身的Workspace层，而采用了新方案。针对新方案有很多网友提出了很多问题。
1.google feed就是采用了这种方案在launcher3上实现负一屏的，launcher_client找不到的问题，launcher3的release版本源码里已经集成了launcher_client, https://github.com/amirzaidi/launcher3/releases，这里我再提供一份供大家下载分析。

2.google feed是作为服务端的，不开源，所以很多网友还是不清楚服务端的具体实现。我写了个服务端的demo，比较简陋，供参考。
学会了之后，我们就可以独立开发个自定义的负一屏来代替google feed了。

负一屏的实现的博文地址
launcher_client包
负一屏实现demo


————————————————
版权声明：本文为CSDN博主「世界末日不是谣言」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u013032242/article/details/95346709