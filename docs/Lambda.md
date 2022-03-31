## 内部类、Lambda表达式、方法引用



### 1、内部类




```java
public class TestLambda01 {
    
    
    interface MyLambda01{
   		void Lambda(int a);
	}
    
	/*
	* 1.内部类
	* **/
	class MyLambda01Impl1 implements MyLambda01{
		@Override
    	public void Lambda(int a) {
        	System.out.println("Lambda01..."+"a="+a);
    	}
    
    /*
    * 2.静态内部类
    * **/
    static class MyLambda01Impl02 implements MyLambda01{
        @Override
        public void Lambda(int a) {
            System.out.println("Lambda02..."+"a="+a);
        }
    }

    public static void main(String[] args) {
        /*
         * 3.局部内部类
         * */
        class MyLambda01Impl03 implements MyLambda01{
            @Override
            public void Lambda(int a) {
                System.out.println("Lambda03..."+"a="+a);
            }
        }

        /*
         * 4.匿名内部类
         * */
        MyLambda01 myLambdaImpl04 = new MyLambda01() {
            @Override
            public void Lambda(int a) {
                System.out.println("Lambda04..."+"a="+a);
            }
        };


        MyLambda01 myLambdaImpl01 = new MyLambda01Impl1();
        myLambdaImpl01.Lambda(1);

        MyLambda01Impl02 myLambdaImpl02 = new MyLambda01Impl02();
        myLambdaImpl02.Lambda(2);

        MyLambda01Impl03 myLambdaImpl03 = new MyLambda01Impl03();
        myLambdaImpl03.Lambda(3);

        myLambdaImpl04.Lambda(4);
    }

}



}
```





### 2、Lambda表达式和方法引用



**Java Lambda 表达式语法**

Java Lambda 表达式的语法结构如下

```
parameter -> expression body
```



**有参数且只有一条语句时**

```java
(a,b) -> a + b
```



**只有一个参数时**

```java
a  -> a
```



**没有参数时**

```java
()  -> System.out.println("简单教程")
```



**有多条语句时**

```java
(a,b) -> {
    int c = a + b;
    System.out.println("简单教程")
}
```

针对这个 Java Lambda 表达式语法，有几个重要的特征需要说明

- **可选的参数类型声明** ： 无需声明参数的类型。编译器可以从参数的值推断出相同的值。
- **可选的参数周围的小括号  `()`** ： 如果只有一个参数，可以忽略参数周围的小括号。但如果有多个参数，则必须添加小括号。
- **可选的大括号 `{}`**  : 如果 Lambda 表达式只包含一条语句，那么可以省略大括号。但如果有多条语句，则必须添加大括号。
- **可选的 `return` 关键字** ： 如果 Lambda 表达式只有一条语句，那么编译器会自动 `return` 该语句最后的结果。但如果显式使用了 `return` 语句，则必须添加大括号 `{}` ，哪怕只有一条语句。



#### Java Lambda 表达式的原理

后面我们会讲到，Java 8 中的 Lambda 表达式其实是一个特殊的只有一个方法的类的实例。

这些类是 Java 8 内部已经定义好的，而且实现了 `java.lang.FunctionalInterface` 这个接口。

这个 `java.lang.FunctionalInterface` 接口是一种信息性注解类型，用于标识一个接口类型声明为函数接口（ functional interface ）。

从某些方面说，Java 8 的 Lambda 表达式是使用匿名内部类的语法创建了 `java.util.function` 包下相应签名的接口的或者其它自定义的只有一个方法的接口实例。

但是，实际上，Java 8 中的 Lambda 不仅仅是使用匿名内部类，还使用了 Java 8 接口的默认方法和一些其它的功能。这方面，有空我会写一篇文章。



**范例一： Java Lambda 表达式**

Lambda 比较常见的使用场景就是 `new Runnable` 匿名内部类的使用

**LambdaTester.java**

