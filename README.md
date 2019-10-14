# LambdaExceptionUtil
写Lambda表达式不想要捕获异常怎么办，试试这个！
一个wrap方法告别lambda表达式中的try-catch语句，最简单的解决方式。


# 使用说明

## 1. lambda方法声明
```java
// 编译不通过，未处理异常“MalformedURLException”：
Function<String, URL> function = URL::new;
// 使用LambdaExceptionUtil后：
Function<String, URL> function = wrapFunction(URL::new);
```
## 2. 看一个更常见的例子
在Stream的map()方法中调用了一个会抛出异常的方法，此处将一批url字符串转换成URL对象模拟这种场景：
### 原代码：
```java
List<String> source = Arrays.asList("http://example1.com","http://example2.com","http://example3.com");
List<URL> urlList = source.stream().map(url -> {
    // 必须要捕获异常，否则无法编译通过
    try {
        return new URL(url);
    } catch (MalformedURLException e) {
        // 抛异常只能抛unchecked-exception(RuntimeException或Error)，或者处理掉异常不往上抛。
        throw new RuntimeException(e);
    }
}).collect(Collectors.toList());
```
上面代码中`new URL(url)`会抛出MalformedURLException，在lambda表达式中必须被try-catch，无法向上抛出，这样不仅代码累赘，而且在实际开发中，绝大多数的异常都是需要向上抛出的，这样就无法简便的使用Stream API了。

### 使用LambdaExceptionUtil之后
```java
import com.robot.LambdaExceptionUtil;

...

List<String> source = Arrays.asList("http://example1.com","http://example2.com","http://example3.com");

// 只需要在原来的lambda表达式外用wrapFunction()方法包裹一下即可
List<URL> urlList = source.stream()
    .map(LambdaExceptionUtil.wrapFunction(url->new URL(url)))
    .collect(Collectors.toList());

// 还可以使用method refrence！
List<URL> urlList1 = source.stream()
.map(LambdaExceptionUtil.wrapFunction(URL::new))
.collect(Collectors.toList());
```
建议使用import static（静态导入），能将方法前的类名也省略，使得代码更加简洁：
```java
    
// 此处静态导入方法
import static com.robot.LambdaExceptionUtil.wrapFunction;

...

List<String> source = Arrays.asList("http://example1.com","http://example2.com","http://example3.com");
// 省略了类名
List<URL> urlList = source.stream().map(wrapFunction(URL::new)).collect(Collectors.toList());
```

# API
```
// 最常用的4个，聪明的你一眼就能看懂怎么用吧
wrapFunction(function);// function： 入参出参各一个
wrapConsumer(consumer);// consumer：一个入参，没有出参
wrapSupplier(supplier);// supplier：没有入参，一个出参
wrapPredicate(predicate);// predicate：一个入参，一个出参，出参类型是boolean

// more
wrapBiFunction(biFunction);
wrapBiConsumer(biConsumer);
wrapBiPredicate(biPredicate);

```

# Tips
如果你使用IDEA的话，可以在代码中直接敲`wrapFunction(...)`，然后按`⌥+↩︎`(Opition+回车，Windows是Alt+回车)，选择弹出菜单中的“import static...”即可快速导入方法，其他API同理。如下图所示：
![快捷静态导入](https://tva1.sinaimg.cn/large/006y8mN6gy1g7xqme3telj31l00a8q6c.jpg)


参考：
- [Java 8 Lambda function that throws exception?](https://stackoverflow.com/questions/18198176/java-8-lambda-function-that-throws-exception)
- [How can I throw CHECKED exceptions from inside Java 8 streams?](https://stackoverflow.com/questions/27644361/how-can-i-throw-checked-exceptions-from-inside-java-8-streams)
