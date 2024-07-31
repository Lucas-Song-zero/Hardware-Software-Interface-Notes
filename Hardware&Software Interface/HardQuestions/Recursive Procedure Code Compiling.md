# 递归/函数嵌套过程的MIPS汇编实现
---
## Core Concepts:
```
1.叶过程
```
---
不调用其他过程的过程称为**叶过程**,但是其他情况下就要涉及到调用其他甚至自身过程(**递归**),这个过程中容易发生返回值以及返回地址的冲突,从而导致程序出错。为了解决这个问题,使用了**将所有必须保留的寄存器压栈**的解决方案。

下面是[計算機組織 Chapter 2.8 Compiling a Recursive C Procedure - 朱宗賢老師](https://www.youtube.com/watch?v=06KE61kXl4w&list=PLylnxZnYW9LbVL5HnYwo7VLmlkhM7lTey)关于Recursive Procedure Compelling的讲解。
其以n=2的阶乘为例:
```
//C Code
int fact(int n)
{
    if(n<2) return 1
    else return n*fact(n-1)
}
```
其编译成汇编程序就是
```
fact:
    addi $sp,$sp,-8
    sw $ra,4($sp)
    sw $a0,0($sp)
    slti $t0,$a0,2
    beq $t0,0,L1
    addi $v0,$zero,1
    addi $sp,$sp,8
    jr $ra

L1:
    addi $a0,$a0,-1
    jal fact
    lw $a0,0($sp)
    lw $ra,4($sp)
    addi $sp,$sp,8
    mul $v0,$a0,$v0
    jr $ra
```
![alt text](..\assets\image.png)
*图1 the condition image of the stack*

第一步:(这里我们认为`$a0`存储了n的值,最开始是2),所以是先开一个2字(2Byte=8bit)的stack用来存储`$a0`和`$ra`(用于返回主程序下一条指令的地址)
对应`fact中的sw两段代码`

第二步:然后比较n(`$a0`)和2谁大,如果n<2就满足终止条件,否则就要跳转到L1(这一步是通过`$t0`临时变量的0/1值决定的)
对应`fact中的slti和beq两段`

第三步:跳转到L1之后,就是将n减一(`addi $a0 $a0 -1`),然后跳回fact(此时已经将这L1中下一段代码地址记录到`$ra`中,也就是`$ra`更新为Y:the address of `lw $a0,0(sp)` in L1 section)

第四步:此时`$a0`和`$ra`都已经更新了,所以还要腾出足够空间把这两个数据存进去。(和第一步是一样的过程)

第五步:再次比较`$a0`(n)和2谁大,此时n=1,比2小,满足条件,`$t0`被置为1,`beq`条件不满足,不跳转。

第六步:返回1,通过`addi $v0,$zero,1`实现,同时pop不需要的栈部分(见图)
(**PS:事实上最后返回之前的入栈并没有起到什么作用**)

第七步:跳转到`$ra`指向的代码片段,是Y:the address of `lw $a0,0(sp)` in L1 section,接下来的两个lw代码实现了重新将存储的值更新到`$a0`和`$ra`中,并且从栈中删除这两个字,然后是`mul`实现了相乘。最后继续跳转到更新后的`$ra`上(是X:the address of main() process)