```java
public class LambdaTester{
    
    public static void main(String[] args){
        
        Runnable r = () -> System.out.println("你好，简单教程，你好，简单编程");
        Thread th = new Thread(r);
        th.start(); 
        
     }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
你好，简单教程，你好，简单编程
```



**范例二**

**LambdaTester.java**

```java
public class LambdaTester {

   public static void main(String args[])
   {
      LambdaTester tester = new LambdaTester();

      // 有声明参数类型
      MathOperation addition = (int a, int b) -> a + b;

      // 没有声明参数类型
      MathOperation subtraction = (a, b) -> a - b;

      // 使用 return 语句显式返回值需要添加大括号
      MathOperation multiplication = (int a, int b) -> { return a * b; };

      // 如果只有一条语句，那么可以省略大括号，Java 会返回表达式的值
      MathOperation division = (int a, int b) -> a / b;

      System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
      System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
      System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
      System.out.println("10 / 5 = " + tester.operate(10, 5, division));
   }

   interface MathOperation {
      int operation(int a, int b);
   }

   private int operate(int a, int b, MathOperation mathOperation) {
      return mathOperation.operation(a, b);
   }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
10 + 5 = 15
10 - 5 = 5
10 x 5 = 50
10 / 5 = 2
```

Java Lambda 表达式的缺点

不知道大家有没有从上面的表达式中看到一些端倪，好像，好像 Java 8 中的 Lambda 不能凭空出现。

Java Lambda 表达式最大的缺点，就是不能像其它语言的 Lambda 表达式一样凭空出现。

Java 中的 Lambda 表达式需要有一个函数接口声明作为模板。这个模板定义了 Lambda 表达式的参数类型和返回值类型。

例如下面的代码，我们先要声明一个函数接口类型，然后才能定义一个参数和返回值都一样的表达式

LambdaTester.java

```java
public class LambdaTester {

   // 先声明一个函数接口
   interface GreetingService {
      void sayMessage(String message);
   }

   public static void main(String args[]){
      LambdaTester tester = new LambdaTester();
      
      // 省略小括号
      GreetingService greetService1 = message -> System.out.println("你好，" + message);

      // 有小括号
      GreetingService greetService2 = (message) -> System.out.println("你好，" + message);

      greetService1.sayMessage("简单教程");
      greetService2.sayMessage("简单编程");
   }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
你好，简单教程
你好，简单编程
```



#### Lambda 表达式作用域 ( scope )

因为 Java 8 的 lambda 表达式其实是函数接口的内联实现，也就是匿名内部类，因此，可以引用任何外部的变量或者常量。

但是，lambda 对这些外部的变量是有要求的： 它们必须使用 `final` 修饰符修饰。

如果一个变量允许被第二次赋值，则 Lambda 表达式会抛出编译错误。

> 注意： 其实这条规则并不是非常严格执行的，普通变量也是可以的，只要，只要不进行第二次赋值就可以。
>
> 注意： 刚刚测试了下，其实只要不是当前作用域声明的变量，可以随意第二次赋值，也不会报错



**范例一**

Java 8 lambda 表达式使用外部 **final** 变量

**LambdaTester.java**

```java
public class LambdaTester{

   final static String salutation = "你好，";

   public static void main(String args[]){
      GreetingService greetService1 = message -> System.out.println(salutation + message);
      greetService1.sayMessage("简单教程");
   }

   interface GreetingService {
      void sayMessage(String message);
   }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
你好，简单教程
```



**范例二**

lambda 引用的普通的变量也是可以的，**只要这个变量没有第二次被赋值**，不管是任何地方。

**LambdaTester.java**

```java
public class LambdaTester{

   static String salutation = "你好，";

   public static void main(String args[]){
      GreetingService greetService1 = message -> System.out.println(salutation + message);
      greetService1.sayMessage("简单教程");
   }

   interface GreetingService {
      void sayMessage(String message);
   }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
你好，简单教程
```



**范例三**

如果 lambda 表达式引用的是当前作用域下的普通的变量，而该变量又在某个地方第二次被赋值，则会抛出一个编译错误

**LambdaTester.java**

