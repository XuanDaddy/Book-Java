# 【三】Lambda表达式

### 概述

* Lambda表达式在其它语言中也称为箭头函数、匿名函数、闭包等；
* Lambda表达式体现的是轻量级函数式编程思想；
* `->`符号是Lambda表达式核心操作符，符号左侧是**操作参数**，符号右侧是**操作表达式**；
* Lambda表达式不是解决未知问题的新技术；
* Lambda表达式是对现有解决方法的语义优化；

在Lambda表达式中，也只能操作一个方法，所以它的核心就是一个**函数式接口**实现。

如之前的函数式接口中，我们是通过匿名类实现，现在我们可以通过Lambda表达式来优化：

```java
//参数判断是否等于123
Predicate<String> predicate = s -> s.equals("123");

//将参数和123拼接后打印输出
Consumer<String> consumer = s -> System.out.println(s + "123");

//通过参数num乘以100后，转换为字符串
Function<Integer, String> function = num -> (num * 100) + "";

//通过UUID创建字符串的工厂
Supplier<String> supplier = () -> UUID.randomUUID().toString();

//传入字符串s，返回一个字符串
UnaryOperator<String> operator = s -> s + "hello";

//传入两个整数，返回一个整数
BinaryOperator<Integer> binaryOperator = (num1, num2) -> num1 * num2;
```

### Lambda语法

![image-20200106142626539](./images/lambda.png)

#### 基本语法

1. **声明**：Lambda绑定的接口类型；
2. **参数**：包含在一对圆括号中，和绑定接口中的抽象方法参数个数及顺序一致；
3. **操作符**：`->`；
4. **执行代码块**：包含在一对大括号中，出现在操作符号右侧；

如：`[接口声明] = （参数） -> {代码执行块}`

#### 无参无返回值Lambda

```java
interface Lambda1{
    void test();
}

//调用
Lambda1 lambda1 = () -> {
    System.out.println("No param");
    System.out.println("No result");
};
lambda1.test();
```

**当`执行代码块`只有一行代码时，可省略`大括号`**：

```java
//调用
Lambda1 lambda1 = () -> {
    System.out.println("No param,No result");
};
lambda1.test();
```

#### 带参数无返回值Lambda

```java
interface Lambda2 {

    void test(String name, int age);
}

//调用
Lambda2 lambda2 = (String name, int age) -> {
    System.out.println("name is " + name + ",age is " + age);
};
lambda2.test("hello", 11);
```

**参数类型不用明确指定，jvm会根据接口推导出参数类型**：

```java
//调用
Lambda2 lambda2 = (name, age) -> {
    System.out.println("name is " + name + ",age is " + age);
};
lambda2.test("hello", 11);
```

#### 带参数带返回值的Lambda

```java
interface Lambda3 {

    String test(String name, int age);
}

//调用
Lambda3 lambda3 =(name, age) -> {
    String temp = "name is " + name + ",age is " + age;
    return temp;
};
lambda3.test("hello",22);
```

**当执行代码只有一行，并不使用`大括号`的情况下，`return` 可省略**：

```java
//调用
Lambda3 lambda3 =(name, age) -> "name is " + name + ",age is " + age;
lambda3.test("hello",22);
```

#### 语法小结

1. Lambda表达式必须和接口进行绑定；
2. Lambda表达式可以附带0到n个参数，括号中的参数类型可以不用指定，jvm在运行时会自动根据绑定的抽象方法进行推导；
3. Lambda表达式的返回值，如果代码块只有一行，并且没有大括号，不用写`return`关键字，单行代码的执行结果会自动返回；如果添加了大括号或者有多行代码，必须通过`return`关键字返回执行结果；

### 变量捕获

#### 匿名内部类中的变量捕获

* 匿名内部类中不能通过`this`访问全局变量；
* 局部变量不允许修改（final）；

```java
String s1 = "全局变量";

public void test() {
    String s2 = "局部变量";
    new Thread(new Runnable() {
        String s3 = "内部变量";

        @Override
        public void run() {
            //访问全局变量
            System.out.println(s1);
//          System.out.println(this.s1);//不能使用this指定全局变量

            //访问局部变量
            System.out.println(s2);
//          s2 = "123";//局部变量不能被修改

            //访问内部变量
            System.out.println(this.s3);
        }
    }).start();
}
```



#### Lambda中的变量捕获

* Lambda中可以通过`this`访问当前方法的对象实例；
* 局部变量不允许修改（final）；

```java
public void testLambda() {
    String s2 = "局部变量Lambda";
     new Thread(() -> {
        String s3 = "内部变量Lambda";
        System.out.println(this.s1);
        System.out.println(s2);
        System.out.println(s3);
    }).start();
}
```

### Lambda底层原理

* Lambda表达式在JVM底层解析成私有静态方法和匿名内部类型；
* 通过实现接口的匿名内部类型中的接口方法调用静态方法，完成Lambda表达式的执行；

```java
public class App{

	public static void main(String[] args){

		ITest test = (msg) -> System.out.println(msg);
		test.test("Hello World");
	}
}

interface ITest{
	void test(String msg);
}
```

1. 通过`javac App.java`得到`App.class`和`ITest.class`;

2. 通过`javap -p App`反编译查看`App.class`，多出一个静态方法`lambda$main$0`，它的实现就是调用了上面的方法`System.out.println(msg)`

   ```java
   ➜  ~ javap -p App
   Compiled from "App.java"
   public class App {
     public App();
     public static void main(java.lang.String[]);
     private static void lambda$main$0(java.lang.String);
   }
   ```

3. 通过`java -Djdk.internal.lambda.dumpProxyClasses App`获取内部类实现，App中生成一个内部类；

   ```java
   final class App$$Lambda$1 implements ITest {
       private App$$Lambda$1() {
       }
   
       @Hidden
       public void test(String var1) {
           App.lambda$main$0(var1);
       }
   }
   ```

4. App中通过实例化内部类，调用方法实现

   ```java
   new App$$Lambda$1().test("Hello World");
   ```

   