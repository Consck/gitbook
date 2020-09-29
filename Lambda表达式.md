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

