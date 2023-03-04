---
title: SHA1
date: 2022-02-23 20:00:00
tags:
  - CryptoDetect
  - algorithm
  - reverse
categories:
  - [CryptoDetect,algorithm]
keywords:
description:
top_img:
comments:
cover:
toc:
toc_number:
copyright:
mathjax:
katex: true
hide:
---

## sha1

与md5类似，差异用`标注`

### 算法

- 输入：任意长的消息，512 比特长的分组。`原始报文长度不能超过2的64次方`
- 输出：`160` 比特的消息摘要

整体流程如下：

- 首先填充原始消息使得对512求余的结果等于448，然后64位记录其长度。
- 512bit一组分为n组。每组中32bit为一段，分为16段
- `将上述16段扩充到80段`
- 对每一组，循环4轮运算，得到新的A,B,C,D,E作为下一组的初始值
- 最后得到的A,B,C,D,E加上第n组原来A,B,C,D,E的值（即计算前的值）
- 按照地址的顺序从低到高打印对应的A,B,C,D,E值，就是所求的sha1值。

#### 填充

如果输入信息的长度(bit)对512求余的结果不等于448，就需要填充使得对512求余的结果等于448。填充的方法是填充一个1和n个0。填充完后，信息的长度就为N*512+448(bit)。

然后用64位来存储填充前信息长度。这64位加在第一步结果的后面，这样信息长度就变为N\*512+448+64=(N+1)*512位 



比如，需要加密消息"gnubd"，最后被填充为

**小端字节序存储**

```
67 6E 75 62 64 80 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 28 00 00 00 00 00 00 00 
```

最后64位（8字节)为0x28（40），消息内容为40位（5字节）。

#### 数据处理

要使A,B,C,D,`E`在内存中的显示情况：

```c
A = 0x01234567
B = 0x89ABCDEF
C = 0xFEDCBA89
D = 0x76543210
E = 0xF0E1D2C3
```

程序定义应为

```c
A = 0x67452301;
B = 0xEFCDAB89;
C = 0x98BADCFE;
D = 0x10325476;
E = 0xC3D2E1F0;
```

每512位（64字节）为1段可以分成n段，（n大于等于1），对于每一段信息（512位，64字节）又划分成16小段（每段32位，4个字节，用M表示）

