# LambdaUtil
[![GitHub](https://img.shields.io/badge/license-MIT-green.svg)](http://opensource.org/licenses/MIT)
[![GitHub last commit](https://img.shields.io/github/last-commit/Robot-L/LambdaUtil?label=Last%20commit)]()
[![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/Robot-L/LambdaUtil)]()

用Stream API写Lambda表达式的时候遇到异常怎么办，试试这个！   
只需一个wrap方法就能将异常冒泡抛给外层，告别lambda表达式中的try-catch，最简单优雅的处理方式。

## 使用说明

### 1. lambda方法声明

```java
// 编译不通过，未处理异常“MalformedURLException”：
Function<String, URL> function = URL::new;
// 使用LambdaUtil后：
Function<String, URL> function = wrapFunction(URL::new);
```
### 2. 看一个更常见的例子

假设在Stream的map()方法中调用了一个会抛出异常的方法，此处将一批url字符串转换成URL对象模拟这种场景：
#### 原代码：

```java
public static void main(String[] args){
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
}
```
上面代码中`new URL(url)`会抛出`MalformedURLException`，在lambda表达式中必须被try-catch，无法向上抛出，这样不仅代码累赘，而且在实际开发中，绝大多数的异常都是需要向上抛出的，这样就无法简便的使用Stream API了。

#### 使用LambdaUtil之后

```java
import com.robot.LambdaUtil;

...
public static void main(String[] args) throws MalformedURLException{
    List<String> source = Arrays.asList("http://example1.com","http://example2.com","http://example3.com");

    // 只需要在原来的lambda表达式外用wrapFunction()方法包裹一下即可，注意异常已经被抛到了上层，main方法签名中增加了MalformedURLException异常申明
    List<URL> urlList = source.stream()
        .map(LambdaUtil.wrapFunction(url->new URL(url)))
        .collect(Collectors.toList());

    // 还可以使用method refrence 了，代码更加简洁！
    List<URL> urlList1 = source.stream()
    .map(LambdaUtil.wrapFunction(URL::new))
    .collect(Collectors.toList());
}
```
建议使用import static（静态导入），能将方法前的类名也省略，达到最终的极简形式：
```java
// 此处静态导入方法
import static com.robot.LambdaUtil.wrapFunction;

...
public static void main(String[] args) throws MalformedURLException{
    List<String> source = Arrays.asList("http://example1.com","http://example2.com","http://example3.com");
    // 通过静态导入省略了类名后：
    List<URL> urlList = source.stream().map(wrapFunction(URL::new)).collect(Collectors.toList());
}
```

## API

```java
// 最常用的4个，聪明的你一眼就能看懂怎么用吧😉
wrapFunction(Function);// Function：普通函数（入参出参各一个）
wrapConsumer(Consumer);// Consumer：消费函数（一个入参，没有出参）
wrapSupplier(Supplier);// Supplier：提供函数（没有入参，一个出参）
wrapPredicate(Predicate);// Predicate：条件函数（一个入参，一个出参，且出参类型是boolean）
// 简单来说就是，原先的lambda表达式是什么类型的函数，就用这种函数对应的wrap方法就好了

// more
wrapBiFunction(BiFunction);
wrapBiConsumer(BiConsumer);
wrapBiPredicate(BiPredicate);
wrapRunnable(Runnable);

```

## Tips

如果你使用IDEA的话，可以在代码中直接敲`wrapFunction(...)`，然后按`⌥+↩︎`(Opition+回车，Windows是Alt+回车)，选择弹出菜单中的“import static...”即可快速导入方法，其他API同理。如下图所示：
![快捷静态导入](https://tva1.sinaimg.cn/large/006y8mN6gy1g7xqme3telj31l00a8q6c.jpg)



## One more thing

工具类中还提供了一个很好用的方法，`uncheck`：
```java
如果已知一个方法绝不会抛出所申明的异常，可以使用该方法进行包装
如：new String(byteArr, "UTF-8")申明了UnsupportedEncodingException，
但编码"UTF-8"是必定不会抛异常的，所以可以使用uncheck()进行包装

String text = uncheck(() -> new String(byteArr, "UTF-8"));

// 通过class创建对象，确保实例化不会产生异常
Object something = uncheck(someClass::newInstance);

// 反射获取某个类的属性，已知这个类必然含有该属性
Field fieldFrom = uncheck(() -> someClass.getDeclaredField("memberValues"));
```
是不是很赞~😉  
> uncheck方法有一定的风险，因为它隐藏了可能的异常申明，导致外层不知道存在某异常也无法对该异常进行处理，所以请在使用该方法前三思，谨慎使用
 

## Note
本工具实现方式是利用泛型的不确定性使得编译器无法区分抛出的异常是uncheck-exception还是check-exception，利用这个漏洞绕开了编译器的检查，所以不会编译报错，然后将异常抛到外层（此时异常还是原来的异常），这和很多解决方案中的把异常转换成RuntimeException抛出的原理是不同的，有兴趣的朋友可以参看源码。

思路源自[@MarcG](https://stackoverflow.com/users/3411681/marcg)与[@PaoloC](https://stackoverflow.com/users/2365724/paoloc)，感谢两位大神。

如果发现问题或建议请提[Issues](https://github.com/Robot-L/LambdaUtil/issues)，如果对你有帮助，请点个Star，谢谢~ ^_^


参考：

- [Java 8 Lambda function that throws exception?](https://stackoverflow.com/questions/18198176/java-8-lambda-function-that-throws-exception)
- [How can I throw CHECKED exceptions from inside Java 8 streams?](https://stackoverflow.com/questions/27644361/how-can-i-throw-checked-exceptions-from-inside-java-8-streams)
