---
title: (Linux 0.11) Draft 4 GCC Assembly
---
The content is recoverd from Wordpress Blog, for more details please check [HERE](recover-my-blog)



#####  [March 27, 2017](https://web.archive.org/web/20210418225512/https://void-shana.moe/linux/draft-4-gcc-assembly.html "12:49 am") 
[VOID001](https://web.archive.org/web/20210418225512/https://void-shana.moe/author/void001 "View all posts by VOID001") Comments  [2 comments](https://web.archive.org/web/20210418225512/https://void-shana.moe/linux/draft-4-gcc-assembly.html#comments)





全文参考　https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#Extended-Asm


阅读文档永远比阅读博客能获得更多的信息哦～


本文对 GCC Assembly 进行总结和介绍，也是个人的笔记，欢迎各位指正，内容供参考


* Basic Asm — 基本不能用的 assembly  

* Extended Asm 功能强大的可扩展的ASM



Extended Assembly
-----------------


语法如下:



```
asm [volatile](Assembler template
:Output Operands
[:Input Operands]
[:Clobbers])
```

 


扩展汇编由以上部分组成，以下是一个例子: (摘自linux0.11 mm/memory.c)



```
\_\_asm\_\_("std ; repne ; scasb\n\t"
"jne 1f\n\t"
"movb $1,1(%%edi)\n\t"
"sall $12,%%ecx\n\t"
"addl %2,%%ecx\n\t"
"movl %%ecx,%%edx\n\t"
"movl $1024,%%ecx\n\t"
"leal 4092(%%edx),%%edi\n\t"
"rep ; stosl\n\t"
" movl %%edx,%%eax\n"
"1: cld"
//Above are Assembler Template
:"=a" (\_\_res) // This is Output ,(s)
:"0" (0),"i" (LOW\_MEM),"c" (PAGING\_PAGES), // This is Input ,(s)
"D" (mem\_map+PAGING\_PAGES-1) // This is Clobber(s)
);

```

 


#### Assembler Template


先来对 Assembler Template 进行讲解,上文代码应该已经包含了 Assembler Template 的大部分特性, 首先我们最初的感受就是,Assembler Template和AT&T很像,不过又有一些汇编中不会出现的写法,比如 %%eax, %2 . 下面就对这些特殊的占位符进行介绍.


* %% 这个就是代表汇编中的一个 ‘%’ 因为在Assembler Template里’%’具有特殊的含义,因而需要转义  

* %= 用来生成全局唯一的标签, 如果在你的代码里有多处用到标签的话,那么用这个来给每一段汇编代码生成标签可以免除给标签唯一编号的痛苦)  

* %{ %} %| 在 Assembler Template 里 ‘{}|’都具有特殊的含义,因而我们需要用’%’才能代表汇编代码中的相应字符


既然说到了Assembler Template里面'{}|’具有不同的含义,那么接下来就具体介绍一下他的含义:  

在Assembler Template里'{}|’表示汇编方言(dialects), 在x86-gcc上支持多种汇编方言,可以使用-masm控制gcc使用的inline assembler方言,比如下方的例子:



```
"bt{l %[Offset],%[Base] | %[Base],%[Offset]}; jc %l2"
```

使用不同的方言的时候将被解释为不同的汇编语句



```
"btl %[Offset],%[Base] ; jc %l2" /* att dialect */
"bt %[Base],%[Offset]; jc %l2" /* intel dialect */
```

(不过到底有什么用处我也不知道)


#### Operands


GCC Extened Assembly 最强大的就是可以让编译器自己决定如何将我们的汇编代码优化，还可以在汇编中使用Ｃ代码里面的变量和标签～，具体的使用方式就是传递参数，通过传递Input, Output Clobber 三种Operands


##### Output Operands


Operands 通过逗号分割, 每一个operand由三部分组成 `[ [asmSymbolicName] ] constraint (cvariablename)`  

其中 asmSymbolicName 可选, 如果不填那么就是从0开始,即 %0 表示第一个操作数 %1, 第二个. 这里要注意 擦作数的编号Input, Output是共用的, 也就是说, **如果你有一个Output Operand两个Input Operand, 那么%1指第一个Input Operand**


constraint看起来比较反人类,这里仅仅介绍一些基本的constraint.


“a”-“d” eax – edx  

“S” “D” esi, edi  

“r” 任意动态分配的寄存器(由编译器控制)  

“g” 任意一个通用寄存器(eax-edx)或者内存  

“m” 使用内存地址  

“o” 使用内存地址,并且可以加Offsethanyi  

“=” 表示操作数会输出,输出会替换掉之前的值  

“+” 表示既可读也可写


#### Input Operands


Input Operands 的constriant不能以 “=”, “+” 开头.


要注意,不要在Extended Assemlby中修改Input Operand的值(不过同样是Output Operand的时候,可以修改),因为编译器会假定Input Operand在asm调用前后保持不变,如果修改可能会导致不可预料的后果


#### Clobbers


在修改Output Operands的时候,还有可能牵扯到中间寄存器的变化,我们可以在Clobbers里说明哪些寄存器可能会发生变化  

Clobber中不应该包含任何一个在 Input / Output Operands 里的操作数  

其中两个特殊的Operands  

* cc 表示FLAG寄存器会被修改  

* “memory” 内存修改符, 用于指出在 Input / Output Operands 中没有出现的操作数,比如,某个寄存器指向的内存地址.这时候指定clobber可以保证那个内存地址存放正确的值


