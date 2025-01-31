---
title: 函数式接口
comment: true
tags:
  - JAVA
  - Lambda
categories:
  - [JAVA]
---

函数式接口（Functional Interface）是 Java 8 引入的一个重要概念，在 Java 语言的函数式编程中扮演着关键角色。

### 定义
函数式接口是指仅包含一个抽象方法的接口。虽然接口中可以有默认方法、静态方法，但抽象方法只能有一个。为了保证接口符合函数式接口的定义，Java 提供了 `@FunctionalInterface` 注解，使用该注解标记的接口会在编译时由编译器检查是否满足函数式接口的条件，不过这个注解并不是必需的。

### 示例代码
```java
// 使用 @FunctionalInterface 注解标记为函数式接口
@FunctionalInterface
interface MyFunctionalInterface {
    // 唯一的抽象方法
    void doSomething();

    // 默认方法
    default void defaultMethod() {
        System.out.println("这是一个默认方法");
    }

    // 静态方法
    static void staticMethod() {
        System.out.println("这是一个静态方法");
    }
}
```

### 特点
- **单一抽象方法**：这是函数式接口的核心特征，确保接口具有明确的功能语义，通常代表一个单一的行为。
- **可使用 Lambda 表达式**：由于函数式接口只有一个抽象方法，因此可以使用 Lambda 表达式来创建该接口的实例，使得代码更加简洁和灵活。
- **支持默认方法和静态方法**：函数式接口中可以包含默认方法和静态方法，这些方法有具体的实现，不会影响接口作为函数式接口的性质。

### 用途
函数式接口主要用于支持 Java 的函数式编程，以下是一些常见的使用场景：
- **Lambda 表达式的目标类型**：Lambda 表达式本质上是函数式接口的一个实例，因此函数式接口为 Lambda 表达式提供了类型。
```java
public class Main {
    public static void main(String[] args) {
        // 使用 Lambda 表达式实现 MyFunctionalInterface
        MyFunctionalInterface myFunction = () -> System.out.println("执行操作");
        myFunction.doSomething();
    }
}
```
- **方法引用**：方法引用也是基于函数式接口的，它是 Lambda 表达式的一种更简洁的形式。
```java
import java.util.Arrays;
import java.util.List;

@FunctionalInterface
interface Printable {
    void print(String str);
}

public class MethodReferenceExample {
    public static void printMessage(String str) {
        System.out.println(str);
    }

    public static void main(String[] args) {
        List<String> list = Arrays.asList("apple", "banana", "cherry");
        // 使用方法引用实现 Printable 接口
        Printable printer = MethodReferenceExample::printMessage;
        list.forEach(printer::print);
    }
}
```
- **Stream API**：Java 8 引入的 Stream API 大量使用了函数式接口，如 `Predicate`、`Function`、`Consumer` 等，用于对集合进行过滤、映射、排序等操作。
```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class StreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        // 使用 Stream API 和 Lambda 表达式过滤偶数
        List<Integer> evenNumbers = numbers.stream()
                .filter(n -> n % 2 == 0)
                .collect(Collectors.toList());
        System.out.println(evenNumbers);
    }
}
```

### 内置函数式接口
Java 8 在 `java.util.function` 包中提供了一系列内置的函数式接口，以满足不同的编程需求，常见的内置函数式接口如下：
- **`Predicate<T>`**：表示一个布尔值函数，接受一个类型为 `T` 的参数，返回一个布尔值。常用于过滤操作。
```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;

public class PredicateExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        Predicate<Integer> isEven = n -> n % 2 == 0;
        numbers.stream()
               .filter(isEven)
               .forEach(System.out::println);
    }
}
```
- **`Function<T, R>`**：表示一个函数，接受一个类型为 `T` 的参数，返回一个类型为 `R` 的结果。常用于映射操作。
```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;
import java.util.stream.Collectors;

public class FunctionExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
        Function<Integer, Integer> square = n -> n * n;
        List<Integer> squaredNumbers = numbers.stream()
                .map(square)
                .collect(Collectors.toList());
        System.out.println(squaredNumbers);
    }
}
```
- **`Consumer<T>`**：表示一个消费者，接受一个类型为 `T` 的参数，不返回任何结果。常用于对元素进行操作。
```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Consumer;

public class ConsumerExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        Consumer<String> printName = name -> System.out.println(name);
        names.forEach(printName);
    }
}
```
- **`Supplier<T>`**：表示一个供应商，不接受任何参数，返回一个类型为 `T` 的结果。常用于创建对象。
```java
import java.util.function.Supplier;

class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

public class SupplierExample {
    public static void main(String[] args) {
        Supplier<Person> personSupplier = () -> new Person("John");
        Person person = personSupplier.get();
        System.out.println(person.getName());
    }
}
```