---
title: Java中的hashcode和equals
date: 2020-12-24 00:18:34
tags: Java基础
categories: Java
---
<meta name="referrer" content="no-referrer" />

## 关于hashcode

1、hashcode的存在主要是用于查找的快捷性，如Hashtable、HashMap等，hashcode是用来在散列存储结构中确定对象的存储地址的。

2、如果两个对象相同，就是适用于`equals(java.lang.Object)`方法，那么这两个对象的hashcode方法一定要相同。

3、如果对象的equals方法被重写，那么对象的hashcode方法也尽量重写，并且产生hashcode使用的对象，一定要和equals方法中使用的一致，否则就会违反上面提到的第2点。

4、两个对象的hashcode相同，并不一定表示两个对象就相同，也就是不一定适用于`equals(java.lang.Object)`方法，只能够说明这两个对象在散列存储结构中，如Hashtable，他们“存放在一个篮子中”。

<!-- More -->
**再次归纳一下就是hashCode是用于查找使用的，而equals是用于比较两个对象是否相等的**

## 关于equals

1、equals和==
- ==用于比较引用和比较基本数据类型时具有不同的功能：
    比较基本数据类型，如果两个值相同，则结果为true；
    而在比较引用时，如果引用指向内存中的同一对象，结果为true。

2、equals作为方法，实现对象的比较。由于==运算符不允许我们进行覆盖，也就是说它限制了我们的表达，因此我们复写equals()方法，达到比较对象内容是否相同的目的。而这些通过==运算符是做不到的。

3、Object类的equals()方法的比较规则为：如果两个对象的类型一致，并且内容一致，则返回true。