#### Sample


首先我们来解释一下上面那一段Extened Assembly的含义



```
\_\_asm\_\_("std ; repne ; scasb\n\t"
"jne 1f\n\t"
"movb $1,1(%%edi)\n\t"
"sall $12,%%ecx\n\t"
"addl %2,%%ecx\n\t"
"movl %%ecx,%%edx\n\t"
"movl $1024,%%ecx\n\t"
"leal 4092(%%edx),%%edi\n\t"
"rep ; stosl\n\t"
" movl %%edx,%%eax\n"
"1: cld"
//Above are Assembler Template
:"=a" (\_\_res) // This is Output ,(s)
:"0" (0),"i" (LOW\_MEM),"c" (PAGING\_PAGES), // This is Input ,(s)
"D" (mem\_map+PAGING\_PAGES-1) // This is Clobber(s)
);
```

 


首先来看Operands, Output为一个寄存器变量 `\_\_res`, 并且可以覆盖掉 `\_\_res` 原来的值, Input Operands 为三个 第一个是 常数 0, 第二个是一个立即数 LOW\_MEM, 第三个是存放在%ecx里的变量,值为 PAGING\_PAGES, Clobber为 %edi 的内容,因为上述中涉及到块操作,需要修改%edi的值(为什么不把它作为Input Operand呢)


然后我们来看代码. 这段代码从 ES(数据段):%edi 指向的地址开始向low addr遍历,直到遇到第一个等于AL(0)就停止循环,这时%ecx的值也随之更新  

然后将%edi + 1处的值set为1  

[中间过程都是比较好理解的汇编,我么你直接跳到addl %2这一行]  

然后我们将%ecx的值+LOW\_MEM算出,放到%edx中.  

[后续过程也不好单独解释,考虑可能会结合Memory Managment一起解释]


然后我们来自己写一个,用来将一块连续的内存复制到目的地址的函数,使用GCC Extended Assembly


 



```
#include <stdio.h>
#include <string.h>

int main(void) {
    char str1[] = "This is a string to be copied";
    char str2[50] = "";
    as\_memcpy(str2, str1, strlen(str1) + 1);
    printf("%s", str2);
}

void as\_memcpy(char *dst, char *src, int len) {
    asm volatile (
            "1: movb (%%esi), %%al\n\t"
            "movb %%al, (%%edi)\n\t"
            "inc %%esi\n\t"
            "inc %%edi\n\t"
            "dec %%ecx\n\t"
            "jne 1b\n\t
            :
            : "c" (len), "S" (src), "D" (dst));
    return ;
}

```

 


完整代码如上, 不过要注意, 编译要加参数 `-m32` 表示这是32位的代码 . 不然的话, 因为兼容性的原因, 我们的%esi的地址就会被破坏掉,引发段异常 **(在64位的环境下, %esi = %rsi & 0xFFFFFFFF, 并且因为兼容性问题, 在64位下访问%esi的地址会被转换为高32位全是1,低32位是%esi的地址形式,因而就跑到了非法的内存区域(内核区),越权访问,引发段异常)**


因而在写 Easm 的时候一定要注意处理器目前所在的环境是多少位的. 不然就会因为mix遇到这种奇怪的问题


同时,写Easm的时候,最好都加上volatile,不然可能编译器将你的ASM优化为你不想要的样子哦


这篇文章就先到这里了~






---


[C](https://web.archive.org/web/20210418225512/https://void-shana.moe/category/linux/c), [Linux](https://web.archive.org/web/20210418225512/https://void-shana.moe/category/linux) [C. Linux](https://web.archive.org/web/20210418225512/https://void-shana.moe/tag/c-linux), [kernel](https://web.archive.org/web/20210418225512/https://void-shana.moe/tag/kernel), [Laravel](https://web.archive.org/web/20210418225512/https://void-shana.moe/tag/laravel), [PHP](https://web.archive.org/web/20210418225512/https://void-shana.moe/tag/php), [Python](https://web.archive.org/web/20210418225512/https://void-shana.moe/tag/python), [Shell](https://web.archive.org/web/20210418225512/https://void-shana.moe/tag/shell), [Web](https://web.archive.org/web/20210418225512/https://void-shana.moe/tag/web), [wine](https://web.archive.org/web/20210418225512/https://void-shana.moe/tag/wine) 






------------------------
## Historical Comments
1. ![](https://web.archive.org/web/20210418225512im_/https://secure.gravatar.com/avatar/13650e6abb26b891c6bed92ffe8b2b5f?s=50&d=identicon&r=g) **ホロ** says: 
[March 27, 2017 at 8:02 pm](https://web.archive.org/web/20210418225512/https://void-shana.moe/linux/draft-4-gcc-assembly.html#comment-250)
导航栏上面那个Home坏了……?


	1. ![](https://web.archive.org/web/20210418225512im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[March 31, 2017 at 9:46 pm](https://web.archive.org/web/20210418225512/https://void-shana.moe/linux/draft-4-gcc-assembly.html#comment-251)
	修好啦


	1. ![](https://web.archive.org/web/20210418225512im_/https://secure.gravatar.com/avatar/5612f7d51961a8e49efb43a5e4cf18a6?s=50&d=identicon&r=g) **VOID001** says: 
	[March 31, 2017 at 9:46 pm](https://web.archive.org/web/20210418225512/https://void-shana.moe/linux/draft-4-gcc-assembly.html#comment-251)
	修好啦

            