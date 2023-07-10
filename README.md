# LambdaExceptionThrower

[![GitHub](https://img.shields.io/github/license/luo-zhan/Transformer)](http://opensource.org/licenses/apache-2-0)
[![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/luo-zhan/LambdaExceptionThrower)]()
[![GitHub last commit](https://img.shields.io/github/last-commit/luo-zhan/LambdaExceptionThrower?label=Last%20commit)]()

Java里写Lambda表达式的时候居然必须要try-catch掉内部异常，简直不能忍！😠

## 使用说明

### 1. lambda方法声明

```java
// 编译不通过，未处理异常“MalformedURLException”
Function<String, URL> function=URL::new;
// 使用LambdaExceptionThrower后，编译通过！
        Function<String, URL> function=wrapFunction(URL::new);
```
### 2. 看一个更常见的例子

如果我们在`Stream`的`map()`方法中使用一个会抛出异常的方法：（此处将一批url字符串转换成URL对象模拟这种场景）

#### Before：

```java
class Test {

    public static void main(String[] args) {
        List<String> source = Arrays.asList("http://example1.com", "http://example2.com", "http://example3.com");
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
}
```
上面代码中`new URL(url)`会抛出`MalformedURLException`，在lambda表达式中必须被try-catch，无法向上抛出，这样不仅代码累赘，而且在实际开发中，绝大多数的异常都是需要向上抛出的，这样就无法简便的使用Stream API了。

#### After

```java
// 此处静态导入方法

import static com.robot.util.LambdaExceptionUtil.wrapFunction;

class Test {
    public static void main(String[] args) throws MalformedURLException { // 注意这里增加了异常申明
        List<String> source = Arrays.asList("http://example1.com", "http://example2.com", "http://example3.com");

        // 只需要在原来的lambda表达式外用wrapFunction()方法包裹一下即可，注意异常已经被抛到了上层
        List<URL> urlList = source.stream()
                .map(wrapFunction(url -> new URL(url)))
                .collect(Collectors.toList());

        // 还可以使用方法引入，代码更加简洁！
        List<URL> urlList1 = source.stream()
                .map(wrapFunction(URL::new))
                .collect(Collectors.toList());
    }
}
```

#### Tips

1.使用IDEA写代码时可以直接敲`wrapFunction(...)`，然后按`⌥+↩︎`(`Opition+回车`，Windows是`Alt+回车`)，选择弹出菜单中的“import
static...”即可快速导入方法
![快捷静态导入](https://msb-edu-dev.oss-cn-beijing.aliyuncs.com/course/lambda2.png)

2.使用`wrapFunction`方法包裹后，idea是会提示未捕获方法异常的，只需要点击一下就自动在方法上申明了异常，和平时写代码异常处理操作无异
![快捷申明异常](https://msb-edu-dev.oss-cn-beijing.aliyuncs.com/course/lambda.png)

## One more thing

工具类中还提供了另一个很好用的方法，`sure()`。

如果一段代码**确保**不会抛出所申明的异常，可以使用该方法进行包装，此时既不用写try-catch，也不用在方法申明上throw异常，代码相当简洁。

```java
import static com.robot.util.LambdaExceptionUtil.sure;

class Test {
    // 如下String的构造方法申明了UnsupportedEncodingException，但编码"UTF-8"是必定不会抛异常的，使用sure(...)进行包装
    String text = sure(() -> new String(byteArr, "UTF-8"));

    // 通过class创建对象，而已知此实例化一定不会产生异常
    Map map = sure(HashMap.class::newInstance);

    // 反射获取某个类的属性，而已知这个类必然含有该属性
    Field field = sure(() -> someClass.getDeclaredField("id"));

    // 测试时用的sleep方法，不必改变测试方法上的异常申明
    sure(() ->Thread.sleep(1000));
}
```

> 注意：sure方法虽然好用，但它也隐藏了可能的异常申明，所以请谨慎使用，确定(sure)一定不会抛出异常！

## API

有手就会😉

```js
// 最常用的4个
wrapFunction(Function);// Function：普通函数（入参出参各一个）
wrapConsumer(Consumer);// Consumer：消费函数（一个入参，没有出参）
wrapSupplier(Supplier);// Supplier：提供函数（没有入参，一个出参）
wrapPredicate(Predicate);// Predicate：条件函数（一个入参，一个出参，且出参类型是boolean）

// 更多，和jdk8中的函数式接口一一对应
wrapBiFunction(BiFunction);
wrapBiConsumer(BiConsumer);
wrapBiPredicate(BiPredicate);
wrapRunnable(Runnable);

// 确保不抛出异常时使用
sure(Runnable);
sure(Supplier);

```

## Note

本工具实现方式是利用泛型的不确定性使得编译器无法区分抛出的异常是uncheck-exception还是check-exception，利用这个漏洞绕开了编译器的检查，所以不会编译报错，然后将异常抛到外层（此时异常还是原来的异常），这和很多解决方案中的把异常转换成RuntimeException抛出的原理是不同的，有兴趣的朋友可以参看源码。

思路源自[@MarcG](https://stackoverflow.com/users/3411681/marcg)
与[@PaoloC](https://stackoverflow.com/users/2365724/paoloc)，感谢两位大神。

如果发现问题或建议请提[Issues](https://github.com/Robot-L/LambdaExceptionThrower/issues)
，如果对你有帮助，请点个Star，非常感谢~ ^_^

扩展阅读：

- [Java 8 Lambda function that throws exception?](https://stackoverflow.com/questions/18198176/java-8-lambda-function-that-throws-exception)
- [How can I throw CHECKED exceptions from inside Java 8 streams?](https://stackoverflow.com/questions/27644361/how-can-i-throw-checked-exceptions-from-inside-java-8-streams)
