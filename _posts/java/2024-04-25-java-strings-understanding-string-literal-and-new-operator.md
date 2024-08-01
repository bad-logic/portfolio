---
layout: post
title: 'Java Strings: Understanding string literal and new operator'
date: 2024-04-25 09:02:44 -0500
categories: posts java
tags: Java,string,stringpool
---

In java there are two ways of creating a string:

- using string literal `String str = "Hello world";`
- using new operator `String str = new String("Hello world")`

Lets dig deep into what happens when each of the above syntax is used to create a string.

String Template

```java
String str = "Hello World";
```

In java this is equivalent to

```java
String str = "Hello World".intern();
```

Let’s see what `intern()` method does

- on invoking this function it checks if the string content “Hello World” exists in the String pool.

![empty_string_pool](/assets/images/understanding_string_literals/string_pool_u4o2g1rolf5s1ls9afv5.avif)

if it does not find same string content in the pool, it will create a new String object in heap. Since we are using string literal, java will then add this string to the string pool for reusing and return the memory address of the string object. let's assume this memory address is `50`.

![adding_string_to_string_pool](/assets/images/understanding_string_literals/string_pool_uj6zrjrchw5pof78jfl9.avif)

Next time if we add another string with string literal and the content of the string is same, then java will return the memory address of the previous string object.

```java
String newStr = "Hello World";
```

![adding_new_string_with_template_literal](/assets/images/understanding_string_literals/string_pool_0fqwc2uskngpjsg9w24j.avif)

But if you create a string using the new operator, it will not use the string pool, instead it will create another string object in the heap. let's assume the new memory address is `100`.

```java
String thirdString = new String("Hello World");
```

![adding_string_with_new_operator](/assets/images/understanding_string_literals/string_pool_1xnoget90btavhd2wyx7.avif)

We can verify this with the `==` operator, since String checks the memory address if we try to confirm equality with `==` operator.

```java
String str = "Hello World".intern();
String newStr = "Hello World";
String thirdString = new String("Hello World");
System.out.println(str == newStr); // true
System.out.println(newStr == thirdString); // false
```

If you want to check if the string contents are same then you would need to use `equals` method available in string object.

```java
String str = "Hello World".intern();
String newStr = "Hello World";
String thirdString = new String("Hello World");
System.out.println(str.equals(newStr)); // true
System.out.println(newStr.equals(thirdString)); // true
```

> Why does java do this ?
> String is `immutable` Object, so everytime you perform some operation on string a new string object is created. and creating new string object everytime is very expensive, so java added this functionality to reuse the already existing string contents. But this is applicable only if you are using string literal to create string objects.
