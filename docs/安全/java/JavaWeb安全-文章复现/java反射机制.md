---
created: '21/07/01'
title: java反射机制
tags:
  - java
  - java安全
---
# java反射机制
通过java反射我们可以实现动态地获取以下东西：
- 类
- 实例
- 方法，调用方法
- 属性，属性值
- ...

## 获取Class对象
有三种方法获取Class对象：
1.  `类名.class`
2.  `Class.forName("")`
3.  `classLoader.loadClass("");`

获取数组类型的Class对象比较特殊：
```java
Class<?> doubleArray = Class.forName("[D");//相当于double[].class
Class<?> cStringArray = Class.forName("[[Ljava.lang.String;");// 相当于String[][].class
```

## 反射获取java.lang.Runtime
```java
package top.longlone;


import sun.misc.IOUtils;

import java.io.InputStream;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;

public class ReflectionStudy {
    public static void main(String[] args) throws Exception {
        // 反射获取Runtime类
        Class<?> runtimeClass = Class.forName("java.lang.Runtime");

        // 反射获取类的构造器，因为它是私有的所以要设置Accessible
        Constructor<?> constructor = runtimeClass.getDeclaredConstructor();
        constructor.setAccessible(true);

        // 获取Runtime实例
        Object runtimeInstance = constructor.newInstance();

        // 获取exec方法
        Method exec = runtimeClass.getMethod("exec", String.class);

        // 反射调用exec方法并获取结果
        Process process = (Process) exec.invoke(runtimeInstance, "whoami");

        InputStream inputStream = process.getInputStream();

        System.out.println(new String(IOUtils.readAllBytes(inputStream)));

    }
}

```

## 反射调用类方法
### 获取成员方法
- 获取当前类所有的成员方法：
```java
Method[] methods = clazz.getDeclaredMethods()
```
- 获取当前类指定的成员方法：
```java
Method method = clazz.getDeclaredMethod("方法名");
Method method = clazz.getDeclaredMethod("方法名", 参数类型如String.class，多个参数用","号隔开);
```

### getMethod与getDeclaredMethod的区别
`getMethod`能获取**当前类和父类**的所有有权限方法（如public）
`getDeclaredMethod`能获取**当前类**的所有方法

### 反射调用类方法
```java
method.invoke(方法实例对象, 方法参数值，多个参数值用","隔开);
```

## 反射获取和修改属性
### 获取当前类的所有成员变量
```java
Field fields = clazz.getDeclaredFields();
```

### 获取当前类指定的成员变量

```java
Field field  = clazz.getDeclaredField("变量名");
```

`getField`和`getDeclaredField`的区别同`getMethod`和`getDeclaredMethod`

### 获取成员变量值

```java
Object obj = field.get(类实例对象);
```

### 修改成员变量值

```java
field.set(类实例对象, 修改后的值);
```

### 示例
```java
package top.longlone;


import java.lang.reflect.Constructor;
import java.lang.reflect.Field;

class NumberTest {
    private Integer number;

    public NumberTest() {
        number = 1;
    }

    public void display() {
        System.out.println("Number is " + number);
    }
}

public class ReflectionStudy {
    public static void main(String[] args) throws Exception {
        // 获取NumberTest类
        Class<?> testClass = Class.forName("top.longlone.NumberTest");

        // 获取NumberTest中的number属性并设置可访问（因为是私有的）
        Field numberField = testClass.getDeclaredField("number");
        numberField.setAccessible(true);

        // 获取NumberTest构造器
        Constructor<?> testConstructor = testClass.getConstructor();

        // 获取NumberTest实例
        NumberTest testInstance = (NumberTest) testConstructor.newInstance();

        // 获取实例中的number属性值，此时为1
        Integer number = (Integer) numberField.get(testInstance);

        // 设置实例中的number属性值为原来的+1
        numberField.set(testInstance, number + 1);

        // 输出结果，为2
        testInstance.display();

    }
}

```
