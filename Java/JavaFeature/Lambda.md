# Java8 Lambda

来源：[Java8 Lambda](https://www.cnblogs.com/qdhxhz/p/9393724.html)

## 一、概述

#### 1、什么是Lambda表达式

Lambda 表达式是一种匿名函数，简单地说，它是没有声明的方法，也即没有访问修饰符、返回值声明和名字。

它可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使 Java 语言的表达能力得到了提升。

#### 2、**Lambda表达式的语法**

基本语法: (parameters) -> expression

   或者：(parameters) ->{ statements; }

举例说明：

```java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s) 
```

#### 3、什么是函数式接口

 再对上面进行举例说明之前，必须先来理解下函数式接口，因为Lambda是建立在函数式接口的基础上的。

 **记住！**

  （1）只包含一个抽象方法的接口，称为函数式接口。

 （2）你可以通过 Lambda 表达式来创建该接口的对象。

 （3）我们可以在任意函数式接口上使用 @FunctionalInterface 注解，这样做可以检测它是否是一个函数式接口，同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口。

在实际开发者有两个比较常见的函数式接口：**Runnable接口，Comparator接口**

  先举例**Runnable接口相关**

```java
public class Test {
    
    public static void main(String[] args) {
        
        // 1.1使用匿名内部类  
        new Thread(new Runnable() {  
            @Override  
            public void run() {  
                System.out.println("Hello world !");  
            }  
        }).start();  
          
        // 1.2使用 lambda 获得Runnable接口对象  
        new Thread(() -> System.out.println("Hello world !")).start();  
        
//=============================================================================
        
        // 2.1使用匿名内部类  
        Runnable race1 = new Runnable() {  
            @Override  
            public void run() {  
                System.out.println("Hello world !");  
            }  
        };  
          
        // 2.2使用 lambda直接获得接口对象 
        Runnable race2 = () -> System.out.println("Hello world !");          
        
        // 直接调用 run 方法(没开新线程哦!)  
        race1.run();  
        race2.run();  
    }
}
/*输出结果
 * Hello world !
 * Hello world !
 * Hello world !
 * Hello world !
 *／
```

通过上面案例可以看出：通过Lambda表达式看去舒服清爽多了，而通过匿名内部类代码总是不够整洁。

再举一个例子：**使用Lambda对数组排序**

```java
public class TestArray {
    
    public static void main(String[] args) {
        String[] players = {"zhansgan", "lisi", "wangwu", "zhaoliu",  "wangmazi"};  

        // 1.1 使用匿名内部类根据 surname 排序 players  
        Arrays.sort(players, new Comparator<String>() {  
            @Override  
            public int compare(String s1, String s2) {  
                return (s1.compareTo(s2));  
            }  
        });  
        
        // 1.2 使用 lambda 排序,根据 surname  
        Arrays.sort(players, (String s1, String s2) ->  s1.compareTo(s2));  
         
//==========================================================================================
          
        // 2.1 使用匿名内部类根据 name lenght 排序 players  
        Arrays.sort(players, new Comparator<String>() {  
            @Override  
            public int compare(String s1, String s2) {  
                return (s1.length() - s2.length());  
            }  
        });  

        // 2.2使用Lambda,根据name length  
        Arrays.sort(players, (String s1, String s2) -> (s1.length() - s2.length()));  
    
//========================================================================================== 
        
        // 3.1 使用匿名内部类排序 players, 根据最后一个字母  
        Arrays.sort(players, new Comparator<String>() {  
            @Override  
            public int compare(String s1, String s2) {  
                return (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1));  
            }  
        });  

        // 3.2 使用Lambda,根据最后一个字母
        Arrays.sort(players, (String s1, String s2) -> (s1.charAt(s1.length() - 1) - s2.charAt(s2.length() - 1)));  
    }
}
```

通过上面例子我们再来思考为什么Lambda表达式需要函数式接口？其实很简单目的就是为来保证唯一。

**你的**Runnable接口只要一个抽象方法，那么我用() -> System.out.println("Hello world !")，就只能代表run方法，如果你下面还有一个抽象方法，那我使用Lambda表达式，

那鬼才知道要调用哪个抽象方法呢。

## **二、方法引用**

#### **1、基本介绍**

**首先注意：**方法引用，不是方法调用！方法引用，不是方法调用！方法引用，不是方法调用！

函数式接口的实例可以通过 lambda 表达式、 方法引用、构造方法引用来创建。方法引用是 lambda 表达式的语法糖，任何用方法引用的地方都可由lambda表达式替换，

但是并不是所有的lambda表达式都可以用方法引用来替换。

举例

这就是一个打印集合所有元素的例子，value -> System.out.println(value) 是一个Consumer函数式接口， 这个函数式接口可以通过方法引用来替换。

```java
public class TestArray {
    
    public static void main(String[] args) {
         List<String> list = Arrays.asList("xuxiaoxiao", "xudada", "xuzhongzhong");
         list.forEach(value -> System.out.println(value));
    }
    /* 输出：
     * xuxiaoxiao
     * xudada
     * xuzhongzhong
     */
}
```

使用方法引用的方式，和上面的输出是一样的，方法引用使用的是双冒号（::）

```java
list.forEach(System.out::println);
```

#### 2、分类

| 类别         | 使用形式                     |
| ------------ | ---------------------------- |
| 静态方法引用 | 类名 :: 静态方法名           |
| 实例方法引用 | 对象名(引用名) :: 实例方法名 |
| 类方法引用   | 类名 :: 实例方法名           |
| 构造方法引用 | 类名 :: new                  |

**（1）静态方法引用**

```java
public class Apple {

    private String name;
    private String color;
    private double weight;

    public Apple(String name, String color, double weight) {
        this.name = name;
        this.color = color;
        this.weight = weight;
    }

    public static int compareByWeight(Apple a1, Apple a2) {
        double diff = a1.getWeight() - a2.getWeight();
        return new Double(diff).intValue();
    }

    //还有getter setter toString
}    

```

有一个苹果的List，现在需要根据苹果的重量进行排序。List 的 sort 函数接收一个 Comparator 类型的参数，Comparator 是一个函数式接口，接收两个参数，

返回一个int值。Apple的静态方法compareByWeight正好符合Comparator函数式接口，所以可以使用：

Apple::compareByWeight 静态方法引用来替代lambda表达式

```java
public class LambdaTest {

    public static void main(String[] args) {

        Apple apple1 = new Apple("红富士", "Red", 280);
        Apple apple2 = new Apple("冯心", "Yello", 470);
        Apple apple3 = new Apple("大牛", "Red", 320);
        Apple apple4 = new Apple("小小", "Green", 300);

        List<Apple> appleList = Arrays.asList(apple1, apple2, apple3, apple4);

        //lambda 表达式形式
        //appleList.sort((Apple a1, Apple a2) -> {
        //    return new Double(a1.getWeight() - a2.getWeight()).intValue();
        //});

        //静态方法引用形式（可以看出引用方法比上面的更加简单
        appleList.sort(Apple::compareByWeight);

        appleList.forEach(apple -> System.out.println(apple));

    }
}
输出：
Apple{category='红富士', color='Red', weight=280.0}
Apple{category='小小', color='Green', weight=300.0}
Apple{category='大牛', color='Red', weight=320.0}
Apple{category='冯心', color='Yello', weight=470.0}
```

**注意：**Apple.compareByWeight是方法的调用，而Apple::compareByWeight方法引用，这两者完全不是一回事。

**（2）实例方法引用**

这个compareByWeight是一个实例方法

```java
public class AppleComparator {

    public int compareByWeight(Apple a1, Apple a2) {
        double diff = a1.getWeight() - a2.getWeight();
        return new Double(diff).intValue();
    }
}
```

下面的例子通过实例对象的方法引用 comparator::compareByWeight 来代替lambda表达式

```java
public class LambdaTest {

    public static void main(String[] args) {

        Apple apple1 = new Apple("红富士", "Red", 280);
        Apple apple2 = new Apple("冯心", "Yello", 470);
        Apple apple3 = new Apple("哈哈", "Red", 320);
        Apple apple4 = new Apple("小小", "Green", 300);


        List<Apple> appleList = Arrays.asList(apple1, apple2, apple3, apple4);

        //lambda 表达式形式
        //appleList.sort((Apple a1, Apple a2) -> {
        //    return new Double(a1.getWeight() - a2.getWeight()).intValue();
        //});

        //实例方法引用
        AppleComparator comparator = new AppleComparator();
        appleList.sort(comparator::compareByWeight);

        appleList.forEach(apple -> System.out.println(apple));

    }
}
输出：
Apple{category='红富士', color='Red', weight=280.0}
Apple{category='小小', color='Green', weight=300.0}
Apple{category='哈哈', color='Red', weight=320.0}
Apple{category='冯心', color='Yello', weight=470.0}
```

通过上面两个例子可以看到，静态方法引用和实例方法引用都是比较好理解的。

**（3）类方法引用**

一般来说，同类型对象的比较，应该当前调用方法的对象与另外一个对象进行比较，好的设计应该像下面： 

```java
public class Apple {

    private String category;
    private String color;
    private double weight;

    public Apple(String category, String color, double weight) {
        this.category = category;
        this.color = color;
        this.weight = weight;
    }
//这里和上面静态方式唯一区别就是这个参数就一个，需要实例对象调这个方法
    public int compareByWeight(Apple other) {
        double diff = this.getWeight() - other.getWeight();
        return new Double(diff).intValue();
    }

    //getter setter toString
}
```

 还是之前List排序的例子，看看使用类方法引用如何写：

```java
public class LambdaTest {

    public static void main(String[] args) {

        Apple apple1 = new Apple("红富士", "Red", 280);
        Apple apple2 = new Apple("黄元帅", "Yello", 470);
        Apple apple3 = new Apple("红将军", "Red", 320);
        Apple apple4 = new Apple("国光", "Green", 300);


        List<Apple> appleList = Arrays.asList(apple1, apple2, apple3, apple4);

        //lambda 表达式形式
        //appleList.sort((Apple a1, Apple a2) -> {
        //    return new Double(a1.getWeight() - a2.getWeight()).intValue();
        //});

        //这里是类方法引用
        appleList.sort(Apple::compareByWeight);

        appleList.forEach(apple -> System.out.println(apple));

    }
}
输出：
Apple{category='红富士', color='Red', weight=280.0}
Apple{category='国光', color='Green', weight=300.0}
Apple{category='红将军', color='Red', weight=320.0}
Apple{category='黄元帅', color='Yello', weight=470.0}
```

   这里使用的是：类名::实例方法名。首先要说明的是，方法引用不是方法调用。compareByWeight一定是某个实例调用的，就是lambda表达式的第一个参数，然后lambda

表达式剩下的参数作为 compareByWeight的参数，这样compareByWeight正好符合lambda表达式的定义。

或者也可以这样理解：

(Apple a1, Apple a2) -> { return new Double(a1.getWeight() - a2.getWeight()).intValue(); }

int compareByWeight(Apple other) 需要当前对象调用，然后与另外一个对象比较，并且返回一个int值。可以理解为lambda表达式的第一个参数 a1 赋值给当前对象， 然后 a2

赋值给 other对象，然后返回int值。

**（4）构造方法引用**

```java
public class ConstructionMethodTest {

    public String getString(Supplier<String> supplier) {
        return supplier.get();
    }

    public static void main(String[] args) {

        ConstructionMethodTest test = new ConstructionMethodTest();

        //lambda表达式形式
        System.out.println(test.getString(() -> { return new String();}));

        //构造方法引用形式
        System.out.println(test.getString(String::new));

    }
}
```

getString 方法接收一个Supplier类型的参数，Supplier 不接收参数，返回一个String。lambda表达式应该这样写：

```java
() -> { return new String();}
```

替换成方法引用的形式如下： 实际上调用的是String 无参构造方法。

```java
String::new
```