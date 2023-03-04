---
title: DES
date: 2022-01-14 20:00:00
tags: 
  - crypto
  - algorithm
  - CryptoDetect
categories:
  - [CryptoDetect,algorithm]
keywords:
	- DES
description:
top_img:
comments:
cover:  https://scorpionre.github.io/2022/01/14/DES/DES.assets/image-20220112114938574.png
toc:
toc_number:
copyright:
mathjax: 
katex:  true
hide:
---

# DES

### 算法原理

- 输入 64 位。
- 输出 64 位。

- 密钥长64位，密钥事实上是56位参与DES运算（第8、16、24、32、40、48、56、64位是校验位， 使得每个密钥都有奇数个1）



算法流程图如下：

![image-20220112114938574](DES.assets/image-20220112114938574.png)



#### 密钥生成

1. 选择置换：不考虑每个字节的第8位，DES的密钥由64位减至56位，每个字节的第8位作为奇偶校验位

   ![image-20220112132946201](DES.assets/image-20220112132946201.png)

2. 循环移位：根据轮数，将两部分分别循环左移1位或2位。

   ![image-20220112133041770](DES.assets/image-20220112133041770.png)

3. 置换：移位后，从56位中选出48位

   ![image-20220112133518482](DES.assets/image-20220112133518482.png)

#### 加密

1. IP置换：将输入的64位数据块按位重新组合，并把输出分为L0、R0两部分，每部分各长32位。比如以下置换规则表，表示此位置的数据在原数据中的位置，即原数据块的第58位放到新数据的第1位

![image-20220112133802349](DES.assets/image-20220112133802349.png)

2. Feistel

   ![image-20220112134612622](DES.assets/image-20220112134612622.png)

   - E（扩张置换）：将32位的半块R0扩展到48位，其输出包括8个6位的块，每块包含4位对应的输入位，加上两个邻接的块中紧邻的位。然后与子密钥异或。

     目的有两个：生成与密钥相同长度的数据以进行异或运算；提供更长的结果，在后续的替代运算中可以进行压缩

     ![image-20220112133737845](DES.assets/image-20220112133737845.png)

   - S盒：替代运算。（非线性，提供安全性）每个S盒将6位输入变为4位输出。给定输入后，输出行由外侧 2 位确定，列由内侧的 4 位确定，例如“011011”的输入的外侧位为“01”，内侧位为“1101”，而每张表的第一行为“00”，第一列为“0000”，输出S盒的第2行，第14列

     |        | x0000x | x0001x | x0010x | x0011x | x0100x | x0101x | x0110x | x0111x | x1000x | x1001x | x1010x | x1011x | x1100x | x1101x | x1110x | x1111x |
     | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
     | 0yyyy0 | 14     | 4      | 13     | 1      | 2      | 15     | 11     | 8      | 3      | 10     | 6      | 12     | 5      | 9      | 0      | 7      |
     | 0yyyy1 | 0      | 15     | 7      | 4      | 14     | 2      | 13     | 1      | 10     | 6      | 12     | 11     | 9      | 5      | 3      | 8      |
     | 1yyyy0 | 4      | 1      | 14     | 8      | 13     | 6      | 2      | 11     | 15     | 12     | 9      | 7      | 3      | 10     | 5      | 0      |
     | 1yyyy1 | 15     | 12     | 8      | 2      | 4      | 9      | 1      | 7      | 5      | 11     | 3      | 14     | 10     | 0      | 6      | 13     |

   - P置换：将32位的半块数据重新排列

     ![image-20220112134549104](DES.assets/image-20220112134549104.png)

3. FP置换（IP置换的逆过程）

   ![image-20220112134705510](DES.assets/image-20220112134705510.png)

### signature

Feistel网络结构

![image-20220113142045280](DES.assets/image-20220113142045280.png)

适用的加密算法

![image-20220112135218512](DES.assets/image-20220112135218512.png)

DSL描述

![image-20220112135026479](DES.assets/image-20220112135026479.png)

### 效果

libcrypto.so.1.1(openssl)

![image-20220112165741033](DES.assets/image-20220112165741033.png)



openssl中查看源代码

