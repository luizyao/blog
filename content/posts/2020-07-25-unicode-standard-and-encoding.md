---
title: "Unicode 标准及其常见的编码方案"
subtitle: ""
summary: "介绍 Unicode 的一些基本概念和常见的编码方案。"
date: 2020-07-25
lastmod: 2020-11-06
author: "Luiz Yao"
tags: ["unicode"]
categories: ["其它"]
draft: false
---

Unicode标准为每一个字符提供一个唯一的数字，而不用区分平台、语言等因素。

> The Unicode Standard provides a unique number for every character, no matter what platform, device, application or language.

## 基本概念
在开始学习之前，我们需要先了解本文所涉及到的一些基本概念。

**抽象字符（Abstract character）**：用于组织、控制或者表示文本数据的信息单元。

- 抽象字符没有具体的形式，不应与[图像字符（glyph）](http://unicode.org/glossary/#glyph)混淆。
- 抽象字符不一定对应于人们所认知的“字”，不应与[字素（grapheme）](http://unicode.org/glossary/#grapheme)混淆。
- 不能被Unicode标准直接编码的抽象字符通常可以通过组合字符序列来表示。

**抽象字符序列（Abstract character sequence）**：一个或多个抽象字符的有序序列。

**Unicode编码空间（Unicode codespace）**：十六进制0x0～0x10FFFF之间的整数。

**码位（Code point）**：Unicode编码空间中的任意值。

**编码字符（Coded character）**：当抽象字符被映射或者分配到编码空间中特定的码位时，它就被称为编码字符。


## 码位
码位是Unicode标准中很重要的一个概念。它的取值范围是十六进制的`0x0～0x10FFFF`，换算成十进制是0～1114111，共计1114112个。

需要注意的是，一个单一的抽象字符可能对应一个以上的码位。例如，`Ω`既可以表示大写的希腊字母Omega，码位是`U+03A9`，也可以表示物理学中的欧姆符号，码位是`U+2126`。

```bash
>>> '\u03a9'
'Ω'
>>> '\u2126'
'Ω'
```

单个抽象字符也可以由一系列码位的序列来表示。例如，`é`的码位是`U+00E9`，它也可以由小写字母`e`（码位为`U+0065`）和` ́`（*Combining Acute Accent*）（码位为`U+0301`）组合而成。

```bash
>>> '\u00e9'
'é'
>>> '\u0065\u0301'
'é'
```

在Unicode标准中，码位的表示方法通常是使用它们的十六进制，并加上`U+`前缀。

### 码位的类型
码位的分类方法多种多样。我们通过下表来阐明Unicode标准使用的七种类型和一些术语。

| 基本类型               | 简要描述                                   | 是否分配给抽象字符 | 码位范围                                               |
| ---------------------- | ------------------------------------------ | ------------------ | ------------------------------------------------------ |
| 图形（Graphic）        | 字母、标记、数字、标点符号、符号和空格     | 是                 |
| 格式（Format）         | 不可见但是影响相邻字符。包括行、段落分割符 | 是                 |
| 控制（Control）        | Unicode标准以外的协议或标准定义的用法      | 是                 | U+0000～U+001F，U+007F，U+0080～U+009F，共计65个       |
| 私用（Private-use）    | Unicode标准以外的私有协议定义的用法        | 是                 |
| 代理 (Surrogate)       | 永久预留给UTF-16编码方案                   | 不允许分配         | U+D800～U+DFFF，共计2048个                             |
| 非字符（Noncharacter） | 永久预留给内部使用                         | 否                 | U+FDD0～U+FDEF，所有以FFFE或者FFFF结尾的码位，共计66个 |
| 保留（Reserved）       | 预留给将来使用                             | 否                 |

我们需要格外注意代理（Surrogate）类型，理解他有助于我们学习UTF-16。它总共包含2048个码位，码位空间为U+D800～U+DFFF。它又引发出两个新的概念：

**高位代理（High-Surrogate）**：U+D800～U+DBFF范围内的码位，共计1024个。

**低位代理（Low-Surrogate）**：U+DC00～U+DFFF范围内的码位，共计1024个。

关于它们更多的内容，稍后结合UTF-16再讨论。


## 编码方案
在介绍具体的编码方案之前，我们先明确一些新的基本概念。

**Unicode标量值（Unicode scalar value）**：除去高位代理和低位代理之外，所有的Unicode码位，也就是U+0000～U+D7FF和U+E000～U+10FFFF范围内的码位。

**编码单元（Code unit）**：最小的比特位组合，表示用于交换或处理的编码文本单元。Unicode标准中定义，UTF-8使用8比特的编码单元，UTF-16使用16比特的编码单元，UTF-32使用32比特的编码单元。

**编码单元序列（Code unit sequence）**：一个或多个编码单元的有序序列。

### UTF-32
UTF-32将每个Unicode标量值映射成一个无符号的32比特的编码单元，数值与Unicode标量值相同，这是一种定长的编码方案。

注意，UTF-32无法编码U+D800～U+DFFF之间的码位，因为它们不属于Unicode标量值。

```bash
>>> '\ud800'.encode('utf32')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'utf-32' codec can't encode character '\ud800' in position 0: surrogates not allowed
```

### UTF-16
UTF-16将Unicode标量值中`U+0000～U+D7FF`和`U+E000～U+FFFF`范围内的码位映射成一个无符号的16比特的编码单元，数值与Unicode标量值相同。将`U+10000～U+10FFFF`范围内的码位映射成一个代理对，所谓的代理对就是上文提到的高位代理和低位代理。UTF-16是一种定长和变长兼顾的编码方案。

下表阐明了UTF-16的编码方式。

| Unicode标量值                 | UTF-16                                  |
| ----------------------------- | --------------------------------------- |
| xxxx xxxx xxxx xxxx           | xxxx xxxx xxxx xxxx                     |
| 000u uuuu xxxx xxxx xxxx xxxx | 1101 10ww wwxx xxxx 1101 11xx xxxx xxxx |

> 其中， wwww = uuuuu - 1

重点分析使用代理对的情况，一个代理对包含一个高位代理编码单元和一个低位代理编码单元，都是16比特的。其中高位代理的范围是U+D800～U+DBFF，转换成二进制，它的格式应该是1101 10xx xxxx xxxx，低位代理的范围是U+DC00～U+DFFF，转换成二进制，它的格式应该是1101 11xx xxxx xxxx。

至此，我们还剩下20位可以填充。我们将码位减去U+10000，再从右到左依次填充进去，就能得到UTF-16的编码。

以字符`𐌂`（*Old Italic Letter Ke*）为例，它的码位是U+10302，二进制表示是0000 0001 0000 0011 0000 0010，减去U+10000（二进制为0000 0001 0000 0000 0000 0000），得到0000 0000 0000 0011 0000 0010。从右到左填充进模版，得到1101 1000 0000 0000 1101 1111 0000 0010，对应的十六进制是D800 DF02。

```bash
>>> '𐌂'.encode('utf-16be')
b'\xd8\x00\xdf\x02'
```

### UTF-8
UTF-8将每个Unicode标量值映射成一到四个无符号的8比特的编码单元，这是一种变长的编码方案。

下表阐明了UTF-8的编码方式。

| Unicode标量值              | 第一个字节 | 第二个字节 | 第三个字节 | 第四个字节 |
| -------------------------- | ---------- | ---------- | ---------- | ---------- |
| 00000000 0xxxxxxx          | 0xxxxxxx   |            |            |            |
| 00000yyy yyxxxxxx          | 110yyyyy   | 10xxxxxx   |            |            |
| zzzzyyyy yyxxxxxx          | 1110zzzz   | 10yyyyyy   | 10xxxxxx   |            |
| 000uuuuu zzzzyyyy yyxxxxxx | 11110uuu   | 10uuzzzz   | 10yyyyyy   | 10xxxxxx   |

仍然以字符`𐌂`（*Old Italic Letter Ke*）为例，它的码位是U+10302，二进制表示是00000001 00000011 00000010，套用表中的模式，得到 11110000 10010000 10001100 10000010，对应的十六进制是F090 8C82。

```bash
>>> '𐌂'.encode('utf-8')
b'\xf0\x90\x8c\x82'
```

下表阐明了UTF-8的所有有效编码范围。

| Unicode标量值范围  | 第一个字节 | 第二个字节 | 第三个字节 | 第四个字节 |
| ------------------ | ---------- | ---------- | ---------- | ---------- |
| U+0000～U+007F     | 00～7F     |            |            |            |
| U+0080～U+07FF     | C2～DF     | 80～BF     |            |            |
| U+0800～U+0FFF     | E0         | A0～BF     | 80～BF     |            |
| U+1000～U+CFFF     | E1～EC     | 80～BF     | 80～BF     |            |
| U+D000～U+D7FF     | ED         | 80～9F     | 80～BF     |            |
| U+E000～U+FFFF     | EE～EF     | 80～BF     | 80～BF     |            |
| U+10000～U+3FFFF   | F0         | 90～BF     | 80～BF     | 80～BF     |
| U+40000～U+FFFFF   | F1～F3     | 80～BF     | 80～BF     | 80～BF     |
| U+100000～U+10FFFF | F4         | 80～8F     | 80～BF     | 80～BF     |



## Unicode带来的问题


### 比较
Unicode标准可能会导致两个或者多个字符存在等价的现象。

```bash
>>> c1 = 'e\u0301'
>>> c2 = '\u00e9'
>>> c1
'é'
>>> c2
'é'
>>> len(c1)
2
>>> len(c2)
1
>>> c1 == c2
False
```

上例中，c1和c2是两个不同的Unicode字符序列，但是它们显示的都是`é`，实际应用中也应该将它们视为相同的字符，称之为**标准等价物**（*canonical equivalent*），但是在Python中它们并不相等。

解决这个问题的做法，通常是通过规范化Unicode字符串。在Python中，我们使用`unicodedata.normalize`函数：

```bash
>>> from unicodedata import normalize
>>> c1 = 'e\u0301'
>>> c1
'é'
>>> c2 = '\u00e9'
>>> c2
'é'
>>> len(c1), len(c2)
(2, 1)
>>> len(normalize('NFC', c1)), len(normalize('NFC', c2))
(1, 1)
>>> len(normalize('NFD', c1)), len(normalize('NFD', c2))
(2, 2)
>>> normalize('NFC', c1) == c2
True
>>> c1 == normalize('NFD', c2)
True
```

normalize函数的第一个参数可以是NFC、NFD、NFKC和NFKD，我们重点看一下NFC和NFD。

**NFD（Normalization Form D）**：把每个字符转换成其分解的形式，也称之为规范分解。

**NFC（Normalization Form C）**：先应用NFD，再合成预先组合的形式。


## 参考资料

1. UnicodeStandard-12.0：<https://www.unicode.org/versions/Unicode12.1.0/>
2. unicodedata.normalize：<https://docs.python.org/3/library/unicodedata.html?highlight=unicodedata#unicodedata.normalize>
