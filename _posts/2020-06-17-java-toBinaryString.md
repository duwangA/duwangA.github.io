---
title:  对于java源码Interger类toBinaryString的理解
layout: post
categories: java源码
tags: java, 源码
excerpt: 自己对于该方法的一些理解
---
---
#	对于java源码Interger类toBinaryString的理解	<span id="home">

---
---
#	目录

* **[源码](#1)**
	* **[toBinaryString](#1.1)**
	* **[toUnsignedString0](#1.2)**
	* **[numberOfLeadingZeros](#1.3)**
	* **[formatUnsignedInt](#1.4)**
* **[toUnsignedString0概述](#2)**
* **[numberOfLeadingZeros详解](#3)**
* **[formatUnsignedInt详解](#4)**
* **[附录](#5)**

---
---

#	源码	<span id = "1">
##	toBinaryString	<span id = "1.1">

	public static String toBinaryString(int i) {
        return toUnsignedString0(i, 1);
    }

##	toUnsignedString0	<span id = "1.2">

	private static String toUnsignedString0(int val, int shift) {
        // assert shift > 0 && shift <=5 : "Illegal shift value";
        int mag = Integer.SIZE - Integer.numberOfLeadingZeros(val);
        int chars = Math.max(((mag + (shift - 1)) / shift), 1);
        char[] buf = new char[chars];

        formatUnsignedInt(val, shift, buf, 0, chars);

        // Use special constructor which takes over "buf".
        return new String(buf, true);
    }

##	numberOfLeadingZeros	<span id = "1.3">

	public static int numberOfLeadingZeros(int i) {
        // HD, Figure 5-6
        if (i == 0)
            return 32;
        int n = 1;
        if (i >>> 16 == 0) { n += 16; i <<= 16; }
        if (i >>> 24 == 0) { n +=  8; i <<=  8; }
        if (i >>> 28 == 0) { n +=  4; i <<=  4; }
        if (i >>> 30 == 0) { n +=  2; i <<=  2; }
        n -= i >>> 31;
        return n;
    }

##	formatUnsignedInt	<span id = "1.4">

	static int formatUnsignedInt(int val, int shift, char[] buf, int offset, int len) {
        int charPos = len;
        int radix = 1 << shift;
        int mask = radix - 1;
        do {
            buf[offset + --charPos] = Integer.digits[val & mask];
            val >>>= shift;
        } while (val != 0 && charPos > 0);

        return charPos;
    }

---

#	toBinaryString
对于toBinaryString，返回参数数值的补码形式，正数则忽略前面的0。（官方注释：返回表示传入参数的一个无符号(这里无符号大概只是指前面没有+-号，但还是有符号位) 的二进制字符串。如果参数为负数x，返回值为 2^32 + x 【就是它的补码】）

#	toUnsignedString0概述	<span id = "2">
可以看出，该方法传入了 val 和 shift 两个参数，这两个参数分别代表了数值和进制。对于shift，1代表2进制，3代表8进制，4代表16进制。

代表主要功能是代码的主要流程是分为两步： 第一步，计算出用于表示二/八/十六进制的字符数组的长度保存在mag参数内；第二步，使用formatUnsignedInt方法填充字符数组，得到所需进制表示的字符串并返回。

#	numberOfLeadingZeros	<span id = "3">
第一步中字符数组长度的计算是通过numberOfLeadingZeros方法计算int变量的计算机二进制表示的高位连续0位的数量，进而获得最高非0位到最低位的长度，也就是需要表示的位数。

以整数6为例，其在计算机中的二进制存储为 `0000 0000 0000 0000 | 0000 0000 0000 0110`

步骤如下
首先i=6≠0。

首个if的判断，如果无符号右移16位，即0000 0000 0000 0000 | 0000 0000 0000 0000，可知符合条件，则n=1+16=17，i左移16位得 `0000 0000 0000 0110 | 0000 0000 0000 0000`

进入下一个if，如果无符号右移24位，即0000 0000 0000 0000 | 0000 0000 0000 0000，可知符合条件，则n=17+8=25,i左移8位得 `0000 0110 0000 0000 | 0000 0000 0000 0000`

进入下一个if，如果无符号右移28位，即0000 0000 0000 0000 | 0000 0000 0000 0000，可知符合条件，则n=25+4=29,i左移4位得 `0110 0000 0000 0000 | 0000 0000 0000 0000`

进入下一个if，如果无符号右移30位，即0000 0000 0000 0000 | 0000 0000 0000 0001，不符合条件

执行 n = n-i >>> 31,得到n=29,mag = 32 -29 = 3,即得出最高非零位前面的0的个数。继续计算得chars = 3，故创建一个字符数组buf，长度为3。

#	formatUnsignedInt	<span id = "4">

`val = 6, shift = 1, buf = new char[3], offset = 0, len = 3`

charPos = 3表示有效位的个数，radix = 2表示要转换成二进制的基数，若是要转换后八进制，则radix=8,同理，十六进制radix=16；mask = 1,一般我们要将十进制转换成其他进制的话，只需要&(与)上对应的进制基数即可得到结果，就好比是八进制的话就&7、十六进制的话就&15，这是最快得到十六进制表示的数值。

例如要把19转换成八进制：

1.先把19转成二进制，也就是10011；

2.因为8是2的3次方，所以把10011从低位开始每3位一组划分，也就是10 011；

3.把10 011按每一组转为八进制，也就是2 3，所以19的八进制表示就是23。（二进制同理，个人感觉八进制更好理解）

val为6，shift为1，radix为2，mask为1。

第一次循环：6 & 1就是0110 & 0001，结果为0，通过digits数组得到这一位`字符`为0（注意，这一步已经转化为字符型）。然后0110右移1位，得到val为011。

第二次循环：011 & 001，结果为1，通过digits数组得到这一位`字符`为1。然后011右移1位，得到val为01。

第二次循环：01 & 01，结果为1，通过digits数组得到这一位`字符`为1。然后01右移1位，得到val为0，循环结束。
最终结果为110（二进制），储存在buf字符数组并输出

---

#附digits表	<span id = "5">

	final static char[] digits = {
        '0' , '1' , '2' , '3' , '4' , '5' ,
        '6' , '7' , '8' , '9' , 'a' , 'b' ,
        'c' , 'd' , 'e' , 'f' , 'g' , 'h' ,
        'i' , 'j' , 'k' , 'l' , 'm' , 'n' ,
        'o' , 'p' , 'q' , 'r' , 's' , 't' ,
        'u' , 'v' , 'w' , 'x' , 'y' , 'z'
    };