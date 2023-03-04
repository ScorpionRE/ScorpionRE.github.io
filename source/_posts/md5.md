---

title: MD5
date: 2022-01-28 12:51:00
tags: 
	- CryptoDetect
	- reverse
	- algorithm
categories:
	- [CryptoDetect,algorithm]
keywords:
description:
top_img:
comments:
cover:  https://scorpionre.github.io/2022/01/28/md5/md5.assets/image-20220127171124399.png
toc:
toc_number:
copyright:
mathjax:
katex:
hide:
---

# MD5

### 算法

- 输入：任意长的消息，512 比特长的分组。
- 输出：128 比特的消息摘要

整体流程如下：

- 首先填充原始消息使得对512求余的结果等于448，然后64位记录其长度。

- 512bit一组分为n组。每组中32bit为一段，分为16段
- 对每一组，循环4次下述4轮运算，得到新的A,B,C,D作为下一组的初始值
- 最后得到的A,B,C,D加上第n组原来A,B,C,D的值（即计算前的值）
- 按照地址的顺序从低到高打印对应的A,B,C,D值，就是所求的MD5值。

![image-20220127171124399](md5.assets/image-20220127171124399.png)

#### 填充

如果输入信息的长度(bit)对512求余的结果不等于448，就需要填充使得对512求余的结果等于448。填充的方法是填充一个1和n个0。填充完后，信息的长度就为N*512+448(bit)。

然后用64位来存储填充前信息长度。这64位加在第一步结果的后面，这样信息长度就变为N\*512+448+64=(N+1)*512位



比如，需要加密消息"gnubd"，最后被填充为

`小端字节序存储`

```
67 6E 75 62 64 80 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 28 00 00 00 00 00 00 00 
```

最后64位（8字节)为0x28（40），消息内容为40位（5字节）。

#### 数据处理

要使A,B,C,D在内存中的显示情况：

```c
A = 0x01234567
B = 0x89ABCDEF
C = 0xFEDCBA89
D = 0x76543210
```

程序定义应为（`小端字节序存储`）

```c
A = 0x67452301;
B = 0xEFCDAB89;
C = 0x98BADCFE;
D = 0x10325476;
```

每512位（64字节）为1段可以分成n段，（n大于等于1），对于每一段信息（512位，64字节）又划分成16小段（每段32位，4个字节，用M表示）



每一组，经过下列运算处理：

其中Mj表示消息的第j个子段（从0到15）

