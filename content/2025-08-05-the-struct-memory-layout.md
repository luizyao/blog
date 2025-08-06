+++
title = "C 语言中结构体的内存布局"
date = "2025-08-05"
[taxonomies]
categories = ["C/C++"]
tags = ["C 语言", "struct 内存布局"]
+++

C 语言中结构体（struct）的内存布局和编译器具体的实现有关，但遵循一般规则：

- 结构体中各成员在内存中的存储顺序和它们的声明顺序一致。
- 为了提高内存的访问速度，编译器可能会在各成员间添加填充字节（也可能在结构体之后，但不会出现在结构体开始的位置）。
- 结构体按照成员的最大内存占用字节数做对齐。

<!-- more -->

## 结构体的一般布局

C18 标准 [$6.7.2.1](http://www2.open-std.org/JTC1/SC22/WG14/www/docs/n2310.pdf) 中一些相关描述：

> 14 Each non-bit-field member of a structure or union object is aligned in an implementation-defined manner appropriate to its type.
>
> 15 Within a structure object, the non-bit-field members and the units in which bit-fields reside have addresses that increase in the order in which they are declared. A pointer to a structure object, suitably converted, points to its initial member (or if that member is a bit-field, then to the unit in which it resides), and vice versa. There may be unnamed padding within a structure object, but not at its beginning.

以下面的结构体为例：

```c
struct foo {
  int a;
  char *b;
  char c;
};
```

`struct foo` 包含三个属性，在大多数 64 位的机器上，`int` 类型占用 4 个字节，`char *` 类型占用 8 个字节，`char` 类型占用 1 个字节，共计 13 各字节。在内存中，按照 8 个字节做对齐：

- `int a` 的起始字节位是 `0`，占用 4 个字节。
- 编译器填充 4 个字节。
- `char *b` 的起始字节位是 `8`，占用 8 个字节。
- `char c` 的起始字节位是 `16`，占用 1 个字节。
- 编译器在最后填充 7 个字节，做对齐。

实际的内存布局中，共占用 24 个字节。

可以使用 [offsetof](https://man7.org/linux/man-pages/man3/offsetof.3.html) 和 `sizeof` 验证上述猜想：

```c
printf("offsets: a=%zu, b=%zu, b=%zu\n", offsetof(struct foo, a),
        offsetof(struct foo, b), offsetof(struct foo, c));
printf("sizeof(struct foo)=%zu\n", sizeof(struct foo));
```

验证结果：

```txt
offsets: a=0, b=8, b=16
sizeof(struct foo)=24
```

## 优化内存布局

结构体在内存中的储存顺序和成员的声明顺序一致，可以通过修改声明顺序，减少内存占用。

按照成员类型大小升序（降序）排列：

```c
struct foo {
  char a;  // 1 个字节
  int b;   // 4 个字节
  char *c; // 8 个字节
};
```

这时的内存布局，共占用 16 个字节，比第一种少了 8 个字节：

```txt
offsets: a=0, b=4, b=8
sizeof(struct foo)=16
```

## `#pragma pack` 指令

> `#pragma pack` 指令的实现依赖具体的编译器，不同的编译器（GNU、CLANG 等）可能会有稍微的不同。
>
> 这里以 [GNU 的实现](https://gcc.gnu.org/onlinedocs/gcc-15.1.0/gcc/Structure-Layout-Pragmas.html)为例，本文使用的 gcc 版本是 `gcc version 15.1.1 20250521 (Red Hat 15.1.1-2)`。

- `#pragma pack(n)`：Sets the new alignment according to n.

  `n` 的单位是字节。值必须是 2 的幂，例如：1、2、4、8、16 等。如果为 0，等同于没有使用 `#pragma pack` 指令。

- `#pragma pack(push[,n])`：Pushes the current alignment setting on an internal stack and then optionally sets the new alignment.
- `#pragma pack(pop)`：Restores the alignment setting to the one saved at the top of the internal stack (and removes that stack entry).

仍以上述结构体为例，取消内存对齐：

```c
#pragma pack(push, 1)
struct foo {
  int a;   // 4 个字节
  char *b; // 8 个字节
  char c;  // 1 个字节
};
#pragma pack(pop)
```

此时的内存布局：

```txt
offsets: a=0, b=4, b=12
sizeof(struct foo)=13
```

### `__attribute__` 机制

> `__attribute__` 是 GCC 特有的机制，用于在声明函数、变量或类型时指定属性，以影响编译器的行为。

可以通过其中的 [aligned 和 packed](https://gcc.gnu.org/onlinedocs/gcc/Common-Type-Attributes.html) 属性控制对齐方式。

```c
struct foo {
  int a;   // 4 个字节
  char *b; // 8 个字节
  char c;  // 1 个字节
} __attribute__((packed));
```

实现同样的效果：

```txt
offsets: a=0, b=4, b=12
sizeof(struct foo)=13
```

在实际的项目中，[libtins](https://github.com/mfontanini/libtins/blob/master/include/tins/macros.h#L52) 库就是使用了 **attribute** 机制。

```c++
// Check if this is Visual Studio
#ifdef _MSC_VER
    // This is Visual Studio
    #define TINS_BEGIN_PACK __pragma( pack(push, 1) )
    #define TINS_END_PACK __pragma( pack(pop) )
    #define TINS_PACKED(DECLARATION) __pragma( pack(push, 1) ) DECLARATION __pragma( pack(pop) )
    #define TINS_DEPRECATED(func) __declspec(deprecated) func
    #define TINS_NOEXCEPT
    #define TINS_LIKELY(x) (x)
    #define TINS_UNLIKELY(x) (x)
#else
    // Not Visual Studio. Assume this is gcc compatible
    #define TINS_BEGIN_PACK
    #define TINS_END_PACK __attribute__((packed))
    #define TINS_PACKED(DECLARATION) DECLARATION __attribute__((packed))
    #define TINS_DEPRECATED(func) func __attribute__ ((deprecated))
    #define TINS_NOEXCEPT noexcept
    #define TINS_LIKELY(x) __builtin_expect((x),1)
    #define TINS_UNLIKELY(x) __builtin_expect((x),0)
#endif // _MSC_VER
```
