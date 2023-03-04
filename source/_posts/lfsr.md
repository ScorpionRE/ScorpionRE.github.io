---
title: LFSR
date: 2022-02-24 18:00:00
tags:
  - CryptoDetect
  - reverse
  - crypto
  - algorithm
categories:
  - [CryptoDetect,algorithm]
keywords:
description:
top_img:
comments:
cover: https://scorpionre.github.io/2022/02/24/lfsr/lfsr.assets/image-20220224160744696.png
toc:
toc_number:
copyright:
mathjax:
katex: true
hide:
---

## lfsr

### 算法

假设R为一个（N）LFSR，对于每一轮，使用反馈函数L从R中的比特位（子集）生成一个新的比特位。如果L是线性的，例如异或，我们把R称为一个LFSR。反之，如果L是非线性的，R就是一个NLFSR。寄存器R中的所有位都向左移动一个位置，忽略最高位，新产生的位被放在最低位且被输出。

所以
$$
对每轮i\in[0,\ 1,\ ...,\ n ]
\\ R_{i+1} = (R_i << 1)\ xor\ L(R_i)
\\ output_i = (R_i)
$$
![image-20220224160744696](lfsr.assets/image-20220224160744696.png)



### signature

![image-20220217205141000](lfsr.assets/image-20220217205141000.png)

TRANSIENT用于表示从表达式生成的DFG节点不是很重要，图清除过程中可以简化。

OPAQUE<o1>中 o1为clamp-label（可选）：为节点类型命名。与任何其他类型的节点进行比较都为真，并增加了所有带有相同类型标签的 opaque 必须映射到相同类型节点的限制（type clamping)

其中VARIANT C最常用。每轮迭代左移一位，通过反馈函数L生成一个新的比特位并放到



### 测试

用论文给出的a5算法样本进行：

https://github.com/marcelmaatkamp/gnuradio-osmocom-gmr/blob/master/src/l1/a5.c

![image-20220224161436546](lfsr.assets/image-20220224161436546.png)

但是无法识别



论文中给出和LFSR相关的图，但是不知道是什么样本

![image-20220224153901121](lfsr.assets/image-20220224153901121.png)

