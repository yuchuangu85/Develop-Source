<h1 align="center">Open Source License</h1>

世界上的开源许可证（Open Source License）大概有上百种，今天我们来介绍下几种我们常见的开源协议。大致有GPL、BSD、MIT、Mozilla、Apache和LGPL等。

![全球各种开源协议，搞研发的学习下](https://p3-tt.byteimg.com/origin/pgc-image/51cf373a6b5a423e80a1b71b637326fc?from=pc)



下面通过这几张图，大家可以一目了然地选择合适的开源协议：

![全球各种开源协议，搞研发的学习下](https://p3-tt.byteimg.com/origin/pgc-image/dbdcf4e4f3cb40568c9d81a115f80700.png?from=pc)



**乌克兰程序员 Paul Bagwell 画了一张分析图**



![全球各种开源协议，搞研发的学习下](https://p1-tt.byteimg.com/origin/pgc-image/46a9477e9dac4d81bfc1ed8f731980de?from=pc)



<img src="https://p1-tt.byteimg.com/origin/pgc-image/c64f754d1dcc4f8595a1702daa57956c?from=pc" alt="全球各种开源协议，搞研发的学习下" style="zoom: 67%;" />



**详细介绍常用开源协议
**
**GNU GPL（GNU General Public License，GNU通用公共许可证）**

<img src="https://p3-tt.byteimg.com/origin/pgc-image/a6e59b14688b42eda6b8e263cc378d05?from=pc" alt="GNU" style="zoom:33%;" />

只要软件中包含了遵循 GPL 协议的产品或代码，该软件就必须也遵循 GPL 许可协议，也就是必须开源免费，不能闭源收费，因此这个协议并不适合商用软件。

遵循 GPL 协议的开源软件数量极其庞大，包括 Linux 系统在内的大多数的开源软件都是基于这个协议的。

**GPL 开源协议的主要特点:**

| **特点** | **说明**                                                     |
| -------- | ------------------------------------------------------------ |
| 复制自由 | 允许把软件复制到任何人的电脑中，并且不限制复制的数量。       |
| 传播自由 | 允许软件以各种形式进行传播。                                 |
| 收费传播 | 允许在各种媒介上出售该软件，但必须提前让买家知道这个软件是可以免费获得的；因此，一般来讲，开源软件都是通过为用户提供有偿服务的形式来盈利的。 |
| 修改自由 | 允许开发人员增加或删除软件的功能，但软件修改后必须依然基于GPL许可协议授权。 |

**BSD（Berkeley Software Distribution，伯克利软件发布版）协议**

<img src="https://p6-tt.byteimg.com/origin/pgc-image/2893eca7c1c84f3f9e31a7cde1ee6a6a?from=pc" alt="全球各种开源协议，搞研发的学习下" style="zoom:33%;" />

BSD 协议给予用户极大的权利，用户可以使用、修改和重新发布遵循该许可的软件，并且可以将软件作为商业软件发布和销售，前提是需要满足下面三个条件：

- 如果再发布的软件中包含源代码，则源代码必须继续遵循 BSD 许可协议。
- 如果再发布的软件中只有二进制程序，则需要在相关文档或版权文件中声明原始代码遵循了 BSD 协议。不允许用原始软件的名字、作者名字或机构名称等进行市场推广。

BSD 对商业比较友好，很多公司在选用开源产品的时候都首选 BSD 协议，因为可以完全控制这些第三方的代码，甚至在必要的时候可以修改或者二次开发。

**Apache 许可证版本（Apache License Version）协议**

<img src="https://p3-tt.byteimg.com/origin/pgc-image/62ac7a146fd24fd8b0504a5f8a81aa09?from=pc" alt="全球各种开源协议，搞研发的学习下" style="zoom:33%;" />

Apache 和 BSD 类似，都适用于商业软件。Apache 协议在为开发人员提供版权及专利许可的同时，允许用户拥有修改代码及再发布的自由。
Hadoop、Apache HTTP Server、MongoDB 等项目都是基于该许可协议研发的，程序开发人员在开发遵循该协议的软件时，要严格遵守下面的四个条件：

- 该软件及其衍生品必须继续使用 Apache 许可协议。
- 如果修改了程序源代码，需要在文档中进行声明。若软件是基于他人的源代码编写而成的，则需要保留原始代码的协议、商标、专利声明及其他原作者声明的内容信息。如果再发布的软件中有声明文件，则需在此文件中标注 Apache 许可协议及其他许可协议。

**Apache 协议还有以下需要说明的地方:**

- **永久权利:** 一旦被授权，永久拥有。
- **全球范围的权利:** 在一个国家获得授权，适用于所有国家。
- **授权免费，且无版税**: 前期，后期均无任何费用。
- **授权无排他性:** 任何人都可以获得授权
- **授权不可撤消:** 一旦获得授权，没有任何人可以取消。比如，你基于该产品代码开发了衍生产品，你不用担心会在某一天被禁止使用该代码。

- **MIT（Massachusetts Institute of Technology）协议**

又称「X条款」或「X11条款」，目前限制最少的开源许可协议之一（比 BSD 和 Apache 的限制都少），只要程序的开发者在修改后的源代码中保留原作者的许可信息即可，因此普遍被商业软件所使用。
使用 MIT 协议的软件有 PuTTY、X Window System、Ruby on Rails、Lua 5.0 onwards、Mono 等。

**GUN LGPL（GNU Lesser General Public License，GNU 宽通用公共许可证）**

LGPL 是 GPL 的一个衍生版本，也被称为 GPL V2，该协议主要是为类库设计的开源协议。

LGPL 允许商业软件通过类库引用（link）的方式使用 LGPL 类库，而不需要开源商业软件的代码。这使得采用 LGPL 协议的开源代码可以被商业软件作为类库引用并发布和销售。

但是如果修改 LGPL 协议的代码或者衍生品，则所有修改的代码，涉及修改部分的额外代码和衍生的代码都必须采用 LGPL 协议。

因此LGPL协议的开源代码很适合作为第三方类库被商业软件引用，但不适合希望以 LGPL 协议代码为基础，通过修改和衍生的方式做二次开发的商业软件采用。



来源：https://www.toutiao.com/i6938035733584216583/?tt_from=weixin&utm_campaign=client_share&wxshare_count=1&timestamp=1615419109&app=news_article&utm_source=weixin&utm_medium=toutiao_android&use_new_style=1&req_id=202103110731480101510790280E18B96A&share_token=fe16c715-101d-43cd-a018-66d4e9050fea&group_id=6938035733584216583