## 语法格式

```
(parameters) -> expression;
(parameters) -> { statements; }
```
## 重要特征
1. **可选类型声明**：不需要声明参数类型，编译器可以统一识别参数值。
1. **可选的参数圆括号**：一个参数无需定义圆括号，但多个参数需要定义圆括号。
1. **可选的大括号**：如果主体包含了一个语句，就不需要大括号。
1. **可选的返回关键字**：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指明表达式返回了一个数值。

## 变量作用域
- lambda表达式只能引用标记final的外层局部变量，不能再lambda内部修改定义在域外的局部变量，否则会编译错误。
- lambda表达式的局部变量可以不用声明为final，但是必须不可被后面的代码修改。
- 不允许声明一个与局部变量名同名的参数或者局部变量。


## 过滤

```
List<CourseLessonExt> Taglist=courseLessonExtList.stream()
    .filter(CourseLessonExt->(CourseLessonExt.getType()==3))
    .collect(Collectors.toList());
```

## 示例
```
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester tester = new Java8Tester();
        
      // 类型声明
      MathOperation addition = (int a, int b) -> a + b;
        
      // 不用类型声明
      MathOperation subtraction = (a, b) -> a - b;
        
      // 大括号中的返回语句
      MathOperation multiplication = (int a, int b) -> { return a * b; };
        
      // 没有大括号及返回语句
      MathOperation division = (int a, int b) -> a / b;
        
      System.out.println("10 + 5 = " + tester.operate(10, 5, addition));
      System.out.println("10 - 5 = " + tester.operate(10, 5, subtraction));
      System.out.println("10 x 5 = " + tester.operate(10, 5, multiplication));
      System.out.println("10 / 5 = " + tester.operate(10, 5, division));
        
      // 不用括号
      GreetingService greetService1 = message ->
      System.out.println("Hello " + message);
        
      // 用括号
      GreetingService greetService2 = (message) ->
      System.out.println("Hello " + message);
        
      greetService1.sayMessage("Runoob");
      greetService2.sayMessage("Google");
   }
    
   interface MathOperation {
      int operation(int a, int b);
   }
    
   interface GreetingService {
      void sayMessage(String message);
   }
    
   private int operate(int a, int b, MathOperation mathOperation){
      return mathOperation.operation(a, b);
   }
}
```

输出结果：

```
$ javac Java8Tester.java 
$ java Java8Tester
10 + 5 = 15
10 - 5 = 5
10 x 5 = 50
10 / 5 = 2
Hello Runoob
Hello Google
```

----------

# 确认lamda表达式的类型

能用lamda表达式来表示的类型，必须是一个函数式接口，而函数式接口就是只有一个抽象方法的接口。例如Runnable接口在JDK中的样子，这就是一个标准的抽象方法，因为只有一个抽象方法。而且接口上有个注解@FunctionalInterface，这个仅仅是在编译期检查接口是否符合函数式接口的条件，比如没有任何抽象方法，或者有多个抽象方法，编译是无法通过的。
```java
@FunctionalInterface
public interface Runnable{
   public abstract void run();
}
```
lamda表达式需要的类型为函数式接口，函数式接口里只有一个抽象方法。

# 找到要实现的方法

lamda表达式就是实现一个方法，就是刚刚那些函数式接口中的抽象方法。

# 实现这个方法

lamda表达式就是要实现这个抽象方法，如果不用lamda表达式，就用匿名类去实现，比如我们实现Predicate接口的匿名类
```Java
Predicate<String> predicate = new Predicate<String>() {
    @Override
    public boolean test(String s) {
        return s.length() != 0;
    }
};
```
如果换成lamda表达式，如下
```Java
Predicate<String> predicate = 
    (String s) -> {
        return s.length() ！= 0;
    };
```
lamda语法由三部分组成：
- 参数块：简单地把要实现的抽象方法的参数原封不动写在这
- 小箭头：-> 这个符号
- 代码块：要实现的方法原封不动写在这

参数块部分，(String s)里面的类型信息是多余的，因为完全可以由编译器推导，去掉它。
```Java
Predicate<String> predicate = 
    (s) -> {
        return s.length() ！= 0;
    };
```
当只有一个参数时，括号也可以去掉
```Java
Predicate<String> predicate = 
    s -> {
        return s.length() ！= 0;
    };
```
再看代码块部分，方法体中只有一行代码，可以把花括号和return关键字都去掉
```Java
Predicate<String> predicate = s -> s.length() ！= 0;
```
示例：lamda表达式实现Runnable接口：
```Java
Runnable r = () -> System.out.println("I am running");
new Thread(()->System.out.println("test lamda")).start;
```

# 多个入参&带有返回值

示例：
```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}

BiFunction<String, String, Integer> findWordInSentence = 
    (word, sentence) -> sentence.indexOf(word);
```

其实函数式接口里那个抽象方法，无非就是入参的个数，以及返回值的类型。入参的个数可以是一个或者两个，返回值可以是void，或者Boolean，或者一个类型。那这些种情况的排列组合，就是JDK给我们提供的Java.util.function包下的类。前缀是Int、Long、Double之类的，是指定了入参的特定类型，而不再是一个可以由用户自定义的泛型，比如DoubleFunction。如下完全可以由更自由的函数式接口Function来实现。

```Java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

简单分类：
- supplier：没有入参，有返回值
- consumer：有入参，无返回值
- predicate：有入参，返回Boolean值
- function：有入参，有返回值

带Bi前缀的，有两个入参；不带就只有一个入参。