```c
 第一轮
a=FF(a,b,c,d,M0,7,0xd76aa478)
b=FF(d,a,b,c,M1,12,0xe8c7b756)
c=FF(c,d,a,b,M2,17,0x242070db)
d=FF(b,c,d,a,M3,22,0xc1bdceee)
a=FF(a,b,c,d,M4,7,0xf57c0faf)
b=FF(d,a,b,c,M5,12,0x4787c62a)
c=FF(c,d,a,b,M6,17,0xa8304613)
d=FF(b,c,d,a,M7,22,0xfd469501)
a=FF(a,b,c,d,M8,7,0x698098d8)
b=FF(d,a,b,c,M9,12,0x8b44f7af)
c=FF(c,d,a,b,M10,17,0xffff5bb1)
d=FF(b,c,d,a,M11,22,0x895cd7be)
a=FF(a,b,c,d,M12,7,0x6b901122)
b=FF(d,a,b,c,M13,12,0xfd987193)
c=FF(c,d,a,b,M14,17,0xa679438e)
d=FF(b,c,d,a,M15,22,0x49b40821)

第二轮
a=GG(a,b,c,d,M1,5,0xf61e2562)
b=GG(d,a,b,c,M6,9,0xc040b340)
c=GG(c,d,a,b,M11,14,0x265e5a51)
d=GG(b,c,d,a,M0,20,0xe9b6c7aa)
a=GG(a,b,c,d,M5,5,0xd62f105d)
b=GG(d,a,b,c,M10,9,0x02441453)
c=GG(c,d,a,b,M15,14,0xd8a1e681)
d=GG(b,c,d,a,M4,20,0xe7d3fbc8)
a=GG(a,b,c,d,M9,5,0x21e1cde6)
b=GG(d,a,b,c,M14,9,0xc33707d6)
c=GG(c,d,a,b,M3,14,0xf4d50d87)
d=GG(b,c,d,a,M8,20,0x455a14ed)
a=GG(a,b,c,d,M13,5,0xa9e3e905)
b=GG(d,a,b,c,M2,9,0xfcefa3f8)
c=GG(c,d,a,b,M7,14,0x676f02d9)
d=GG(b,c,d,a,M12,20,0x8d2a4c8a)

第三轮
a=HH(a,b,c,d,M5,4,0xfffa3942)
b=HH(d,a,b,c,M8,11,0x8771f681)
c=HH(c,d,a,b,M11,16,0x6d9d6122)
d=HH(b,c,d,a,M14,23,0xfde5380c)
a=HH(a,b,c,d,M1,4,0xa4beea44)
b=HH(d,a,b,c,M4,11,0x4bdecfa9)
c=HH(c,d,a,b,M7,16,0xf6bb4b60)
d=HH(b,c,d,a,M10,23,0xbebfbc70)
a=HH(a,b,c,d,M13,4,0x289b7ec6)
b=HH(d,a,b,c,M0,11,0xeaa127fa)
c=HH(c,d,a,b,M3,16,0xd4ef3085)
d=HH(b,c,d,a,M6,23,0x04881d05)
a=HH(a,b,c,d,M9,4,0xd9d4d039)
b=HH(d,a,b,c,M12,11,0xe6db99e5)
c=HH(c,d,a,b,M15,16,0x1fa27cf8)
d=HH(b,c,d,a,M2,23,0xc4ac5665)

第四轮
a=II(a,b,c,d,M0,6,0xf4292244)
b=II(d,a,b,c,M7,10,0x432aff97)
c=II(c,d,a,b,M14,15,0xab9423a7)
d=II(b,c,d,a,M5,21,0xfc93a039)
a=II(a,b,c,d,M12,6,0x655b59c3)
b=II(d,a,b,c,M3,10,0x8f0ccc92)
c=II(c,d,a,b,M10,15,0xffeff47d)
d=II(b,c,d,a,M1,21,0x85845dd1)
a=II(a,b,c,d,M8,6,0x6fa87e4f)
b=II(d,a,b,c,M15,10,0xfe2ce6e0)
c=II(c,d,a,b,M6,15,0xa3014314)
d=II(b,c,d,a,M13,21,0x4e0811a1)
a=II(a,b,c,d,M4,6,0xf7537e82)
b=II(d,a,b,c,M11,10,0xbd3af235)
c=II(c,d,a,b,M2,15,0x2ad7d2bb)
d=II(b,c,d,a,M9,21,0xeb86d391)

FF(a,b,c,d,Mj,s,ti)表示a=b+((a+F(b,c,d)+Mj+ti)<<<s)
GG(a,b,c,d,Mj,s,ti)表示a=b+((a+G(b,c,d)+Mj+ti)<<<s)
HH(a,b,c,d,Mj,s,ti)表示a=b+((a+H(b,c,d)+Mj+ti)<<<s)
II(a,b,c,d,Mj,s,ti)表示a=b+((a+I(b,c,d)+Mj+ti)<<<s)
    
//四个线性函数定义如下
F(X,Y,Z)=(X&Y)|((~X)&Z)
G(X,Y,Z)=(X&Z)|(Y&(~Z))
H(X,Y,Z)=X^Y^Z
I(X,Y,Z)=Y^(X|(~Z))
```

每轮循环后，将A，B，C，D分别加上a，b，c，d，然后进入下一循环，共循环四次。



循环结束后，新的A,B,C,D作为下一组A,B,C,D的初始值，继续进行上述循环运算，得到新的A,B,C,D再加上初始A,B,C,D值。最后按照地址的顺序从低到高打印对应的A,B,C,D值，就是所求的MD5值。



### signature

DSL定义如下

![image-20220127171654001](md5.assets/image-20220127171654001.png)后面的3614090360（0xD76AA478),即为相应值转化为十进制后的数。

ROTATE表示移位

```java
private static long F(long x, long y, long z) {
        return (x & y) | ((~x) & z);
}

private static long FF(long a, long b, long c, long d, long x, long s,
            long ac) {
        a += (F(b, c, d)&0xFFFFFFFFL) + x + ac;
        a = ((a&0xFFFFFFFFL)<< s) | ((a&0xFFFFFFFFL) >>> (32 - s));
        a += b;
        return (a&0xFFFFFFFFL);
    }


a = FF(a, b, c, d, groups[0], S11, 0xd76aa478L);


```





