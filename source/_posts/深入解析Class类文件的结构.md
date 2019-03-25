---
title: 深入解析Class类文件的结构
tags:
  - java
  - jvm
categories:
  - 原创
  - JVM
copyright: true
abbrlink: 7eb7d5c7
date: 2019-03-21 23:23:07
---
## 前言
要深入学习Java以及Java虚拟机，深入学习Java字节码文件是绕不开的一条路，只有知道了字节码文件里的排列结构，你才能透彻的了解在JVM里，类加载是怎么加载Java类的，是怎么将二进制流转化为运行时数据结构的。

Class文件是是一组以8字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件中，中间没有任何分隔符。

这里的Class文件其实不是特指Java的字节码文件，任何编程语言的编译器只要按照字节码文件规范编译成Class文件，都可以在JVM上运行，所以字节码文件和JVM是和语言无关的。

另外一般Class文件指的不一定是存储在磁盘上的以.class后缀结束的文件，是一种泛指，指的是一切按照字节码文件规范排列的二进制字节流。
## 字节码文件解析
Class文件采用下面这种类似C语言的结构体的伪结构来存储数据，整个Class文件是一张表，表里又由无符号数和表组成。<!--more-->
```c++
ClassFile { 
    u4  magic; // 魔数，固定为"0xCAFEBABY"
    u2  minor_version; //jdk次版本号
    u2  major_version;  //jdk主版本号
    u2  constant_pool_count;  //常量池数组大小，从1计数
    cp_info  constant_pool[constant_pool_count - 1]; //常量池数组
    u2  access_flags;  //类的访问标志，如：public
    u2  this_class;  //类索引，指向常量池中的类符号引用
    u2  super_class;  //父类索引，指向常量池中的类符号引用
    u2  interfaces_count; //实现的接口的数量
    u2  interfaces[interfaces_count]; //接口列表，按implements后面的接口顺序
    u2  fields_count;  //字段数
    field_info  fields[fields_count]; //字段表
    u2  methods_count; //方法数
    method_info  methods[methods_count]; //方法表
    u2  attributes_count; //属性表大小
    attribute_info  attributes[attributes_count]; //属性表
}

```
从上面的伪结构可以看到，Class文件根据上面的顺序把规定的数据类型按照占用的字节依次排列下来。

下面通过一个例子来实战分析一下Class文件
```java
//Test.class
public class Test { 
    public static int a = 1;
    public static final int b = 1; 
    public void say(){
        System.out.println("Hello");
    }
}
```

![字节码文件实战分析](7eb7d5c7/字节码文件实战分析.png)

上图是编译后的Test.class文件的二进制数据，可以按照上面ClassFile的结构顺序依次分析下，下面是部分分析结果：  
(1) ***u4 magic***  
&nbsp;&nbsp;&nbsp;&nbsp;4个字节(000h:0123)魔数: 0xCAFEBABY  

(2) ***u2  minor_version***  
&nbsp;&nbsp;&nbsp;&nbsp;2个字节(000h:45)次版本号: 0x0000, 次版本号为0  

(3) ***u2  major_version***  
&nbsp;&nbsp;&nbsp;&nbsp;2个字节(000h:67)主版本号: 0x0034,即52,JDK1.0-1.1：45.0 ~ 45.3, 1.1后版本增1，数字加1，所以这里用的是1.1 + 0.(52-45) = 1.8  

(4) ***u2  constant_pool_count***  
&nbsp;&nbsp;&nbsp;&nbsp;2个字节(000h:89)常量池大小:0x0027,即39，常量池数组是从1开始计数的，说明常量池中有38个常量，后面依次排列的就是常量池的38个常量  

(5) ***cp_info  constant_pool[constant_pool_count - 1]***  
&nbsp;&nbsp;&nbsp;&nbsp;常量池所占的字节数是由常量池中常量的数量以及类型所决定的，这里有38个常量，每个常量开头都有一个字节的tag标识常量的类型，具体类型可以参考最下面的脑图，根据这个标识可以找到这个常量所占的字节以及含义，下面分析其中一个常量，其余的读者有兴趣可以全部完成  
- 000h:a 0x0A,表示常量类型为10，查表可知是CONSTANT_Methodref方法符号引用，那接下来的四个字节，前两个字节表示指向常量池中方法所在类的符号引用的索引项，就是常量池的数组下标，所在的位置是方法所在类的符号引用
- 000h:bc 0x0007,指向常量池数组第7个元素，第7个常量是一个java.lang.Object类的符号引用
- 000h:de 0x0018, 指向常量池数组的第24个元素，第24个常量是一个名称和类型的符号引用，方法名是`<init>`，描述符是`()V`
这样第一个常量就分析完成，共占5个字节，表示的是方法符号引用，该方法所在的类是Object类，方法名称是`<init>`, 无参数，返回值是void  

借助工具javap可以更直观的看到我们刚刚分析的部分结果以及全部类文件的结构，使用以下命令即可：
```
javap -v Test.class
```
结果如图：  
![javap](7eb7d5c7/javap.png)

通过上面的图可以看到，和我们上面的部分分析是一致的  
## Class文件结构脑图
下面是我在看《深入理解Java虚拟机》这本书的时候整理的关于Class文件结构的脑图，图片比较大，右键另存为图片再查看会更方便。  

![Class文件结构脑图](7eb7d5c7/class.png)

---