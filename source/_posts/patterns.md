title: 范型方法的反模式
tags:
  - patterns
category: java
date: 2015-07-20 10:16:00

---
## 范型方法的反模式

[原文地址](http://it.deepinmind.com/java/2015/05/07/this-common-api-technique-is-actually-an-anti-pattern.html)
[英文地址](http://blog.jooq.org/2015/04/30/this-common-api-technique-is-actually-an-anti-pattern/)


我承认，我自己也忍不住用过这项技术。它简直太方便了，可以省去一次不必要的类型转化。这就是：

```

interface SomeWrapper {
  <T> T get();
}

```

现在你便可以安全地将包装类转换成任意类型了：

```

SomeWrapper wrapper = ...
 
// Obviously
Object a = wrapper.get();
 
// Well...
Number b = wrapper.get();
 
// Risky
String[][] c = wrapper.get();
 
// Unprobable
javax.persistence.SqlResultSetMapping d = 
    wrapper.get();
    
````
这也正是你在使用JOOR时所用到的API，jOOR是我们开发并开源的一个反射工具库，它可以提升我们编写集成测试的效率。有了jOOR，你可以这么编写代码：

```

Employee[] employees = on(department)
    .call("getEmployees").get();
  
for (Employee employee : employees) {
    Street street = on(employee)
        .call("getAddress")
        .call("getStreet")
        .get();
    System.out.println(street);
}

```
这个API非常简单。on()方法会对某个对象或类进行封装。call()方法会通过反射去调用这个对象上的方法（不需要签名正确，也不需要方法声明成public，同时也不会抛出任何的受检查异常）。同时你也不需要强制类型转换，便能通过get()方法将结果转换成任意的类型。

对于一款像jOOR这样的反射库而言，这么做是没问题的，因为这个库本身就不是类型安全的。当然不会安全了，因为本来就是反射。

不过这还是会让人感到有些不爽。感觉就是在调用点这里承诺会返回某个类型，但实际上却无法兑现，这很可能会导致ClassCastException异常——在有了Java 5以及泛型之后才开始写Java的开发人员是很难体会到这种感觉的

#### 但JDK也是这么做的。。
没错，JDK是这么干了。不过这样的情况并不多，并且只是在泛型参数的类型并不那么重要的情况下才会这么做。比如说，当你通过Collection.emptyList()获取一个空列表的时候，这个方法的实现是这样的

```

@SuppressWarnings("unchecked")
public static final <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}

```

没错，EMPTY_LIST从List强转成List是挺不安全的。但从语义层面来说，这样的强转是安全的。你不能修改这个列表，因为这是个空列表。List中也不会有任何一个方法会返回给你一个非目标类型的T或者T[]的实例。因此，这些做法都是合法的：

```

// perfectly fine
List<?> a = emptyList();
 
// yep
List<Object> b = emptyList();
 
// alright
List<Number> c = emptyList();
 
// no problem
List<String[][]> d = emptyList();
 
// if you must
List<javax.persistence.SqlResultSetMapping> e = emptyList();
    
```
JDK的开发人员从来都会非常小心地不去对你所可能获取到的泛型类型做出任何无法兑现的承诺。也就是说，即便存在一个更适合的类型的时候，通常返回给你的也都是Object类型。

哪个类型更适合只有你知道，编译器可不了解。类型擦除是有代价的，代价就是当包装类或者集合为空的时候。像这样的一个表达式编译器是无法得知它的类型的，它可不能不懂装懂。也就是说：

*不要使用这种只为了省一次类型转化的泛型方法的反模式。*