```java
public class LambdaTester{
    
   public static void main(String args[]){
       
      String salutation = "你好，";

      GreetingService greetService1 = message -> System.out.println(salutation + message);
      greetService1.sayMessage("简单教程");
      salutation = "Hello，";
   }

   interface GreetingService {
      void sayMessage(String message);
   }
}
```

运行结果如下 

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
LambdaTester.java:8: 错误: 从lambda 表达式引用的本地变量必须是最终变量或实际上的最终变量
      System.out.println(salutation + message);
                         ^
1 个错误
```



**范例四**

如果 lambda 表达式引用的变量并**不是当前作用域下声明**的，也可以随意赋值，并不会报错

**LambdaTester.java**

```java
public class LambdaTester{
    
   static String salutation = "你好，";
    
   public static void main(String args[]){
       
      salutation = "Hello，";
       
      GreetingService greetService1 = message -> System.out.println(salutation + message);
      greetService1.sayMessage("简单教程");
      salutation = "你好，";
   }

   interface GreetingService {
      void sayMessage(String message);
   }
}
```

运行结果如下 

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
Hello，简单教程
```

**总结**

**Java lambda 表达式可以随意引用外部变量，但如果外部变量是在当前作用域声明的，则一定不可以进行第二次赋值，哪怕是在 lambda 语句之后。**



#### lambda 表达式语法

lambda 表达式语法非常易于阅读和理解

lambda 表达式语法看起来像

```
(Argument  part)  -> Body part 
```

例如

1. 如果某个方法没有参数，并打印信息，则可能像下面这样

   ```
   () -> System.out.println("Your message");
   ```

2. 如果某个方法接受两个参数，执行一些业务逻辑并返回值，则看起来可能像下面这样

   ```
   (int a, int b) ->  a+b;
   ```

   表达式会返回 `a+b` 的计算结果

3. 如果某个方法接受一个参数，并执行一些计算，则可能像下面这样

   ```
   (String s) ->  s + "Hello World";
   ```



#### Java 8 lambda 表达式的一些使用范例



##### 在 `Runnable` 中使用 lambda 表达式

下面这个范例中，我们将运行一个 Runnable 线程。

在 Java 8 之前，在 lambda 表达式出现前，要实现 Runnable 线程，我们需要定义一个实现 Runnable 接口的类来获取 Runnable 对象。



**LambdaTester.java**

```java
public class LambdaTester{
    
   public static void main(String args[])
   {
      Runnable r = new Runnable() {

         @Override
         public void run() {
            System.out.println("你好，简单教程，简单编程");
         }
      };

      Thread thread = new Thread(r);
      thread.run();
   }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
你好，简单教程，简单编程
```

现在，我们来看看如何使用 lambda 表达式实现相同的对象。



**LambdaTester.java**

```java
public class LambdaTester
{
   public static void main(String args[])
   {
      Runnable r = () -> System.out.println("你好，简单教程，简单编程");

      Thread thread = new Thread(r);
      thread.run();
   }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
你好，简单教程，简单编程
```

没有对比，就没有伤害，使用 lambda 表达式真的是简单明了，代码又少。



##### 在用户自定义的函数接口中使用 lambda 表达式

在下面这个范例中，我们将创建自己的函数接口，我们先用普通的方式来实现一下



**LambdaTester.java**

```java
public class LambdaTester
{
   public static void main(String args[])
   {
      Calculator cal = new Calculator() {
         @Override
         public int add(int n1, int n2 ) {
            return n1 + n2;
         }
      };

      System.out.println(cal.add(5,8));
   }

   public interface Calculator {

      public int add(int n1, int n2);    
   }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
13
```

匿名内部类实现已经很简洁了。但，如果你看了 lambda 的实现，就会发现，匿名内部类还是有点复杂的



**LambdaTester.java**

```java
public class LambdaTester{
    
   public static void main(String args[]){
       
      Calculator cal = (n1,n2) -> n1 + n2;
      System.out.println(cal.add(5,8));
   }

   public interface Calculator {
      public int add(int n1, int n2);    
   }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
13
```

