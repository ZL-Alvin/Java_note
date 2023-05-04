# 第一种用法（静态）

```java
package com.bupt;

import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

public class thymeleaf_1 {
    public static void main(String[] args) {
        //创建模板引擎
        TemplateEngine engine = new TemplateEngine();
        //准备模板
        String input = "<input> type='text' th:value='hello!'/>";
        //准备数据
        Context context = new Context();
        //调用引擎，处理模板和数据
        String out = engine.process(input, context);
        System.out.println("结果数据：" + out);
    }
}
```

# 第二种用法（动态）

```java
@Test
public void test01() {
    //创建模板引擎
    TemplateEngine engine = new TemplateEngine();
    //准备模板
    String inStr = "<input type='text' th:value='${name}'/>";
    //准备数据
    Context context = new Context();
    context.setVariable("name", "李四");
    String html = engine.process(inStr, context);
    System.out.println(html);
}
```

# 第三种使用模板文件（动态）

在resources文件夹下创建html模板文件

![image-20230322154603350](C:\Users\Alvin\AppData\Roaming\Typora\typora-user-images\image-20230322154603350.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    模板：<input type="text" th:value="${name}"/>
</body>
</html>
```

```java
@Test
public void test02() {
    //创建模板引擎
    TemplateEngine engine = new TemplateEngine();
    //读取磁盘中的模板文件,模板解析器
    ClassLoaderTemplateResolver resolver = new ClassLoaderTemplateResolver();
    //设置引擎使用resolver
    engine.setTemplateResolver(resolver);
    //指定数据
    Context context = new Context();
    context.setVariable("name", "张三");
    //处理模板
    String html = engine.process("main.html", context);
    System.out.println(html);
}
```

## 设置模板前缀，后缀

举例：在resources文件夹下面新建一个templates文件夹，用于存放模板文件。

​			在templates文件夹下新建模板文件index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Title</title>
</head>
<body>
  模板：<input type="text" th:value="${name}"/>
</body>
</html>
```

```java
@Test
public void test03() {
    //创建模板引擎
    TemplateEngine engine = new TemplateEngine();
    //读取磁盘中的模板文件,模板解析器
    ClassLoaderTemplateResolver resolver = new ClassLoaderTemplateResolver();
    //设置前缀，相对于resources文件夹下，模板文件地址
    resolver.setPrefix("templates/");
    //设置后缀，模板文件后缀
    resolver.setSuffix(".html");
    //设置引擎使用resolver
    engine.setTemplateResolver(resolver);
    //指定数据
    Context context = new Context();
    context.setVariable("name", "张三");
    //处理模板
    String html = engine.process("index", context);
    System.out.println(html);
}
```