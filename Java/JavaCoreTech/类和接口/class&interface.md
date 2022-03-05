<h1 align="center">类与接口</h1>

[toc]



## 类 - 接口之间的关系

| 类和类之间的关系 | 使用 extends         单继承 |
| ---------------- | --------------------------- |
| 类和接口间的关系 | 使用 implements      多实现 |
| 接口和接口的关系 | 使用 extends         多继承 |



## 接口，抽象类区别

1. 抽象类用 abstract class 定义，接口用 interface 定义
2. 继承抽象类用 extends ，实现接口用 implements
3. 继承抽象类只能单继承，二实现接口可以多实现
4. 抽象类可以定义构造方法，接口不能
5. 抽象类可以定义成员变量，接口只能应以常量
6. 抽象类中可以有成员方法，接口中只能由抽象方法
7. 抽象类中的增加方法可以不影响子类，接口增加新方法大多数情况下会影响子类
8. Java 1.8 之后，接口中定义 default 修饰的成员方法可不影响子类
9. 抽象类不能有对象（不能用 new 关键字来创建抽象类的对象）
10. 抽象类中的抽象方法必须在子类中被重写
11. 接口中的所有属性默认为：public static final ****；
12. 接口中的所有方法默认为：public abstract ****；

## 抽象类要不要实现接口的方法？

- 抽象类可以实现接口的部分方法或者一个方法也不实现

## Object

### equals 方法

对两个对象的地址值进行的比较（即比较引用是否相同）

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

### hashCode 方法

hashCode() 方法给对象返回一个 hash code 值。这个方法被用于 hash tables，例如 HashMap。

它的性质是：

- 在一个Java应用的执行期间，如果一个对象提供给 equals 做比较的信息没有被修改的话，该对象多次调用 hashCode() 方法，该方法必须始终如一返回同一个 integer。

- 如果两个对象根据 equals(Object) 方法是相等的，那么调用二者各自的 hashCode() 方法必须产生同一个 integer 结果。

- 并不要求根据 equals(Object) 方法不相等的两个对象，调用二者各自的 hashCode() 方法必须产生不同的 integer 结果。然而，程序员应该意识到对于不同的对象产生不同的 integer 结果，有可能会提高 hash table 的性能。

在 JDK 中，Object 的 hashcode 方法是本地方法，也就是用 c 语言或 c++ 实现的，该方法直接返回对象的 内存地址。在 String 类，重写了 hashCode 方法

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
