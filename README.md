# LambdaExceptionUtil
写Lambda表达式不想要捕获异常怎么办，试试这个！
一个类告别lambda表达式中的try-catch语句，最简单的解决方式。

``
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
            // 抛异常只能抛unchecked-exception(RuntimeException或Error)，或者处理掉异常不往上抛
            throw new RuntimeException(e);
        }
    }).collect(Collectors.toList());
```
上面代码中`new URL(url)`会抛出MalformedURLException，在lambda表达式中必须被try-catch，无法向上抛出，这样不仅代码累赘，而且在实际开发中绝大多数的都是需要将异常向上抛出的。

### 使用LambdaExceptionUtil
```java
    import com.asiainfo.res.core.util.LambdaExceptionUtil;
   
    ...

    List<String> source = Arrays.asList("http://example1.com","http://example2.com","http://example3.com");
    
    List<URL> urlList = source.stream()
        .map(LambdaExceptionUtil.wrapFunction(url->new URL(url)))
        .collect(Collectors.toList());
        
    // 因为处理了异常，还可以使用method refrence使代码更简洁！
    List<URL> urlList1 = source.stream()
    .map(LambdaExceptionUtil.wrapFunction(URL::new))
    .collect(Collectors.toList());
```
如果使用import static（静态导入），能将方法前的类名也省略，使得代码更加简洁：
```java
    
    // 此处静态导入方法
    import static com.robot.LambdaExceptionUtil.wrapFunction;
    
    ...

    List<String> source = Arrays.asList("http://example1.com","http://example2.com","http://example3.com");
    // 省略了类名
    List<URL> urlList = source.stream().map(wrapFunction(URL::new)).collect(Collectors.toList());
```

# API
```java
// 最常用的4个，聪明的你一眼就能看懂怎么用吧
wrapFunction(function);// function： 入参出参各一个
wrapConsumer(consumer);// consumer：一个入参，没有出参
wrapSupplier(supplier);// supplier：没有入参，一个出参
wrapPredicate(predicate);// predicate：一个入参，一个出参，出参类型是boolean

// lambda表达式中有两个入参的：
wrapBiFunction(biFunction);
wrapBiConsumer(biConsumer);
wrapBiPredicate(biPredicate);


```

参考：[Java 8 Lambda function that throws exception?](https://stackoverflow.com/questions/18198176/java-8-lambda-function-that-throws-exception)
