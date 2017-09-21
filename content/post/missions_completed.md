+++
title = "Flags Archives"
description = "最怕一生碌碌无为 还说平凡难能可贵"
tags = [
    "flag"
]
date = "2017-09-15"
topics = [
    "life"
]
toc = true
+++

- [日读源码](../soure_code_reading/)
- [手倒立](../the_road_to_handstand/)
- THIS.RECORDING


## IDEAS


- DH工程javadoc注释，分享
- 流程中添加zip包检查逻辑——避免走到了执行阶段才发现条件不符合要求
- 源码阅读理解，以故事的方式解释代码逻辑
- 七牛图床使用


## READING MATERIALS



[安息吧 REST API，GraphQL 长存](http://www.zcfy.cc/article/rest-apis-are-rest-in-peace-apis-long-live-graphql-3935.html)</br>
[REST APIs are REST-in-Peace APIs. Long Live GraphQL.](https://medium.freecodecamp.org/rest-apis-are-rest-in-peace-apis-long-live-graphql-d412e559d8e4)</br>

[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api)


[关于容器、虚拟机以及 Docker 的一个入门教程](http://zcfy.cc/article/a-beginner-friendly-introduction-to-containers-vms-and-docker-4139.html?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)</br>
[A Beginner-Friendly Introduction to Containers, VMs and Docker](https://medium.freecodecamp.org/a-beginner-friendly-introduction-to-containers-vms-and-docker-79a9e3e119b)


### Java bit
[Java位运算](http://www.cnblogs.com/zhengtao/articles/1916751.html)</br>
[整数二进制补码的数学原理(two's complement)](http://www.cnblogs.com/effulgent/archive/2011/10/30/two_s_complement.html)</br>

二进制数在内存中是以补码的形式存放：正数的补码与原码相同,负数的原码是其反码+1

原始来源为二进制表示：

正数的补码，反码都是其本身

>正数9的二进制为：1001，在内存中存储为01001，必须补上符号位（开头为符号位，0表示正数，1表示负数）
>9的补码为01001，反码也是01001


负数的补码是：符号位不变，其余各位求反，末位加1 

>-9的二进制为 11001 + 1 = 11010


负数的反码是：符号位为1，其余各位求反，但末位不加1

```
public class Test {
	public static void main(String args[]) {
        int a = 60;	/* 60 = 0011 1100 */
        int b = 13;	/* 13 = 0000 1101 */
        int c = 0;

        c = a & b;        /* 12 = 0000 1100 */
        System.out.println("a & b = " + c);

        c = a | b;        /* 61 = 0011 1101 */
        System.out.println("a | b = " + c);

      			/* 60 = 0011 1100 */
                  /* 13 = 0000 1101 */
        c = a ^ b;        /* 49 = 0011 0001 */
        System.out.println("a ^ b = " + c);

      			/* 60 = 0011 1100 */
      			/* 13 = 0000 1101 */
        c = ~a;           /*-61 = 1100 0011 */
        System.out.println("~a = " + c);

      			/* 60 = 0011 1100 */
      			/* 13 = 0000 1101 */
        c = a << 2;       /* 240 = 1111 0000 */
        System.out.println("a << 2 = " + c);

     			/* 60 = 0011 1100 */
      			/* 13 = 0000 1101 */
        c = a >> 2;       /* 15 = 1111 */
        System.out.println("a >> 2  = " + c);

      			/* 60 = 0011 1100 */
      			/* 13 = 0000 1101 */
        c = a >>> 2;      /* 15 = 0000 1111 */
        System.out.println("a >>> 2 = " + c);


        // http://www.cnblogs.com/eRrsr/p/6401881.html
        //按位与运算&
        System.out.println(0 & 0);//0
        System.out.println(0 & 1);//0
        System.out.println(1 & 1);//1
        System.out.println("===========");
        //按位或运算符|
        System.out.println(0 | 0);//0
        System.out.println(0 | 1);//1
        System.out.println(1 | 1);//1
        System.out.println("===========");
        //异或运算符^
        System.out.println(0 ^ 0);//0
        System.out.println(0 ^ 1);//1
        System.out.println(1 ^ 1);//0
        System.out.println("===========");

        // http://blog.csdn.net/smilecall/article/details/42454471
        // 取反运算符~
        // 部分要点:
        // 什么是取反:取反就是0=>1 1=>0
        // 什么是补码:正数的补码是其反码,负数的补码为其反码+1,
        // 例5的二进制为0 0101,而0 0101的补码是1 1010,-5的二进制是1 0101,而1 0101的补码是1 1011
        // 什么是原码:规定正数的补码与原码相同,负数的原码是其反码+1
        //---------------------------------------------------------
        // 6为正数,二进制为 0 0110 (第一个0代表正负) 0000 0110的缩写样式，在内存中的表示方法
        // 然后计算补码,即1 1001
        // 求原码,对后4位进行按位取反,即 1 0110
        // 然后对二进制进行补码+1操作,即 1 0111
        // 1010转成十进制为7,加上前面的负号,得-7
        // 如6为正数,其二进制为110,取反后为001,补码右边+1为1010,原来6为正,取反为负,得-2
        //----------------------------------------------------------
        System.out.println(~6);//-7
        System.out.println(~42);//-43
        System.out.println("===========");
        // 左移运算符<<(即向左移动,右边补0)
        // 如2的二进制为10,若2<<2,则1000,也就是十进制8,同理若2<<3,则10000,也就是十进制16,根据规律可以看出n<<m=n*(2^m)
        System.out.println(15 << 2);//60: 15*(2^2)
        // 右移有符号运算符>>(即向右移动,但左边补0还是1需要看原来的数是正的还是负的)
        System.out.println(2 >> 2); //0(右边移除的数将被丢弃)
        System.out.println(-8 >> 3); //-1

    }
}
```
[Bitwise Operators Example](https://www.tutorialspoint.com/java/java_bitwise_operators_examples.htm)</br>
[Right shift on negative number](https://stackoverflow.com/questions/15457893/java-right-shift-on-negative-number)</br>
[How are integers internally represented at a bit level in Java?](https://stackoverflow.com/questions/13422259/how-are-integers-internally-represented-at-a-bit-level-in-java)</br>
[Binary presentation of negative integer in Java](https://stackoverflow.com/questions/26315782/binary-presentation-of-negative-integer-in-java)</br>
</br>
[java原码、补码、反码总结](http://blog.csdn.net/qq_30739519/article/details/50991484)


[TOP命令各个参数代表意义详解](https://blog.linuxeye.cn/139.html)</br>