```c
//des_local.h
# define D_ENCRYPT(LL,R,S) { \
        LOAD_DATA_tmp(R,S,u,t,E0,E1); \
        t=ROTATE(t,4); \
        LL^= \
            DES_SPtrans[0][(u>> 2L)&0x3f]^ \
            DES_SPtrans[2][(u>>10L)&0x3f]^ \
            DES_SPtrans[4][(u>>18L)&0x3f]^ \
            DES_SPtrans[6][(u>>26L)&0x3f]^ \
            DES_SPtrans[1][(t>> 2L)&0x3f]^ \
            DES_SPtrans[3][(t>>10L)&0x3f]^ \
            DES_SPtrans[5][(t>>18L)&0x3f]^ \
            DES_SPtrans[7][(t>>26L)&0x3f]; }
        
//des_enc.c
void DES_encrypt2(DES_LONG *data, DES_key_schedule *ks, int enc)
{
    register DES_LONG l, r, t, u;
    register DES_LONG *s;

    r = data[0];
    l = data[1];

    /*
     * Things have been modified so that the initial rotate is done outside
     * the loop.  This required the DES_SPtrans values in sp.h to be rotated
     * 1 bit to the right. One perl script later and things have a 5% speed
     * up on a sparc2. Thanks to Richard Outerbridge for pointing this out.
     */
    /* clear the top bits on machines with 8byte longs */
    r = ROTATE(r, 29) & 0xffffffffL;
    l = ROTATE(l, 29) & 0xffffffffL;

    s = ks->ks->deslong;
    /*
     * I don't know if it is worth the effort of loop unrolling the inner
     * loop
     */
    if (enc) {
        D_ENCRYPT(l, r, 0);     /* 1 */
        D_ENCRYPT(r, l, 2);     /* 2 */
        D_ENCRYPT(l, r, 4);     /* 3 */
        D_ENCRYPT(r, l, 6);     /* 4 */
        D_ENCRYPT(l, r, 8);     /* 5 */
        D_ENCRYPT(r, l, 10);    /* 6 */
        D_ENCRYPT(l, r, 12);    /* 7 */
        D_ENCRYPT(r, l, 14);    /* 8 */
        D_ENCRYPT(l, r, 16);    /* 9 */
        D_ENCRYPT(r, l, 18);    /* 10 */
        D_ENCRYPT(l, r, 20);    /* 11 */
        D_ENCRYPT(r, l, 22);    /* 12 */
        D_ENCRYPT(l, r, 24);    /* 13 */
        D_ENCRYPT(r, l, 26);    /* 14 */
        D_ENCRYPT(l, r, 28);    /* 15 */
        D_ENCRYPT(r, l, 30);    /* 16 */
    } else {
        D_ENCRYPT(l, r, 30);    /* 16 */
        D_ENCRYPT(r, l, 28);    /* 15 */
        D_ENCRYPT(l, r, 26);    /* 14 */
        D_ENCRYPT(r, l, 24);    /* 13 */
        D_ENCRYPT(l, r, 22);    /* 12 */
        D_ENCRYPT(r, l, 20);    /* 11 */
        D_ENCRYPT(l, r, 18);    /* 10 */
        D_ENCRYPT(r, l, 16);    /* 9 */
        D_ENCRYPT(l, r, 14);    /* 8 */
        D_ENCRYPT(r, l, 12);    /* 7 */
        D_ENCRYPT(l, r, 10);    /* 6 */
        D_ENCRYPT(r, l, 8);     /* 5 */
        D_ENCRYPT(l, r, 6);     /* 4 */
        D_ENCRYPT(r, l, 4);     /* 3 */
        D_ENCRYPT(l, r, 2);     /* 2 */
        D_ENCRYPT(r, l, 0);     /* 1 */
    }
    /* rotate and clear the top bits on machines with 8byte longs */
    data[0] = ROTATE(l, 3) & 0xffffffffL;
    data[1] = ROTATE(r, 3) & 0xffffffffL;
    l = r = t = u = 0;
}

void DES_encrypt3(DES_LONG *data, DES_key_schedule *ks1,
                  DES_key_schedule *ks2, DES_key_schedule *ks3)
{
    register DES_LONG l, r;

    l = data[0];
    r = data[1];
    IP(l, r);
    data[0] = l;
    data[1] = r;
    DES_encrypt2((DES_LONG *)data, ks1, DES_ENCRYPT);
    DES_encrypt2((DES_LONG *)data, ks2, DES_DECRYPT);
    DES_encrypt2((DES_LONG *)data, ks3, DES_ENCRYPT);
    l = data[0];
    r = data[1];
    FP(r, l);
    data[0] = l;
    data[1] = r;
}
void DES_ede3_cbc_encrypt(const unsigned char *input, unsigned char *output,
                          long length, DES_key_schedule *ks1,
                          DES_key_schedule *ks2, DES_key_schedule *ks3,
                          DES_cblock *ivec, int enc)
{
    register DES_LONG tin0, tin1;
    register DES_LONG tout0, tout1, xor0, xor1;
    register const unsigned char *in;
    unsigned char *out;
    register long l = length;
    DES_LONG tin[2];
    unsigned char *iv;

    in = input;
    out = output;
    iv = &(*ivec)[0];

    if (enc) {
        c2l(iv, tout0);
        c2l(iv, tout1);
        for (l -= 8; l >= 0; l -= 8) {
            c2l(in, tin0);
            c2l(in, tin1);
            tin0 ^= tout0;
            tin1 ^= tout1;

            tin[0] = tin0;
            tin[1] = tin1;
            DES_encrypt3((DES_LONG *)tin, ks1, ks2, ks3);
            tout0 = tin[0];
            tout1 = tin[1];

            l2c(tout0, out);
            l2c(tout1, out);
        }
        if (l != -8) {
            c2ln(in, tin0, tin1, l + 8);
            tin0 ^= tout0;
            tin1 ^= tout1;

            tin[0] = tin0;
            tin[1] = tin1;
            DES_encrypt3((DES_LONG *)tin, ks1, ks2, ks3);
            tout0 = tin[0];
            tout1 = tin[1];

            l2c(tout0, out);
            l2c(tout1, out);
        }
        iv = &(*ivec)[0];
        l2c(tout0, iv);
        l2c(tout1, iv);
    } else {
        register DES_LONG t0, t1;

        c2l(iv, xor0);
        c2l(iv, xor1);
        for (l -= 8; l >= 0; l -= 8) {
            c2l(in, tin0);
            c2l(in, tin1);

            t0 = tin0;
            t1 = tin1;

            tin[0] = tin0;
            tin[1] = tin1;
            DES_decrypt3((DES_LONG *)tin, ks1, ks2, ks3);
            tout0 = tin[0];
            tout1 = tin[1];

            tout0 ^= xor0;
            tout1 ^= xor1;
            l2c(tout0, out);
            l2c(tout1, out);
            xor0 = t0;
            xor1 = t1;
        }
        if (l != -8) {
            c2l(in, tin0);
            c2l(in, tin1);

            t0 = tin0;
            t1 = tin1;

            tin[0] = tin0;
            tin[1] = tin1;
            DES_decrypt3((DES_LONG *)tin, ks1, ks2, ks3);
            tout0 = tin[0];
            tout1 = tin[1];

            tout0 ^= xor0;
            tout1 ^= xor1;
            l2cn(tout0, tout1, out, l + 8);
            xor0 = t0;
            xor1 = t1;
        }

        iv = &(*ivec)[0];
        l2c(xor0, iv);
        l2c(xor1, iv);
    }
    tin0 = tin1 = tout0 = tout1 = xor0 = xor1 = 0;
    tin[0] = tin[1] = 0;
}

#endif                          /* DES_DEFAULT_OPTIONS */

//

```

