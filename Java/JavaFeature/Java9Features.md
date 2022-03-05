<h1 align="center">Java 9 Features</h1>

Java 9 is a major release and it has brought us a lot of features for developers. In this article, we will look into Java 9 features in detail.

Java 10 has been released, for a complete overview of Java 10 release, go through Java 10 Features.

## Java 9 Features



Some of the important java 9 features are;

1. [Java 9 REPL (JShell)](https://www.journaldev.com/13121/java-9-features-with-examples#repl)
2. [Factory Methods for Immutable List, Set, Map and Map.Entry](https://www.journaldev.com/13121/java-9-features-with-examples#factory-methods-immutable)
3. [Private methods in Interfaces](https://www.journaldev.com/13121/java-9-features-with-examples#private-methods)
4. [Java 9 Module System](https://www.journaldev.com/13121/java-9-features-with-examples#module-system)
5. [Process API Improvements](https://www.journaldev.com/13121/java-9-features-with-examples#process-api)
6. [Try With Resources Improvement](https://www.journaldev.com/13121/java-9-features-with-examples#try-with-resources)
7. [CompletableFuture API Improvements](https://www.journaldev.com/13121/java-9-features-with-examples#completablefuture)
8. [Reactive Streams](https://www.journaldev.com/13121/java-9-features-with-examples#reactive-streams)
9. [Diamond Operator for Anonymous Inner Class](https://www.journaldev.com/13121/java-9-features-with-examples#diamond-operator)
10. [Optional Class Improvements](https://www.journaldev.com/13121/java-9-features-with-examples#optional)
11. [Stream API Improvements](https://www.journaldev.com/13121/java-9-features-with-examples#stream-api)
12. [Enhanced @Deprecated annotation](https://www.journaldev.com/13121/java-9-features-with-examples#deprecated)
13. [HTTP 2 Client](https://www.journaldev.com/13121/java-9-features-with-examples#http2-client)
14. [Multi-Resolution Image API](https://www.journaldev.com/13121/java-9-features-with-examples#image-api)
15. [Miscellaneous Java 9 Features](https://www.journaldev.com/13121/java-9-features-with-examples#java-9-core)

Oracle Corporation is going to release Java SE 9 around the end of March 2017. In this post, I’m going to discuss “Java 9 Features” briefly with some examples.


1. ### Java 9 REPL (JShell)

   Oracle Corp has introduced a new tool called “jshell”. It stands for Java Shell and also known as REPL (Read Evaluate Print Loop). It is used to execute and test any Java Constructs like class, interface, enum, object, statements etc. very easily.

   We can download JDK 9 EA (Early Access) software from https://jdk9.java.net/download/

   ```
   G:\>jshell
   |  Welcome to JShell -- Version 9-ea
   |  For an introduction type: /help intro
   
   
   jshell> int a = 10
   a ==> 10
   
   jshell> System.out.println("a value = " + a )
   a value = 10
   ```

   If you want to know more about REPL tool, Please go through [Java 9 REPL Basics (Part-1)](https://www.journaldev.com/9879/java-repl-jshell) and [Java 9 REPL Features (Part-2)](https://www.journaldev.com/12938/jshell-java-shell).


2. ### Factory Methods for Immutable List, Set, Map and Map.Entry

   Oracle Corp has introduced some convenient factory methods to create [Immutable](https://www.journaldev.com/129/how-to-create-immutable-class-in-java) List, Set, Map and Map.Entry objects. These utility methods are used to create empty or non-empty Collection objects.

   In Java SE 8 and earlier versions, We can use Collections class utility methods like `unmodifiableXXX` to create Immutable Collection objects. For instance, if we want to create an Immutable List, then we can use `Collections.unmodifiableList` method.

   However, these `Collections.unmodifiableXXX` methods are a tedious and verbose approach. To overcome those shortcomings, Oracle Corp has added a couple of utility methods to List, Set and Map interfaces.

   List and Set interfaces have “of()” methods to create an empty or no-empty Immutable List or Set objects as shown below:

   **Empty List Example**

   ```
   List immutableList = List.of();
   ```

   **Non-Empty List Example**

   ```
   List immutableList = List.of("one","two","three");
   ```

   The Map has two sets of methods: `of()` methods and `ofEntries()` methods to create an Immutable Map object and an Immutable Map.Entry object respectively.

   **Empty Map Example**

   ```
   jshell> Map emptyImmutableMap = Map.of()
   emptyImmutableMap ==> {}
   ```

   **Non-Empty Map Example**

   ```
   jshell> Map nonemptyImmutableMap = Map.of(1, "one", 2, "two", 3, "three")
   nonemptyImmutableMap ==> {2=two, 3=three, 1=one}
   ```

   If you want to read more about these utility methods, please go through the following links:

   - [Java 9 Factory Methods for Immutable List](https://www.journaldev.com/12942/javase9-factory-methods-immutable-list)
   - [Java 9 Factory Methods for Immutable Set](https://www.journaldev.com/12984/javase9-factories-for-immutable-set)
   - [Java 9 Factory Methods for Immutable Map and Map.Entry](https://www.journaldev.com/13057/javase9-factory-methods-immutable-map)


3. ### Private methods in Interfaces

   In [Java 8](https://www.journaldev.com/2389/java-8-features-with-examples), we can provide method implementation in Interfaces using Default and Static methods. However we cannot create private methods in Interfaces.

   To avoid redundant code and more re-usability, Oracle Corp is going to introduce private methods in Java SE 9 Interfaces. From Java SE 9 onwards, we can write private and private static methods too in an interface using a ‘private’ keyword.

   These private methods are like other class private methods only, there is no difference between them.

   ```
   public interface Card{
   
     private Long createCardID(){
       // Method implementation goes here.
     }
   
     private static void displayCardDetails(){
       // Method implementation goes here.
     }
   
   }
   ```

   If you want to read more about this new feature, please go through this link: [Java 9 Private methods in Interface](https://www.journaldev.com/12850/java-9-private-methods-interfaces).


4. ### Java 9 Module System

   One of the big changes or java 9 feature is the Module System. Oracle Corp is going to introduce the following features as part of **Jigsaw Project**.

   - Modular JDK
   - Modular Java Source Code
   - Modular Run-time Images
   - Encapsulate Java Internal APIs
   - Java Platform Module System

   Before Java SE 9 versions, we are using Monolithic Jars to develop Java-Based applications. This architecture has a lot of limitations and drawbacks. To avoid all these shortcomings, Java SE 9 is coming with the Module System.

   JDK 9 is coming with 92 modules (may change in final release). We can use JDK Modules and also we can create our own modules as shown below:

   **Simple Module Example**

   ```
   module com.foo.bar { }
   ```

   Here we are using ‘module’ to create a simple module. Each module has a name, related code, and other resources.

   To read more details about this new architecture and hands-on experience, please go through my original tutorials here:

   - [Java 9 Module System Basics](https://www.journaldev.com/13106/java-9-modules)
   - [Java 9 Module Examples using command prompt](https://www.journaldev.com/13543/javase9-simple-module-cmd-prompt-part3)
   - [Java 9 Hello World Module Example using Eclipse IDE](https://www.journaldev.com/13630/javase9-helloworld-module-ides-part4)


5. ### Process API Improvements

    Java SE 9 is coming with some improvements in Process API. They have added couple new classes and methods to ease the controlling and managing of OS processes.

    Two new interfcase in Process API:

    - java.lang.ProcessHandle
    - java.lang.ProcessHandle.Info

    **Process API example**

    ```
     ProcessHandle currentProcess = ProcessHandle.current();
     System.out.println("Current Process Id: = " + currentProcess.getPid());
    ```


6. ### Try With Resources Improvement

    We know, Java SE 7 has introduced a new exception handling construct: Try-With-Resources to manage resources automatically. The main goal of this new statement is “Automatic Better Resource Management”.

    Java SE 9 is going to provide some improvements to this statement to avoid some more verbosity and improve some Readability.

    **Java SE 7 example**

    ```
    void testARM_Before_Java9() throws IOException{
     BufferedReader reader1 = new BufferedReader(new FileReader("journaldev.txt"));
     try (BufferedReader reader2 = reader1) {
       System.out.println(reader2.readLine());
     }
    }
    ```

    **Java 9 example**

    ```
    void testARM_Java9() throws IOException{
     BufferedReader reader1 = new BufferedReader(new FileReader("journaldev.txt"));
     try (reader1) {
       System.out.println(reader1.readLine());
     }
    }
    ```

    To read more about this new feature, please go through my original tutorial at: [Java 9 Try-With-Resources Improvements](https://www.journaldev.com/12940/javase9-try-with-resources-improvements)


7. ### CompletableFuture API Improvements

    In Java SE 9, Oracle Corp is going to improve CompletableFuture API to solve some problems raised in Java SE 8. They are going add to support some delays and timeouts, some utility methods and better sub-classing.

    ```
    Executor exe = CompletableFuture.delayedExecutor(50L, TimeUnit.SECONDS);
    ```

    Here delayedExecutor() is a static utility method used to return a new Executor that submits a task to the default executor after the given delay.


8. ### Reactive Streams

    Nowadays, Reactive Programming has become very popular in developing applications to get some beautiful benefits. Scala, Play, Akka, etc. Frameworks have already integrated Reactive Streams and getting many benefits. Oracle Corps is also introducing new Reactive Streams API in Java SE 9.

    Java SE 9 Reactive Streams API is a Publish/Subscribe Framework to implement Asynchronous, Scalable and Parallel applications very easily using Java language.

    Java SE 9 has introduced the following API to develop Reactive Streams in Java-based applications.

    - java.util.concurrent.Flow
    - java.util.concurrent.Flow.Publisher
    - java.util.concurrent.Flow.Subscriber
    - java.util.concurrent.Flow.Processor

    Read more at [Java 9 Reactive Streams](https://www.journaldev.com/20723/java-9-reactive-streams).


9. ### Diamond Operator for Anonymous Inner Class

    We know, Java SE 7 has introduced one new feature: Diamond Operator to avoid redundant code and verbosity, to improve readability. However, in Java SE 8, Oracle Corp (Java Library Developer) has found that some limitations in the use of Diamond operator with Anonymous Inner Class. They have fixed those issues and going to release them as part of Java 9.

    ```
      public List getEmployee(String empid){
         // Code to get Employee details from Data Store
         return new List(emp){ };
      }
    ```

    Here we are using just “List” without specifying the type parameter.


10. ### Optional Class Improvements

    In Java SE 9, Oracle Corp has added some useful new methods to java.util.Optional class. Here I’m going to discuss about one of those methods with some simple example: stream method

    If a value present in the given Optional object, this stream() method returns a sequential Stream with that value. Otherwise, it returns an empty Stream.

    They have added “stream()” method to work on Optional objects lazily as shown below:

    ```
    Stream<Optional> emp = getEmployee(id)
    Stream empStream = emp.flatMap(Optional::stream)
    ```

    Here Optional.stream() method is used to convert a Stream of Optional of Employee object into a Stream of Employee so that we can work on this result lazily in the result code.

    To understand more about this feature with more examples and to read more new methods added to Optional class, please go through my original tutorial at: **[Java SE 9: Optional Class Improvements](https://www.journaldev.com/13108/javase9-optional-class-improvements)**



11. ### Stream API Improvements

    In Java SE 9, Oracle Corp has added four useful new methods to java.util.Stream interface. As Stream is an interface, all those new implemented methods are default methods. Two of them are very important: dropWhile and takeWhile methods

    If you are familiar with Scala Language or any Functions programming language, you will definitely know about these methods. These are very useful methods in writing some functional style code. Let us discuss the takeWhile utility method here.

    This takeWhile() takes a predicate as an argument and returns a Stream of the subset of the given Stream values until that Predicate returns false for the first time. If the first value does NOT satisfy that Predicate, it just returns an empty Stream.

    ```
    jshell> Stream.of(1,2,3,4,5,6,7,8,9,10).takeWhile(i -> i < 5 )
                     .forEach(System.out::println);
    1
    2
    3
    4
    ```

    To read more about takeWhile and dropWhile methods and other new methods, please go through my original tutorial at: **[Java SE 9: Stream API Improvements](https://www.journaldev.com/13204/javase9-stream-api-improvements)**


12. ### Enhanced @Deprecated annotation

    In Java SE 8 and earlier versions, @Deprecated annotation is just a Marker interface without any methods. It is used to mark a Java API that is a class, field, method, interface, constructor, enum etc.

    In Java SE 9, Oracle Corp has enhanced @Deprecated annotation to provide more information about deprecated API and also provide a **Tool** to analyze an application’s static usage of deprecated APIs. They have add two methods to this Deprecated interface: **forRemoval** and **since** to serve this information.



13. ### HTTP 2 Client

    In Java SE 9, Oracle Corp is going to release New HTTP 2 Client API to support HTTP/2 protocol and WebSocket features. As existing or Legacy HTTP Client API has numerous issues (like supports HTTP/1.1 protocol and does not support HTTP/2 protocol and WebSocket, works only in Blocking mode and lot of performance issues.), they are replacing this HttpURLConnection API with new HTTP client.

    They are going to introduce a new HTTP 2 Client API under the “java.net.http” package. It supports both HTTP/1.1 and HTTP/2 protocols. It supports both Synchronous (Blocking Mode) and Asynchronous Modes. It supports Asynchronous Mode using the WebSocket API.

    We can see this new API at https://download.java.net/java/jdk9/docs/api/java/net/http/package-summary.html

    **HTTP 2 Client Example**

    ```
    jshell> import java.net.http.*
    
    jshell> import static java.net.http.HttpRequest.*
    
    jshell> import static java.net.http.HttpResponse.*
    
    jshell> URI uri = new URI("https://rams4java.blogspot.co.uk/2016/05/java-news.html")
    uri ==> https://rams4java.blogspot.co.uk/2016/05/java-news.html
    
    jshell> HttpResponse response = HttpRequest.create(uri).body(noBody()).GET().response()
    response ==> java.net.http.HttpResponseImpl@79efed2d
    
    jshell> System.out.println("Response was " + response.body(asString()))
    ```

    Please go through my original tutorial at: **Java SE 9: HTTP 2 Client** to understand HTTP/2 protocol & WebSocket, Benefits of new API and Drawbacks of OLD API with some useful examples.



14. ### Multi-Resolution Image API

    In Java SE 9, Oracle Corp is going to introduce a new Multi-Resolution Image API. Important interface in this API is MultiResolutionImage . It is available in java.awt.image package.

    MultiResolutionImage encapsulates a set of images with different Height and Widths (that is different resolutions) and allows us to query them with our requirements.



15. ### Miscellaneous Java 9 Features

    In this section, I will just list out some miscellaneous Java SE 9 New Features. I’m NOT saying these are less important features. They are also important and useful to understand them very well with some useful examples.

    As of now, I did not get enough information about these features. That’s why I am going to list them here for a brief understanding. I will pick up these features one by one and add to the above section with a brief discussion and example. And finally write a separate tutorial later.

    - GC (Garbage Collector) Improvements
    - Stack-Walking API
    - Filter Incoming Serialization Data
    - Deprecate the Applet API
    - Indify String Concatenation
    - Enhanced Method Handles
    - Java Platform Logging API and Service
    - Compact Strings
    - Parser API for Nashorn
    - Javadoc Search
    - HTML5 Javadoc

I will pickup these java 9 features one by one and update them with enough description and examples.

That’s all about Java 9 features in brief with examples.

source: https://www.journaldev.com/13121/java-9-features-with-examples