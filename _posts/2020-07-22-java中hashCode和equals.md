---
layout:     post
title:      "java中hashCode()和equals()使用"
subtitle:   "Java基础"
date:       2020-07-22
author:     "liuwenzi"
tags:
    - Java基础
---
### java中hashCode()和equals()使用

-------------

>equals()的作用
>
>hashCode的作用
>
>什么时候需要覆盖hashCode()

- equals()

  - equals()用来判断两个对象是否相等

  - 默认情况下通过判断两个对象地址是否相等（即是否是同一个对象,与==作用相同）

  - 也可以对其进行重写

    ```java
    @Override
        public boolean equals(Object obj) {
            if (obj == null){
                return false;
            }
            if (this == obj){
                return true;
            }
            if (this.getClass() != obj.getClass()){
                return false;
            }
            Person person = (Person)obj;
            return name.equals(person.name) && age.equals(person.age);
        }
    ```

    

- hashCode

  - hashCode也就是哈希值，是一个int整数，这个哈希值作用是确定该对象在哈希表中的索引位置

  - hashCode通过调用hashCode()方法生成，hashCode()通过本地方法实现

    ```java
    public native int hashCode();
    ```

  - hashCode()方法生成hashCode的方法可以被重写

- 什么时候需要重写hashCode()

  - 当需要创建该类对应的散列表时（HashSet、HashTable、HashMap）

    >当想散列表中加入对象时，需要先判断散列表中是否有相同的hashCode，若不重写，则加入的在equals()情况下相同的对象hashCode可能不同，在散列表中hashCode不同被当成不同的对象，因此，若不重写在散列表中会加入重复的元素

    hashMap中增加元素的判断

    ```java
    if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
       	e = p;
    ```

参考文章：[浅谈Java中的hashcode方法](https://www.cnblogs.com/dolphin0520/p/3681042.html)

