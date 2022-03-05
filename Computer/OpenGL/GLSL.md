<h1 align="center">GLSL</h1>

[toc]



## 3种数据修饰类型

glsl提供了3种数据修饰类型，用于在OpenGL ES客户端和服务端之间传递值。

### uniform

uniform可以从客户端传递到顶点种色器、也可以传递到片元着色器，对应的客户端代码glGetUniformLocation用于获取glsl中的变量、glUniform...用于给变量设置值。
 uniform可以传递：视图矩阵、投影矩阵、投影视图矩阵，还有纹理数据

### attribute

attribute从客户端传递到顶点着色器。
 attribute可以传递：顶点、纹理坐标、颜色、法线

### varying

在顶点着色器和片元着色器之间传递数据
 因为纹理坐标无法使用attribute传递到片元着色器，使用varying可以将纹理坐标从顶点着色器传递到片元着色器
 GLSL本身是一段字符串，没有特定的编写文件。所以创建GLSL的编写文件后缀可以随意，后缀的作用可以区分是顶点着色器还是片元着色器.

## 编写glsl代码

### 1.顶点着色器代码shaderv.vsh

```cpp
attribute vec4 position;  //顶点坐标
attribute vec2 textCoordinate; // 纹理坐标
varying lowp vec2 varyTextCoord; // 用于与片元着色器桥接的纹理坐标变量

void main()
{
    varyTextCoord = textCoordinate;
  	// gl_Position：内建变量，顶点说色气计算之后的顶点接口
    gl_Position = position;
}
```

### 2.片元着色器代码shaderf.fsh



```cpp
precision highp float;  //使用高精度，避免一些错误
varying lowp vec2 varyTextCoord; // 接收顶点着色器传递过来的纹理坐标
uniform sampler2D colorMap; //纹理数据

void main()
{
    //1.获取纹理对应坐标下纹素（像素），如果 1200 * 1200 像素，片元着色器会执行1200 *1200次
    // 内建函数 texture2D(纹理，纹理坐标），返回颜色值
    // 内建函数gl_FragColor：片元着色器处理完后的颜色值结果
    gl_FragColor = texture2D(colorMap, varyTextCoord);
}
```

glsl也有一个main方法，是程序的入口。GPU显存有的是顶点缓冲区、颜色缓冲区、深度缓存区等这样的内存空间，所以GPU并不能处理逻辑性比较复杂的数据，所以GLSL代码并不会很多。


链接：https://www.jianshu.com/p/5711720463c0

## 参考

* [GLSL（着色器语言） - 代码先锋网 (codeleading.com)](https://codeleading.com/article/49574021468/)

* [opengles着色语言中的限定符 - 代码先锋网 (codeleading.com)](https://codeleading.com/article/79134524551/)
* [opengles_jackyqiziheng的专栏-CSDN博客](https://blog.csdn.net/jackyqiziheng/category_6924503.html)
* [OpenGL_阿飞的博客-CSDN博客](https://blog.csdn.net/afei__/category_8502759.html)
* [OpenGL ES 3.0实践_handy周-CSDN博客](https://blog.csdn.net/byhook/category_9273761.html)

