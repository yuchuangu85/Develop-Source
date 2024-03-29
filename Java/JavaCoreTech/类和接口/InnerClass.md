<h1 align="center">内部类</h1>

[toc]

## 内部类

- 非静态内部类没法在外部类的静态方法中实例化。
- 非静态内部类的方法可以直接访问外部类的所有数据，包括私有的数据。
- 在静态内部类中调用外部类成员，成员也要求用 static 修饰。
- 创建静态内部类的对象可以直接通过外部类调用静态内部类的构造器；创建非静态的内部类的对象必须先创建外部类的对象，通过外部类的对象调用内部类的构造器。

## 匿名内部类

- 匿名内部类不能定义任何静态成员、方法
- 匿名内部类中的方法不能是抽象的
- 匿名内部类必须实现接口或抽象父类的所有抽象方法
- 匿名内部类不能定义构造器
- 匿名内部类访问的外部类成员变量或成员方法必须用 final 修饰