### 测试

libcrypto.so.1.1(openssl)

其中和md5有关的函数如下

![image-20220127175833516](md5.assets/image-20220127175833516.png)

利用where's crypto分析

![image-20220127181300578](md5.assets/image-20220127181300578.png)

![image-20220128122013626](md5.assets/image-20220128122013626.png)

反汇编MD5_Update,主要还是调用了md5_block_data_order

![image-20220128122933514](md5.assets/image-20220128122933514.png)o

md5_block_data_order反汇编后

![image-20220128123055931](md5.assets/image-20220128123055931.png)

与signature中对应关系如下

![image-20220128124206153](md5.assets/image-20220128124206153.png)

openssl中源代码

```c
#ifndef md5_block_data_order
# ifdef X
#  undef X
# endif
void md5_block_data_order(MD5_CTX *c, const void *data_, size_t num)
{
    const unsigned char *data = data_;
    register unsigned MD32_REG_T A, B, C, D, l;
# ifndef MD32_XARRAY
    /* See comment in crypto/sha/sha_local.h for details. */
    unsigned MD32_REG_T XX0, XX1, XX2, XX3, XX4, XX5, XX6, XX7,
        XX8, XX9, XX10, XX11, XX12, XX13, XX14, XX15;
#  define X(i)   XX##i
# else
    MD5_LONG XX[MD5_LBLOCK];
#  define X(i)   XX[i]
# endif

    A = c->A;
    B = c->B;
    C = c->C;
    D = c->D;

    for (; num--;) {
        (void)HOST_c2l(data, l);
        X(0) = l;
        (void)HOST_c2l(data, l);
        X(1) = l;
        /* Round 0 */
        R0(A, B, C, D, X(0), 7, 0xd76aa478L);
        (void)HOST_c2l(data, l);
        X(2) = l;
        R0(D, A, B, C, X(1), 12, 0xe8c7b756L);
        (void)HOST_c2l(data, l);
        X(3) = l;
        R0(C, D, A, B, X(2), 17, 0x242070dbL);
        (void)HOST_c2l(data, l);
        X(4) = l;
        R0(B, C, D, A, X(3), 22, 0xc1bdceeeL);
        (void)HOST_c2l(data, l);
        X(5) = l;
        R0(A, B, C, D, X(4), 7, 0xf57c0fafL);
        (void)HOST_c2l(data, l);
        X(6) = l;
        R0(D, A, B, C, X(5), 12, 0x4787c62aL);
        (void)HOST_c2l(data, l);
        X(7) = l;
        R0(C, D, A, B, X(6), 17, 0xa8304613L);
        (void)HOST_c2l(data, l);
        X(8) = l;
        R0(B, C, D, A, X(7), 22, 0xfd469501L);
        (void)HOST_c2l(data, l);
        X(9) = l;
        R0(A, B, C, D, X(8), 7, 0x698098d8L);
        (void)HOST_c2l(data, l);
        X(10) = l;
        R0(D, A, B, C, X(9), 12, 0x8b44f7afL);
        (void)HOST_c2l(data, l);
        X(11) = l;
        R0(C, D, A, B, X(10), 17, 0xffff5bb1L);
        (void)HOST_c2l(data, l);
        X(12) = l;
        R0(B, C, D, A, X(11), 22, 0x895cd7beL);
        (void)HOST_c2l(data, l);
        X(13) = l;
        R0(A, B, C, D, X(12), 7, 0x6b901122L);
        (void)HOST_c2l(data, l);
        X(14) = l;
        R0(D, A, B, C, X(13), 12, 0xfd987193L);
        (void)HOST_c2l(data, l);
        X(15) = l;
        R0(C, D, A, B, X(14), 17, 0xa679438eL);
        R0(B, C, D, A, X(15), 22, 0x49b40821L);
        /* Round 1 */
        R1(A, B, C, D, X(1), 5, 0xf61e2562L);
        R1(D, A, B, C, X(6), 9, 0xc040b340L);
        R1(C, D, A, B, X(11), 14, 0x265e5a51L);
        R1(B, C, D, A, X(0), 20, 0xe9b6c7aaL);
        R1(A, B, C, D, X(5), 5, 0xd62f105dL);
        R1(D, A, B, C, X(10), 9, 0x02441453L);
        R1(C, D, A, B, X(15), 14, 0xd8a1e681L);
        R1(B, C, D, A, X(4), 20, 0xe7d3fbc8L);
        R1(A, B, C, D, X(9), 5, 0x21e1cde6L);
        R1(D, A, B, C, X(14), 9, 0xc33707d6L);
        R1(C, D, A, B, X(3), 14, 0xf4d50d87L);
        R1(B, C, D, A, X(8), 20, 0x455a14edL);
        R1(A, B, C, D, X(13), 5, 0xa9e3e905L);
        R1(D, A, B, C, X(2), 9, 0xfcefa3f8L);
        R1(C, D, A, B, X(7), 14, 0x676f02d9L);
        R1(B, C, D, A, X(12), 20, 0x8d2a4c8aL);
        /* Round 2 */
        R2(A, B, C, D, X(5), 4, 0xfffa3942L);
        R2(D, A, B, C, X(8), 11, 0x8771f681L);
        R2(C, D, A, B, X(11), 16, 0x6d9d6122L);
        R2(B, C, D, A, X(14), 23, 0xfde5380cL);
        R2(A, B, C, D, X(1), 4, 0xa4beea44L);
        R2(D, A, B, C, X(4), 11, 0x4bdecfa9L);
        R2(C, D, A, B, X(7), 16, 0xf6bb4b60L);
        R2(B, C, D, A, X(10), 23, 0xbebfbc70L);
        R2(A, B, C, D, X(13), 4, 0x289b7ec6L);
        R2(D, A, B, C, X(0), 11, 0xeaa127faL);
        R2(C, D, A, B, X(3), 16, 0xd4ef3085L);
        R2(B, C, D, A, X(6), 23, 0x04881d05L);
        R2(A, B, C, D, X(9), 4, 0xd9d4d039L);
        R2(D, A, B, C, X(12), 11, 0xe6db99e5L);
        R2(C, D, A, B, X(15), 16, 0x1fa27cf8L);
        R2(B, C, D, A, X(2), 23, 0xc4ac5665L);
        /* Round 3 */
        R3(A, B, C, D, X(0), 6, 0xf4292244L);
        R3(D, A, B, C, X(7), 10, 0x432aff97L);
        R3(C, D, A, B, X(14), 15, 0xab9423a7L);
        R3(B, C, D, A, X(5), 21, 0xfc93a039L);
        R3(A, B, C, D, X(12), 6, 0x655b59c3L);
        R3(D, A, B, C, X(3), 10, 0x8f0ccc92L);
        R3(C, D, A, B, X(10), 15, 0xffeff47dL);
        R3(B, C, D, A, X(1), 21, 0x85845dd1L);
        R3(A, B, C, D, X(8), 6, 0x6fa87e4fL);
        R3(D, A, B, C, X(15), 10, 0xfe2ce6e0L);
        R3(C, D, A, B, X(6), 15, 0xa3014314L);
        R3(B, C, D, A, X(13), 21, 0x4e0811a1L);
        R3(A, B, C, D, X(4), 6, 0xf7537e82L);
        R3(D, A, B, C, X(11), 10, 0xbd3af235L);
        R3(C, D, A, B, X(2), 15, 0x2ad7d2bbL);
        R3(B, C, D, A, X(9), 21, 0xeb86d391L);

        A = c->A += A;
        B = c->B += B;
        C = c->C += C;
        D = c->D += D;
    }
}
#endif
```

md5_local.h

```c
#define F(b,c,d)        ((((c) ^ (d)) & (b)) ^ (d))
#define G(b,c,d)        ((((b) ^ (c)) & (d)) ^ (c))
#define H(b,c,d)        ((b) ^ (c) ^ (d))
#define I(b,c,d)        (((~(d)) | (b)) ^ (c))

#define R0(a,b,c,d,k,s,t) { \
        a+=((k)+(t)+F((b),(c),(d))); \
        a=ROTATE(a,s); \
        a+=b; };

#define R1(a,b,c,d,k,s,t) { \
        a+=((k)+(t)+G((b),(c),(d))); \
        a=ROTATE(a,s); \
        a+=b; };

#define R2(a,b,c,d,k,s,t) { \
        a+=((k)+(t)+H((b),(c),(d))); \
        a=ROTATE(a,s); \
        a+=b; };

#define R3(a,b,c,d,k,s,t) { \
        a+=((k)+(t)+I((b),(c),(d))); \
        a=ROTATE(a,s); \
        a+=b; };
```