`将16段Mt(0-15)扩充到80段Wt(0-79)：`
$$
W t = M t , 当0≤t≤15
\\ W t = ( W(t-3) ⊕ W(t-8)⊕ W(t-14)⊕ W(t-16 ) <<< 1, 当16≤t≤79
$$


SHA1的4轮运算，每轮包括20个步骤，共80个步骤使用同一个操作程序
$$
A,B,C,D,E←[(A<<<5)+ ft(B,C,D)+E+Wt+Kt],A,(B<<<30),C,D
$$
其中 ft(B,C,D)为逻辑函数，Wt为子明文分组W[t]，Kt为固定常数。操作表示具体含义如下

- 将[(A<<<5)+ ft(B,C,D)+E+Wt+Kt]的结果赋值给变量A
- 将A的初始值赋值给B
- 将B初始值循环左移30位赋值给
- 将C初始值赋给D
- 将D初始值赋给E

四轮运算的逻辑函数如下表所示

| 轮次 | 步骤    | 函数定义                      |
| ---- | ------- | ----------------------------- |
| 1    | 0≤t≤19  | ft(B,C,D)=(B&C)\|(~B&D)       |
| 2    | 20≤t≤39 | ft(B,C,D)=B⊕C⊕D               |
| 3    | 40≤t≤59 | ft(B,C,D)=(B&C)\|(B&D)\|(C&D) |
| 4    | 60≤t≤79 | ft(B,C,D)=B⊕C⊕D               |

固定常数Ki的取值如下表

| 轮   | 步骤    | 函数定义      |
| ---- | ------- | ------------- |
| 1    | 0≤t≤19  | *K*t=5A827999 |
| 2    | 20≤t≤39 | *K*t=6ED9EBA1 |
| 3    | 40≤t≤59 | *K*t=8F188CDC |
| 4    | 60≤t≤79 | *K*t=CA62C1D6 |

##### 例子

假设W[1]=0x12345678，此时变量的值分别为A=0x67452301、B=0xEFCDAB89、C=0x98BADCFE、D=0x10325476、E=0xC3D2E1F0

那么第1轮第1步的运算过程如下。

1. 将链接变量A循环左移5位，得到的结果为：0xE8A4602C。

2. 将B，C，D经过相应的逻辑函数：

(B&C)|(~B&D)=(0xEFCDAB89&0x98BADCFE)|(~0xEFCDAB89&0x10325476)=0x98BADCFE

3. 将第1步，第2步的结果与E，W[1]，和K[1]相加得：

0xE8A4602C+0x98BADCFE+0xC3D2E1F0+0x12345678+0x5A827999=0xB1E8EF2B

4. 将B循环左移30位得：(B<<<30)=0x7BF36AE2。

5. 将第3步结果赋值给A，A（这里是指A的原始值）赋值给B，步骤4的结果赋值给C，C的原始值赋值给D，D的原始值赋值给E。

6. 最后得到第1轮第1步的结果：

A = 0xB1E8EF2B

B = 0x67452301

C = 0x7BF36AE2

D = 0x98BADCFE

E = 0x10325476

按照这种方法，将80个步骤进行完毕。

第四轮最后一个步骤的A，B，C，D，E输出，将分别与原始值A′，B′，C′，D′，E′中的数值求和运算。其结果将作为输入成为下一个512位明文分组的A，B，C，D，E，当最后一个明文分组计算完成以后，A，B，C，D，E中的数据就是最后散列函数值。







### signature

DSL定义如下

```
IDENTIFIER SHA1

sub_40:OPAQUE;
sub_45:OPAQUE;
sub_51:OPAQUE;
sub_52:OPAQUE;
sub_53:OPAQUE;
sub_54:OPAQUE;
sub_55:OPAQUE;
sub_56:OPAQUE;
sub_58:OPAQUE;
sub_59:OPAQUE;
sub_61:OPAQUE;
sub_62:OPAQUE;
sub_64:OPAQUE;
sub_65:OPAQUE;
sub_66:OPAQUE;
sub_67:OPAQUE;
sub_68:OPAQUE;
sub_69:OPAQUE;
sub_70:OPAQUE;
sub_71:OPAQUE;
sub_73:OPAQUE;
sub_75:OPAQUE;
sub_77:OPAQUE;
sub_79:OPAQUE;
sub_81:OPAQUE;
sub_82:(OPAQUE+ROTATE(sub_73,27)+XOR(AND(XOR(sub_75,sub_77),sub_79),sub_77)+sub_81+1518500249);
sub_83:ROTATE(sub_79,2);
sub_84:(OPAQUE+ROTATE(sub_82,27)+XOR(AND(XOR(sub_83,sub_75),sub_73),sub_75)+sub_77+1518500249);
sub_85:ROTATE(sub_73,2);
sub_86:(OPAQUE+ROTATE(sub_84,27)+XOR(AND(XOR(sub_85,sub_83),sub_82),sub_83)+sub_75+1518500249);
sub_87:ROTATE(sub_82,2);
sub_88:(OPAQUE+ROTATE(sub_86,27)+XOR(AND(XOR(sub_87,sub_85),sub_84),sub_85)+sub_83+1518500249);
sub_89:ROTATE(sub_84,2);
sub_90:(OPAQUE+ROTATE(sub_88,27)+XOR(AND(XOR(sub_89,sub_87),sub_86),sub_87)+sub_85+1518500249);
sub_91:ROTATE(sub_90,2);
sub_92:ROTATE(sub_88,2);
sub_93:ROTATE(sub_86,2);
sub_94:(OPAQUE+ROTATE(sub_90,27)+XOR(AND(XOR(sub_93,sub_89),sub_88),sub_89)+sub_87+1518500249);
sub_95:(OPAQUE+ROTATE(sub_94,27)+XOR(AND(XOR(sub_92,sub_93),sub_90),sub_93)+sub_89+1518500249);
sub_96:(OPAQUE+ROTATE(sub_95,27)+XOR(AND(XOR(sub_91,sub_92),sub_94),sub_92)+sub_93+1518500249);
sub_97:ROTATE(sub_94,2);
sub_98:(OPAQUE+ROTATE(sub_96,27)+XOR(AND(XOR(sub_97,sub_91),sub_95),sub_91)+sub_92+1518500249);
sub_99:ROTATE(sub_95,2);
sub_100:(OPAQUE+ROTATE(sub_98,27)+XOR(AND(XOR(sub_99,sub_97),sub_96),sub_97)+sub_91+1518500249);
sub_101:ROTATE(sub_96,2);
sub_102:(OPAQUE+ROTATE(sub_100,27)+XOR(AND(XOR(sub_101,sub_99),sub_98),sub_99)+sub_97+1518500249);
sub_103:ROTATE(sub_98,2);
sub_104:(OPAQUE+ROTATE(sub_102,27)+XOR(AND(XOR(sub_103,sub_101),sub_100),sub_101)+sub_99+1518500249);
sub_105:ROTATE(sub_100,2);
sub_106:(OPAQUE+ROTATE(sub_104,27)+XOR(AND(XOR(sub_105,sub_103),sub_102),sub_103)+sub_101+1518500249);
sub_107:ROTATE(sub_102,2);
sub_108:(OPAQUE+ROTATE(sub_106,27)+XOR(AND(XOR(sub_107,sub_105),sub_104),sub_105)+sub_103+1518500249);
sub_109:ROTATE(sub_104,2);
sub_110:(OPAQUE+ROTATE(sub_108,27)+XOR(AND(XOR(sub_109,sub_107),sub_106),sub_107)+sub_105+1518500249);
sub_111:ROTATE(sub_106,2);
sub_112:(OPAQUE+ROTATE(sub_110,27)+XOR(AND(XOR(sub_111,sub_109),sub_108),sub_109)+sub_107+1518500249);
sub_113:ROTATE(sub_108,2);
sub_114:(OPAQUE+ROTATE(sub_112,27)+XOR(AND(XOR(sub_113,sub_111),sub_110),sub_111)+sub_109+1518500249);
sub_115:ROTATE(sub_110,2);
sub_116:(OPAQUE+ROTATE(sub_114,27)+XOR(AND(XOR(sub_115,sub_113),sub_112),sub_113)+sub_111+1518500249);
sub_117:ROTATE(sub_112,2);
sub_118:(OPAQUE+ROTATE(sub_116,27)+XOR(AND(XOR(sub_117,sub_115),sub_114),sub_115)+sub_113+1518500249);
sub_119:ROTATE(sub_114,2);
sub_120:(OPAQUE+ROTATE(sub_118,27)+XOR(AND(XOR(sub_119,sub_117),sub_116),sub_117)+sub_115+1518500249);
sub_121:ROTATE(sub_116,2);
sub_122:(OPAQUE+ROTATE(sub_120,27)+XOR(sub_118,sub_121,sub_119)+sub_117+1859775393);
sub_123:ROTATE(sub_118,2);
sub_124:(OPAQUE+ROTATE(sub_122,27)+XOR(sub_120,sub_123,sub_121)+sub_119+1859775393);
sub_125:ROTATE(sub_120,2);
sub_126:(OPAQUE+ROTATE(sub_124,27)+XOR(sub_122,sub_125,sub_123)+sub_121+1859775393);
sub_127:ROTATE(sub_122,2);
sub_128:(OPAQUE+ROTATE(sub_126,27)+XOR(sub_124,sub_127,sub_125)+sub_123+1859775393);
sub_129:ROTATE(sub_124,2);
sub_130:(OPAQUE+ROTATE(sub_128,27)+XOR(sub_126,sub_129,sub_127)+sub_125+1859775393);
sub_131:ROTATE(sub_126,2);
sub_132:(OPAQUE+ROTATE(sub_130,27)+XOR(sub_128,sub_131,sub_129)+sub_127+1859775393);
sub_133:ROTATE(sub_128,2);
sub_134:(OPAQUE+ROTATE(sub_132,27)+XOR(sub_130,sub_133,sub_131)+sub_129+1859775393);
sub_135:ROTATE(sub_130,2);
sub_136:(OPAQUE+ROTATE(sub_134,27)+XOR(sub_132,sub_135,sub_133)+sub_131+1859775393);
sub_137:ROTATE(sub_132,2);
sub_138:(OPAQUE+ROTATE(sub_136,27)+XOR(sub_134,sub_137,sub_135)+sub_133+1859775393);
sub_139:ROTATE(sub_134,2);
sub_140:(OPAQUE+ROTATE(sub_138,27)+XOR(sub_136,sub_139,sub_137)+sub_135+1859775393);
sub_141:ROTATE(sub_136,2);
sub_142:(OPAQUE+ROTATE(sub_140,27)+XOR(sub_138,sub_141,sub_139)+sub_137+1859775393);
sub_143:ROTATE(sub_138,2);
sub_144:(OPAQUE+ROTATE(sub_142,27)+XOR(sub_140,sub_143,sub_141)+sub_139+1859775393);
sub_145:ROTATE(sub_140,2);
sub_146:(OPAQUE+ROTATE(sub_144,27)+XOR(sub_142,sub_145,sub_143)+sub_141+1859775393);
sub_147:ROTATE(sub_142,2);
sub_148:(sub_40+ROTATE(sub_146,27)+XOR(sub_144,sub_147,sub_145)+sub_143+1859775393);
sub_149:ROTATE(sub_144,2);
sub_150:(OPAQUE+ROTATE(sub_148,27)+XOR(sub_146,sub_149,sub_147)+sub_145+1859775393);
sub_151:ROTATE(sub_146,2);
sub_152:(sub_51+ROTATE(sub_150,27)+XOR(sub_148,sub_151,sub_149)+sub_147+1859775393);
sub_153:ROTATE(sub_148,2);
sub_154:(sub_45+ROTATE(sub_152,27)+XOR(sub_150,sub_153,sub_151)+sub_149+1859775393);
sub_155:ROTATE(sub_150,2);
sub_156:(OPAQUE+ROTATE(sub_154,27)+XOR(sub_152,sub_155,sub_153)+sub_151+1859775393);
sub_157:ROTATE(sub_152,2);
sub_158:(sub_52+ROTATE(sub_156,27)+XOR(sub_154,sub_157,sub_155)+sub_153+1859775393);
sub_159:ROTATE(sub_154,2);
sub_160:(sub_55+ROTATE(sub_158,27)+XOR(sub_156,sub_159,sub_157)+sub_155+1859775393);
sub_161:ROTATE(sub_158,2);
sub_162:ROTATE(sub_156,2);
sub_163:(OR(AND(OR(sub_158,sub_162),sub_159),AND(sub_158,sub_162))+OPAQUE+ROTATE(sub_160,27)+sub_157+2400959708);
sub_164:(OR(AND(OR(sub_160,sub_161),sub_162),AND(sub_160,sub_161))+sub_53+ROTATE(sub_163,27)+sub_159+2400959708);
sub_165:ROTATE(XOR(sub_69,sub_53,sub_51,sub_40),31);
sub_166:ROTATE(XOR(sub_70,sub_165,sub_66,sub_53),31);
sub_167:ROTATE(XOR(sub_165,sub_54,sub_52,sub_45),31);
sub_168:ROTATE(XOR(sub_166,sub_167,sub_69,sub_54),31);
sub_169:ROTATE(XOR(sub_167,sub_56,sub_53,sub_55),31);
sub_170:ROTATE(XOR(sub_168,sub_169,sub_165,sub_56),31);
sub_171:ROTATE(XOR(sub_169,sub_59,sub_54,sub_58),31);
sub_172:ROTATE(XOR(sub_170,sub_171,sub_167,sub_59),31);
sub_173:ROTATE(XOR(sub_71,sub_166,sub_67,sub_165),31);
sub_174:ROTATE(XOR(sub_171,sub_62,sub_56,sub_61),31);
sub_175:ROTATE(XOR(sub_174,sub_65,sub_59,sub_64),31);
sub_176:ROTATE(sub_163,2);
sub_177:ROTATE(sub_160,2);
sub_178:(OR(AND(OR(sub_163,sub_177),sub_161),AND(sub_163,sub_177))+sub_58+ROTATE(sub_164,27)+sub_162+2400959708);
sub_179:(OR(AND(OR(sub_164,sub_176),sub_177),AND(sub_164,sub_176))+sub_66+ROTATE(sub_178,27)+sub_161+2400959708);
sub_180:ROTATE(sub_178,2);
sub_181:ROTATE(sub_164,2);
sub_182:(OR(AND(OR(sub_178,sub_181),sub_176),AND(sub_178,sub_181))+sub_54+ROTATE(sub_179,27)+sub_177+2400959708);
sub_183:(OR(AND(OR(sub_179,sub_180),sub_181),AND(sub_179,sub_180))+sub_61+ROTATE(sub_182,27)+sub_176+2400959708);
sub_184:ROTATE(sub_182,2);
sub_185:ROTATE(sub_179,2);
sub_186:(OR(AND(OR(sub_182,sub_185),sub_180),AND(sub_182,sub_185))+sub_69+ROTATE(sub_183,27)+sub_181+2400959708);
sub_187:(OR(AND(OR(sub_183,sub_184),sub_185),AND(sub_183,sub_184))+sub_56+ROTATE(sub_186,27)+sub_180+2400959708);
sub_188:ROTATE(sub_186,2);
sub_189:ROTATE(sub_183,2);
sub_190:(OR(AND(OR(sub_186,sub_189),sub_184),AND(sub_186,sub_189))+sub_64+ROTATE(sub_187,27)+sub_185+2400959708);
sub_191:(OR(AND(OR(sub_187,sub_188),sub_189),AND(sub_187,sub_188))+sub_165+ROTATE(sub_190,27)+sub_184+2400959708);
sub_192:ROTATE(sub_190,2);
sub_193:ROTATE(sub_187,2);
sub_194:(OR(AND(OR(sub_190,sub_193),sub_188),AND(sub_190,sub_193))+sub_59+ROTATE(sub_191,27)+sub_189+2400959708);
sub_195:(OR(AND(OR(sub_191,sub_192),sub_193),AND(sub_191,sub_192))+sub_67+ROTATE(sub_194,27)+sub_188+2400959708);
sub_196:ROTATE(sub_194,2);
sub_197:ROTATE(sub_191,2);
sub_198:(OR(AND(OR(sub_194,sub_197),sub_192),AND(sub_194,sub_197))+sub_167+ROTATE(sub_195,27)+sub_193+2400959708);
sub_199:(OR(AND(OR(sub_195,sub_196),sub_197),AND(sub_195,sub_196))+sub_62+ROTATE(sub_198,27)+sub_192+2400959708);
sub_200:ROTATE(sub_198,2);
sub_201:ROTATE(sub_195,2);
sub_202:(OR(AND(OR(sub_198,sub_201),sub_196),AND(sub_198,sub_201))+sub_70+ROTATE(sub_199,27)+sub_197+2400959708);
sub_203:(OR(AND(OR(sub_199,sub_200),sub_201),AND(sub_199,sub_200))+sub_169+ROTATE(sub_202,27)+sub_196+2400959708);
sub_204:ROTATE(sub_202,2);
sub_205:ROTATE(sub_199,2);
sub_206:(OR(AND(OR(sub_202,sub_205),sub_200),AND(sub_202,sub_205))+sub_65+ROTATE(sub_203,27)+sub_201+2400959708);
sub_207:(OR(AND(OR(sub_203,sub_204),sub_205),AND(sub_203,sub_204))+sub_166+ROTATE(sub_206,27)+sub_200+2400959708);
sub_208:ROTATE(sub_206,2);
sub_209:ROTATE(sub_203,2);
sub_210:(OR(AND(OR(sub_206,sub_209),sub_204),AND(sub_206,sub_209))+sub_171+ROTATE(sub_207,27)+sub_205+2400959708);
sub_211:(OR(AND(OR(sub_207,sub_208),sub_209),AND(sub_207,sub_208))+sub_68+ROTATE(sub_210,27)+sub_204+2400959708);
sub_212:ROTATE(sub_207,2);
sub_213:(sub_168+ROTATE(sub_211,27)+XOR(sub_210,sub_212,sub_208)+sub_209+3395469782);
sub_214:ROTATE(sub_210,2);
sub_215:(sub_174+ROTATE(sub_213,27)+XOR(sub_211,sub_214,sub_212)+sub_208+3395469782);
sub_216:ROTATE(sub_211,2);
sub_217:(sub_71+ROTATE(sub_215,27)+XOR(sub_213,sub_216,sub_214)+sub_212+3395469782);
sub_218:ROTATE(sub_213,2);
sub_219:(sub_170+ROTATE(sub_217,27)+XOR(sub_215,sub_218,sub_216)+sub_214+3395469782);
sub_220:ROTATE(sub_215,2);
sub_221:(sub_175+ROTATE(sub_219,27)+XOR(sub_217,sub_220,sub_218)+sub_216+3395469782);
sub_222:ROTATE(sub_217,2);
sub_223:(sub_173+ROTATE(sub_221,27)+XOR(sub_219,sub_222,sub_220)+sub_218+3395469782);
sub_224:ROTATE(sub_219,2);
sub_225:(sub_172+ROTATE(sub_223,27)+XOR(sub_221,sub_224,sub_222)+sub_220+3395469782);
sub_226:ROTATE(XOR(sub_172,sub_174,sub_169,sub_62),31);
sub_227:ROTATE(XOR(sub_226,sub_175,sub_171,sub_65),31);
sub_228:ROTATE(XOR(sub_175,sub_68,sub_62,sub_67),31);
sub_229:ROTATE(XOR(sub_227,sub_228,sub_174,sub_68),31);
sub_230:ROTATE(sub_221,2);
sub_231:(sub_228+ROTATE(sub_225,27)+XOR(sub_223,sub_230,sub_224)+sub_222+3395469782);
sub_232:ROTATE(sub_231,2);
sub_233:ROTATE(XOR(sub_228,sub_71,sub_65,sub_70),31);
sub_234:ROTATE(XOR(sub_173,sub_168,sub_70,sub_167),31);
sub_235:ROTATE(sub_223,2);
sub_236:(sub_234+ROTATE(sub_231,27)+XOR(sub_225,sub_235,sub_230)+sub_224+3395469782);
sub_237:ROTATE(sub_225,2);
sub_238:(sub_226+ROTATE(sub_236,27)+XOR(sub_231,sub_237,sub_235)+sub_230+3395469782);
sub_239:(sub_233+ROTATE(sub_238,27)+XOR(sub_236,sub_232,sub_237)+sub_235+3395469782);
sub_240:ROTATE(XOR(sub_233,sub_173,sub_68,sub_166),31);
sub_241:ROTATE(XOR(sub_234,sub_170,sub_166,sub_169),31);
sub_242:ROTATE(sub_236,2);
sub_243:(sub_241+ROTATE(sub_239,27)+XOR(sub_238,sub_242,sub_232)+sub_237+3395469782);
sub_244:ROTATE(sub_238,2);
sub_245:(sub_227+ROTATE(sub_243,27)+XOR(sub_239,sub_244,sub_242)+sub_232+3395469782);
sub_246:ROTATE(sub_239,2);
sub_247:(sub_240+ROTATE(sub_245,27)+XOR(sub_243,sub_246,sub_244)+sub_242+3395469782);
sub_248:ROTATE(sub_245,2);
sub_249:ROTATE(sub_243,2);
sub_250:ROTATE(sub_247,2);
sub_251:ROTATE(XOR(sub_241,sub_172,sub_168,sub_171),31);
sub_252:(sub_251+ROTATE(sub_247,27)+XOR(sub_245,sub_249,sub_246)+sub_244+3395469782);
sub_253:(sub_229+ROTATE(sub_252,27)+XOR(sub_247,sub_248,sub_249)+sub_246+3395469782);
sub_254:ROTATE(sub_253,2);
sub_255:ROTATE(XOR(sub_240,sub_234,sub_71,sub_168),31);
sub_256:ROTATE(XOR(sub_229,sub_233,sub_175,sub_71),31);
sub_257:(sub_255+ROTATE(sub_253,27)+XOR(sub_252,sub_250,sub_248)+sub_249+3395469782);
sub_258:ROTATE(sub_252,2);
sub_259:(ROTATE(XOR(sub_251,sub_226,sub_170,sub_174),31)+ROTATE(sub_257,27)+XOR(sub_253,sub_258,sub_250)+sub_248+3395469782);
sub_260:ROTATE(sub_259,27);
sub_261:XOR(sub_257,sub_254,sub_258);
sub_262:ROTATE(sub_257,2);
sub_263:(sub_77+sub_262);
sub_264:(sub_73+ROTATE(XOR(sub_255,sub_241,sub_173,sub_170),31)+ROTATE((sub_256+sub_260+sub_261+sub_250+3395469782),27)+XOR(sub_259,sub_262,sub_254)+sub_258+3395469782);
sub_265:(sub_81+sub_254);
sub_266:(sub_75+ROTATE(sub_259,2));
sub_267:(sub_79+sub_256+sub_260+sub_261+sub_250+3395469782);

```



### 测试

libcrypto.so.1.1(openssl)

其中和sha1有关的函数如下

![image-20220223192555863](sha1.assets/image-20220223192555863.png)

利用where's crypto分析

![image-20220223191837393](sha1.assets/image-20220223191837393.png)

反汇编SHA1_Update,主要还是调用了sha1_block_data_order

![image-20220223200517936](sha1.assets/image-20220223200517936.png)

md5_block_data_order反汇编后

![image-20220223200537067](sha1.assets/image-20220223200537067.png)

与signature中对应关系如下

![image-20220223213829056](sha1.assets/image-20220223213829056.png)

其余均类似



`openssl中相关函数源码通过perl生成汇编代码`