代码少的有点不像话了.....



##### 在 `Comparator` 中使用 lambda 表达式

现在，我们来看看 `Comparator` 接口，了解下如何使用 lambda 表达式对包含字符串的集合进行排序。



**LambdaTester.java**

```java
import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
public class LambdaTester {

    public static void main(String[] args)
    {
        List<String> list = Arrays.asList("Ram","Shyam","Kabir");
        System.out.println("...排序之前...");
        for(String s : list){
            System.out.println(s);
        } 


        //定义一个 comparator
        Comparator<String> comp= (String s1, String s2) -> s1.compareTo(s2);
        Collections.sort(list, comp);

        System.out.println("...排序之后...");
        for(String s : list){
            System.out.println(s);
        }
    }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java && java LambdaTester
...排序之前...
Ram
Shyam
Kabir
...排序之后...
Kabir
Ram
Shyam
```



##### 使用 Lambda 表达式实现函数接口来迭代集合

Java 8 中引入了 `java.util.function` 包。这个包提供了 `java.util.function.Function` 接口，我们可以在这个接口的帮助下，使用 Lambda 表达式来迭代集合。

`java.util.function.Function` 只有一个方法 `apply()` ，这个方法可以在用户定义的函数中调用。我们可以使用 `Function` 接口实现一个自定义的打印方法

```java
public String customShow( Function<String,String> fun) {
    return fun.apply(this);
}
```

然后，我们就可以使用 Lambda 来实现这个接口了



**LambdaTester.java**

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;

public class LambdaTester {

    public static void main(String[] args){
        LambdaTester tester = new LambdaTester();
        tester.run();
    }

    public void run(){
        List<LambdaTester.Student> list = new ArrayList<LambdaTester.Students>();

        list.add(new LambdaTester.Student("Ram",20));
        list.add(new LambdaTester.Student("Shyam",22));
        list.add(new LambdaTester.Student("Kabir",18));
        for(Student st: list){
            System.out.println(st.customShow(s->s.getName()+": "+s.getAge()));
        }
    }

    public class Student{
        
        private String name;
        private int age;
        public Student(String name,int age){
            this.name = name;
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        public  String customShow(Function<Student,String> fun){
            return fun.apply(this);
        }
    }
}
```

运行结果如下

```
Ram: 20
Shyam: 22
Kabir: 18
```





### Java 8 方法引用

Java 8 中新增加了 **方法引用** 这个概念。 但，什么是方法引用呢 ？

我们先来看一个例子，下面这个范例演示了如何遍历字符串列表并进行一些操作



**LambdaTester.java**

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;

public class LambdaTester {

    public static void main(String[] args){
        LambdaTester tester = new LambdaTester();
        tester.run();
    }

    public void run(){
        List<String> list = Arrays.asList("Ram","Shyam","Kabir");

        // 输出
        for(String st: list){
            System.out.println(st);
        }

        // 转换为大写
        for(String st: list){
            upperAndPrint(st);
        }
    }

    public static void upperAndPrint(String s){
        System.out.println(s.toUpperCase());
    }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java  && java LambdaTester
Ram
Shyam
Kabir
RAM
SHYAM
KABIR
```

有简洁代码倾向的我们，看到那重复的 `for(String st: list)` 就会想如何能够直接把代码改的更简洁一些。

第一个想到的，肯定是使用 Java 8 新增的 lambda 表达式和 `forEach` 改造下



**LambdaTester.java**

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;

public class LambdaTester {

    public static void main(String[] args){
        LambdaTester tester = new LambdaTester();
        tester.run();
    }

    public void run(){
        List<String> list = Arrays.asList("Ram","Shyam","Kabir");

        // 输出
        list.forEach(item -> System.out.println(item));


        // 转换为大写
        list.forEach(item -> upperAndPrint(item));
    }

