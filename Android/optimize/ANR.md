## ANR分类及超时时间
* UI：5s
* BroadcastReceiver：10s
* Service：20s

## ANR原因

引起ANR的原因主要是在主线程做耗时操作导致时间超出系统规定时间限制，就会报出ANR。系统设置ANR的主要原因是给用户更好的体验，防止App卡在某一点没有反应。

#### UI线程出现ANR
这个是在UI线程做耗时操作：
* 点击事件中做耗时操作，比如：加载较大数据，查询数据库，请求网络等
* 在View绘制过程中（dispatchDrow或者onDraw方法中）创建对象或者做耗时操作


#### BroadcastReceiver的onReceive方法中出现ANR
这个是在onReceive方法中做了耗时操作，因此需要创建线程和异步任务，或者启动服务，在服务中启动线程进行处理，避免出现ANR。


### Service出现ANR
这个通常很少出现，但是如果在Service的主线程中有耗时操作同样会出现ANR现象，因此，耗时操作可以通过Thread或者AsyncTask来处理，也可以通过线程池进行处理。

## 头条文章

* [今日头条 ANR 优化实践系列 - 设计原理及影响因素 - 掘金 (juejin.cn)](https://juejin.cn/post/6940061649348853796)
* [今日头条 ANR 优化实践系列 - 监控工具与分析思路 - 掘金 (juejin.cn)](https://juejin.cn/post/6942665216781975582)
* [今日头条 ANR 优化实践系列分享 - 实例剖析集锦 - 掘金 (juejin.cn)](https://juejin.cn/post/6945267342671052807)
* [今日头条 ANR 优化实践系列 - Barrier 导致主线程假死 - 掘金 (juejin.cn)](https://juejin.cn/post/6947986170135445535)
* [今日头条 ANR 优化实践系列 - 告别 SharedPreference 等待 (qq.com)](https://mp.weixin.qq.com/s/kfF83UmsGM5w43rDCH544g)