该图匹配到了feistel VARIANT B,根据常量看好像是encrypt3中的最后一部分，感觉不太明白子图和具体代码构建出来的图![image-20220113124310851](DES.assets/image-20220113124310851.png)



新的代码

![image-20220113141131473](DES.assets/image-20220113141131473.png)

```c
uint64_t raw_des(uint64_t text, uint64_t key, uint8_t mode){
    uint64_t* subkeys = (uint64_t*)malloc(sizeof(uint64_t)*16);
    generateSubkeys(key, subkeys);
    uint32_t L = 0, R = 0, SR = 0, PR = 0, tmp = 0;
    uint64_t ER = 0, output = 0, subkey = 0;
    text = ip(text);
    L = (text >> 32) & 0x00000000ffffffff;
    R = text & 0x00000000ffffffff;
    for (int i = 0; i < 16; i ++){
        ER = extend(R);
        subkey = (mode == 0) ? subkeys[i] : subkeys[15-i];
        ER = ER ^ subkey;
        SR = s(ER);
        PR = p(SR);
        tmp = R;
        R = L ^ PR;
        L = tmp;
    }
    free(subkeys);
    output = inv_ip(((uint64_t)(R) << 32 ) | (uint64_t)(L));
    return output;
}
uint64_t des_ede2(uint64_t text, uint64_t key1, uint64_t key2, uint8_t mode){
    uint64_t result;
    result = (mode == 0) ? raw_des(text, key1, 0) : raw_des(text, key1, 1);
    result = (mode == 0) ? raw_des(result, key2, 1) : raw_des(result, key2, 0);
    result = (mode == 0) ? raw_des(result, key1, 0) : raw_des(result, key1, 1);
    return result;
}
```

这个的图结构看起来就更清晰，符合feistel网络结构，也符合定义的signature的VARIANT C

![image-20220113141442031](DES.assets/image-20220113141442031.png)

每轮加密

![image-20220113142202479](DES.assets/image-20220113142202479.png)

![image-20220113142344671](DES.assets/image-20220113142344671.png)



`发现好像有的函数会代入图中，有的不会（都是BL调用函数）`