    public static void upperAndPrint(String s){
        System.out.println(s.toUpperCase());
    }
}
```



哇，使用 Lambda 表达式真的是简洁了不少，但是，还能更简单一点吗？

比如 `item -> System.out.println(item)` 这个表达式，其实就是调用 `System.out.println()` 方法，并把 `forEach` 迭代列表生成的唯一参数 `item` 传给它而已

比如 `item -> upperAndPrint(item)` 这个表达式，也是一样啊，就是把 `forEach` 生成的唯一参数 `item` 传给给 `upperAndPrint()` 方法而已。

既然这样，我们为什么不能直接传递方法名给 `forEach` 呢？ 

对吧，所以我们改改，改成如下这种方式



```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;

public class LambdaTester {

    public static void main(String[] args){
        LambdaTester tester = new LambdaTester();
        tester.run();
    }

    public void run(){
        List<String> list = Arrays.asList("Ram","Shyam","Kabir");

        // 输出
        list.forEach(System.out.println);

        // 转换为大写
        list.forEach(upperAndPrint);
    }

    public static void upperAndPrint(String s){
        System.out.println(s.toUpperCase());
    }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java  && java LambdaTester
LambdaTester.java:18: 错误: 找不到符号
        list.forEach(System.out.println);
                               ^
  符号:   变量 println
  位置: 类型为PrintStream的变量 out
LambdaTester.java:21: 错误: 找不到符号
        list.forEach(upperAndPrint);
                     ^
  符号:   变量 upperAndPrint
  位置: 类 LambdaTester
2 个错误
```

这在所有的 Java 版本中都会报错，但是自 Java 8 开始，还真是可以直接传递方法名的，只是我们使用的方式不对而已。

加入我们改成下面这样，就会正确了

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;

public class LambdaTester {

    public static void main(String[] args){
        LambdaTester tester = new LambdaTester();
        tester.run();
    }

    public void run(){
        List<String> list = Arrays.asList("Ram","Shyam","Kabir");

        // 输出
        list.forEach(System.out::println);

        // 转换为大写
        list.forEach(LambdaTester::upperAndPrint);
    }

    public static void upperAndPrint(String s){
        System.out.println(s.toUpperCase());
    }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java  && java LambdaTester
Ram
Shyam
Kabir
RAM
SHYAM
KABIR
```

哈哈哈，正确了。

在这个正确的方法中，有两个重点：

1. **方法名和类名之间不是使用点号 ( `.` ) 而是使用两个冒号 `::` 。**
2. **提供的方法名必须包含类名，如果没有引入该类名，则需要使用全限定类名，也就是需要添加包名作为前缀。**

**这种使用方法名做参数的做法，在 Java 8 中称之为 「 方法引用 」**



**Java 8 方法引用**

**方法引用** 是 Java 8 新增加的功能。方法引用有点类似于 [C / C++](https://www.twle.cn/l/yufei/cprogramming/cprogramming-basic-index.html) 中的 **函数指针** ，通过方法名称指向方法。

Java 8 中的方法引用通过 `::` 符号引用方法，而且支持一下类型的方法引用

1. 静态方法
2. 实例方法
3. 使用 `new` 运算符的构造函数。例如 `TreeSet::new`



**范例**

我们重写一下上面的范例，演示下如何引用静态方法和实例方法

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;

public class LambdaTester {

    public static void main(String[] args){
        LambdaTester tester = new LambdaTester();
        tester.run();
    }

    public void run(){
        List<String> list = Arrays.asList("Ram","Shyam","Kabir");

        // 输出
        list.forEach(System.out::println);

        // 转换为大写
        list.forEach(LambdaTester::upperAndPrint);

        // 转换为小写并输出
        list.forEach(this::lowerAndPrint);
    }

    public void lowerAndPrint(String s){
        System.out.println(s.toLowerCase());
    }

    public static void upperAndPrint(String s){
        System.out.println(s.toUpperCase());
    }
}
```

运行结果如下

```
[yufei@www.twle.cn helloworld]$ javac LambdaTester.java  && java LambdaTester
Ram
Shyam
Kabir
RAM
SHYAM
KABIR
ram
shyam
kabir
```

