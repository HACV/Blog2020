---
title: C++的const底层机制
date: 2020-05-31 08:11:04
categories: 
- 编程语言
tags: 
- C++
img: /photos/2020.5.30_31/2020_5_31title.jpg
top: false
summary: C++对C的const关键字增强的底层机制
mathjax: true

cover: false
# password:
---

# C++中对C的const关键字增强

## 一、问题导入
背景：
我们总说C语言中const修饰的变量看上去似乎是常量，其实是个“冒牌货”，应该叫”常变量”，比如用指针间接赋值，就能改变了。

代码测试：
将指针间接修改变量值的代码放到C语言编译器和C++编译器去测试

DevC++的C语言编译器
```c
//demo.c 
#include<stdio.h>
int main()
{
	const int a=98;
	
	int *p=NULL;
	p=(int *)&a;
	*p=100;
	
	printf("%d",a);
	return 0;
}
```
结果：
 <img src="/photos/2020.5.30_31/01.png" width="80%">

DevC++的C++语言编译器
```cpp
//test.cpp 
#include<cstdio>
int main()
{
	const int a=98;
	
	int *p=NULL;
	p=(int *)&a;
	*p=100;
	
	printf("%d",a);
	return 0;
}
```
结果：
 <img src="/photos/2020.5.30_31/02.png" width="80%">

奇特的结果：
我们要是写了这样的函数在银行程序中，要是用不同编译器，那么对账就对不上了。
总结：
C中的const是个常变量，变量的值能够被间接修改。
C++中的const是一个真正的常量！

Tips：
<table><td bgcolor=#FFFF FF>
以上两次测试，都没有显示warning和error</td></table>

那么，我们或许会疑惑：
Q：我们说C++中的是一个真正的常量，那为什么，没有C++编译器对我们”用指针间接修改”的行为，没有报warning或者error呢？
A：C++要兼容C，所以，它认为这个语法是可以的
Q：那么问题又来了，那他既然兼容，那么为什么最后却没改变那个变量的值呢？
A：因为C++只是兼容那种语法写法，但是底层的实现却对const关键字进行了加强。



## 二、底层原理分析
### 1）C++编译器对const做了一些加强，做了一些特殊的处理
当C++编译器，扫描到常量声明时，它不再像C语言那样，把这个const给它单独分配内存。

在我们先前的//test.cpp中。 
C++进行了如下操作：
>- 1）扫描到这一行，const int a=98;
C++编译器会把这个<b>变量a</b>放在一个<b>符号表（键值-值对）</b>里面
<b>此时，并没有分配内存！！！</b>
注意：这样的话，key和value是定了，不能修改的了。
符号表具体的实现和我们的内存中的，栈，堆不是同一套实现机制。
有很多常量就都放在这个里面了。
Tips：
当你去<b>使用</b>这个a的时候，它就给你从符号表里面给你把这个98给拿出来，供你<b>使用</b>
<table><td bgcolor=#FFFF FF>
注意"使用"一词</td></table>

>- 2）遇到类似这样的情况，此时才给a变量另外分配一个内存。
扫描到这一句p=(int *)&a;
当你对这个a变量取地址的时候，C++编译器，会为这个a再<b>单独的开辟一块内存空间</b>，然后你把这个内存空间，赋给了p，相当于一个指针P指向了这里。然后你通过*p去间接的修改的地址，不再是原来的值（value）,而是我们新开辟的空间的值（注意理解）
所以，当你再使用a的时候，你打印的还是98（符号表中的a）
 <img src="/photos/2020.5.30_31/03.png" width="90%">


### 2）证明
我们现在来证明这个开辟的内存空间的存在
```cpp
//solution.cpp 
#include<cstdio>
int main()
{
	const int a=98;
	
	int *p=NULL;
	p=(int *)&a;
	*p=100; 
	
	printf("a=%d\n",a);
	printf("*p=%d",*p);
	return 0;
}
```
打印的是:
 <img src="/photos/2020.5.30_31/04.png" width="80%">

<b>注意：</b>
<table><td bgcolor=#FFFF FF>
C++编译器虽然可能为const常量分配空间，但不会<b>使用</b>其存储空间中的值，除非你用指针操作。</td></table>


## 三、结论和补充
1）C语言中的const变量
C语言中const变量是只读变量，<b>有自己的存储空间</b>
2）C++中的const常量
<table><td bgcolor=#FFFF FF>
注意：可能分配存储空间,也可能不分配存储空间!</td></table>

编译过程中若发现<b>使用常量</b>则直接以符号表中的值替换

Tips：
只有下面两种的时候，它才会分配空间
>- 当const常量为全局，并且需要在其它文件中使用,即使用了<b>extern</b>操作符</b>
>- 当使用&操作符取const常量的地址，编译过程中若发现对const使用了<b>&操作符</b>，则给对应的常量分配存储空间（兼容C）

## 四、补充疑问

Q：那么要是分配内存，C++中那个const的分配内存是在什么时候分配的呢？是在编译器<b>编译阶段</b>，还是在<b>执行阶段</b>分配？
<table><td bgcolor=#FFFF FF>
Ａ：C++中const分配内存的时机，是在编译期间！（记住！）</td></table>

证明的代码：
```cpp
//test.cpp 
#include<cstdio>
int main()
{
	int a;
	const int b=98;
	int c;
	printf("&a=%d\n",&a);
	printf("&b=%d\n",&b);//用了取地址 
	printf("&c=%d\n",&c);
	
	return 0;
}
```
 <img src="/photos/2020.5.30_31/05.png" width="80%">

结果表明：
const int b的地址在a和c之间，符合我们局部变量申请内存的<b>压栈的顺序</b>，<b>它并没有因为，&b这句话写到int c后面，就先分配a，c最后才b</b>，而是，它扫描完之后，看到这里有&b了，就分配地址了。

