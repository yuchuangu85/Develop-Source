<h1 align="center">Material Design</h1>

介于拟物和扁平之间的Material Design自面世以来，便引起了很多人的关注与思考，就此产生的讨论也不绝于耳。本文详细介绍了在Android开发者圈子里颇受青睐的十个Material Design开源项目，从示例、FAB、菜单、动画、Ripple到Dialog，看被称为“Google第一次在设计语言和规范上超越了Apple”的Material Design是如何逐渐成为App的一种全新设计标准。

1. [MaterialDesignLibrary](https://github.com/navasmdc/MaterialDesignLibrary)

在众多新晋库中，MaterialDesignLibrary可以说是颇受开发者瞩目的一个控件效果库，能够让开发者在Android 2.2系统上使用Android 5.0才支持的控件效果，比如扁平、矩形、浮动按钮，复选框以及各式各样的进度指示器等。

除上述之外，MaterialDesignLibrary还拥有SnackBar、Dialog、Color selector组件，可非常便捷地对应用界面进行设置。

2. [RippleEffect](https://github.com/traex/RippleEffect)

由来自法兰西的Robin Chutaux开发的RippleEffect基于MIT许可协议开源，能够在Android API 9+上实现Material Design，为开发者提供了一种极为简易的方式来创建带有可扩展视图的header视图，并且允许最大程度上的自定制。

3. [MaterialEditText](https://github.com/rengwuxian/MaterialEditText)

随着Material Design的到来，AppCompat v21也为开发者提供了Material Design的控件外观支持，其中就包括EditText，但却并不好用，没有设置颜色的API，也没有任何Google Material Design Spec中提到的特性。于是，来自国内的开发者“扔物线”开发了MaterialEditText库，直接继承EditText，无需修改Java文件即能实现自定义控件颜色。

4. [Android-LollipopShowcase](https://github.com/mikepenz/Android-LollipopShowcase)

Android-LollipopShowcase是由来自奥地利的移动、后端及Web开发者Mike Penz所开发的演示应用，集中演示了新Material Design中所有的UI效果，以及Android Lollipop中其他非常酷炫的特性元素，比如Toolbar、RecyclerView、ActionBarDrawerToggle、Floating Action Button（FAB）、Android Compat Theme等。


5. [MaterialList](https://github.com/dexafree/MaterialList)

MaterialList是一个能够帮助所有Android开发者获取谷歌UI设计规范中新增的CardView（卡片视图）的开源库，支持Android 2.3+系统。作为ListView的扩展，MaterialList可以接收、存储卡片列表，并根据它们的Android风格和设计模式进行展示。此外，开发者还可以创建专属于自己的卡片布局，并轻松将其添加到CardList中。


6. [android-floating-action-button](https://github.com/futuresimple/android-floating-action-button)

Floating Action Button（FAB）是众多专家大牛针对Material Design讨论比较细化的一个点，通过圆形元素与分割线、卡片、各种Bar的直线形成鲜明对比，并使用色彩设定中鲜艳的辅色，带来更具突破性的视觉效果。也正因如此，在Github上，有着许多与FAB相关的开源项目，基于Material Design规范的开源Android浮动Action Button控件android-floating-action-button便是其中之一。


其主要特性如下：

支持常规56dp和最小40dp的按钮；
支持自定义正常、Press状态以及可拖拽图标的按钮背景颜色；
AddFloatingActionButton类能够让开发者非常方便地直接在代码中写入加号图标；
FloatingActionsMenu类支持展开/折叠显示动作。
相关链接：android-floating-action-button的mobilehub主页

7. [android-ui](https://github.com/markushi/android-ui)

android-ui是Android UI组件类库，支持Android API 14+，包含了ActionView、RevealColorView等UI组件。其中，ActionView可使Action动作显示动画效果，而RevealColorView则带来了Android 5.0中的圆形显示/隐藏动画体验。



8. [Material Menu](https://github.com/balysv/material-menu)

Material Menu为开发者带来了非常酷炫的Android菜单、返回、删除以及检查按钮变形，完全控制动画，并为开发者提供了两种MaterialMenuDrawable包装。


9. [Android-ObservableScrollView](https://github.com/ksoichiro/Android-ObservableScrollView)

Android-ObservableScrollView是一款用于在滚动视图中观测滚动事件的Android库。它能够轻而易举地与Android 5.0 Lollipop引进的工具栏（Toolbar）进行交互，还可以帮助开发者实现拥有Material Design应用视觉体验的界面外观，支持ListView、ScrollView、WebView、RecyclerView、GridView组件。


10. [Material Design Icons](https://github.com/google/material-design-icons)

最后，再来介绍一下Google Material Design规范的官方开源图标集Material Design Icons。良心Google开源了包括Material Design系统图标包在内的750个字形，涵盖动作、音视频、通信、内容、编辑器、文件、硬件、图像、地图、导航、通知、社交等各个方面，适用于Web、Android和iOS应用开发，绝对是开发者及设计师必备的资源。



图标格式主要包括： 

SVG格式，24px和48px；
SVG和CSS Sprites；
适用于Web平台的1x、2x PNG格式图标；
适用于iOS的1x、2x、3x PNG图标；
所有图标的Hi-dpi版本（hdpi、mdpi、xhdpi、xxhdpi、xxxhdpi）