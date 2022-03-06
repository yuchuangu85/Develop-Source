<h1 align="center">UML</h1>

[TOC]

## UML详解

[https://www.uml-diagrams.org](https://www.uml-diagrams.org)

## UML依赖关系

<img src="media/%E6%88%AA%E5%B1%8F2021-02-18%2018.52.05.png" alt="截屏2021-02-18 18.52.05" style="zoom:50%;" />

* 依赖：对象B进行修改会影响到A
* 关联：对象A知道对象B。类A依赖于类B。
* 聚合：对象A知道对象B且由对象B构成。类A依赖于类B。
* 组合：对象A知道对象B、由B构成而且管理着B的声明周期。类A依赖于类B。
* 实现：类A定义的方法由接口B声明。对象A可被视为对象B。类A依赖于类B。
* 继承：类A继承类B的接口和实现，但是可以对其进行扩展。对象A可被视为对象B。类A依赖于类B。

