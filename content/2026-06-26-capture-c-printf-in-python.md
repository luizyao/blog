+++
title = "在 Python 中 C/C++ 的 printf 输出"
date = "2026-06-26"
[taxonomies]
categories = ["C/C++"]
tags = ["C/C++", "Python"]
+++

一个简单的 C 程序：

```c
#include <stdio.h>

void say_hello() {
    printf("Hello World!");
}
```

编译成共享库：`gcc -shared -fPIC hello.c -o libhello.so`。

如果想在 Python 里抓到这些输出：

```python
import io, sys, ctypes
sys.stdout = io.StringIO()

lib = ctypes.CDLL("./libhello.so")
lib.say_hello()

output = sys.stdout.getvalue()

print("#### output: \n", output)
```

结果 `output` 是空的。

一条都没抓到。

怎么回事？C 明明写了 stdout，Python 为什么看不见？故事就从这里开始。

---

## C 是怎么写 stdout 的

`printf("Hello World!")` 本质上等于 `fprintf(stdout, "Hello World!")`。这里的 `stdout` 是一个 `FILE*` 类型的指针，不是文件描述符 1。`FILE*` 是 C 标准库对文件描述符的一层包装，它带了一个关键的东西——**内存缓冲区**。

当你调用 `printf` 时，数据并不一定立刻写出去。它的路径是这样的：

```
printf("Hello World!") → 写入 FILE* 的 buffer → （某个时刻）→ write(1, buf, len) → 操作系统 → 终端/文件
```

这个"某个时刻"取决于缓冲策略：

- **行缓冲**（终端模式）：遇到 `\n` 就 `write`
- **全缓冲**（重定向到文件）：buffer 满了（通常 4096 字节）才 `write`
- **无缓冲**（stderr）：来一条写一条

用一个比喻可能更好理解：

C 的 stdout 就像一根中间接了个水箱的水管。你往水管里倒水（`printf`），水先积在水箱里（FILE buffer）。水箱满了、或者你倒的水里有个浮标（`\n`），水箱的阀门才打开，水才流到河里去（`write(FD 1)`）。

注意上面那个示例：`printf("Hello World!")` 没有 `\n`。如果你的程序跑在一个管道或文件重定向的环境里（全缓冲），这行字符串会一直躺在 buffer 里，直到程序退出时才被 `write` 出去。

---

## Python 的 `sys.stdout` 替换抓不到 C

先解释一下为什么前面用了 `sys.stdout = io.StringIO()`——这在 Python 世界里是捕获输出的"标准操作"。

Python 的 `print()` 函数底层调用的是 `sys.stdout.write()`。所以如果你把 `sys.stdout` 换成一个 `io.StringIO` 对象，所有 Python 代码的输出都会被截住。这个模式遍布在测试代码、教学示例、甚至标准库的 `contextlib.redirect_stdout` 里：

```python
# Python 文档上的标准做法
old_stdout = sys.stdout
sys.stdout = io.StringIO()
print("hello")                    # 走 sys.stdout.write()
output = sys.stdout.getvalue()    # 从 StringIO 拿回来
sys.stdout = old_stdout           # 恢复
```

简单、无副作用、纯 Python 层操作，是一个 Python 开发者的肌肉记忆。

**但问题在于**：这个操作隐含了一个假设——所有输出都走 `sys.stdout.write()`。这个假设在纯 Python 世界里成立。C 根本不走这一层：

```
Python: print("hello") → sys.stdout.write() → [io.StringIO 捕获成功]
C:      printf("hello") → FILE* buffer → write(1, buf) → [走 FD，StringIO 抓不到]
```

Python 和 C 的 stdout 是**两张皮**，在 Python 层替换没用。

---

### `os.dup2()` — 在文件描述符层面动手

不管是 C 还是 Python，最终都会走到 `write(1, buf)`——文件描述符 1 是所有人最后的共同出口。所以唯一一个能通杀 C 和 Python 的截获点，就是 **FD 1 本身**。

`os.dup2(new_fd, 1)` 可以把 FD 1 重定向到任意目标：一个临时文件、一个管道、甚至 `/dev/null`。替换之后，不管是谁写 FD 1，数据都流到新目标上。

看一段纯 Python 的演示（直接用 `libhello.so`）：

```python
import os, tempfile, ctypes

libhello = ctypes.CDLL("./libhello.so")

# ---------- 开始捕获 ----------
ctypes.CDLL(None).fflush(None)       # flush 替换前的老 buffer
tmp = tempfile.NamedTemporaryFile(delete=False)
tmp.close()

orig_fd = os.dup(1)                  # 保存原始的 FD 1
tmp_fd = os.open(tmp.name, os.O_WRONLY)
os.dup2(tmp_fd, 1)                   # FD 1 → 指向临时文件
os.close(tmp_fd)

# ---------- 被捕获的代码 ----------
libhello.say_hello()

ctypes.CDLL(None).fflush(None)        # flush 替换后的新 buffer -> 进入临时文件

# ---------- 恢复 FD 1 ----------
os.dup2(orig_fd, 1)                  # 恢复原始 FD 1
os.close(orig_fd)

# ---------- 读取捕获 ----------
with open(tmp.name) as f:
    content = f.read()
os.unlink(tmp.name)

print(repr(content))
```

输出：

```
'Hello World!'
```

不管 C 还是 Python，都被截到了同一个文件里。

流程图：

```
printf("Hello World!") → FILE* buffer → write(1, buf) ──→ [FD 1]
                                                             │
                                                  os.dup2 →  ▼
                                                          临时文件
```

---

## C 标准库的缓冲区

上面的演示能抓到数据，是因为 `say_hello` 返回后，共享库代码里没有额外 buffer 滞留问题（`printf` 写完马上返回）。但如果你在一个**正在运行的 Python 进程里调 C 扩展库**，情况就不同了：

- C 扩展库调了 `printf("Hello World!")`，没有 `\n`
- 如果你的进程不是连接到终端（比如被重定向了），C 的缓冲模式变成全缓冲
- "Hello World!" 躺在 FILE buffer 里，还没 `write` 出去
- 你 `os.dup2` 换了 FD 1——换的是水管出水口，但水箱里的水还在水箱里

形象地说：你换掉了水管出口（FD 1 → 文件），但 C 的水箱（FILE buffer）里的水还在里面，没放出来。

解法是：**手动让 C 吐出 buffer**。

```python
import ctypes
libc = ctypes.CDLL(None)   # 获取当前进程的全局符号表
libc.fflush(None)           # 等价于 C 的 fflush(NULL)，刷新所有输出流
```

`ctypes.CDLL(None)` 返回一个代表当前进程全局符号的 CDLL 对象，可以调用任何 C 标准库函数。`fflush(NULL)` 在 C 里的语义是"把所有输出流的 buffer 都刷出去"。

把这个操作封装成两个步骤：

```python
import os, tempfile, ctypes

def start_capture():
    ctypes.CDLL(None).fflush(None)      # flush 替换前的老 buffer
    tmp = tempfile.NamedTemporaryFile(delete=False)
    tmp.close()
    saved_fd = os.dup(1)
    os.dup2(os.open(tmp.name, os.O_WRONLY), 1)
    return tmp.name, saved_fd

def stop_capture(tmp_path, saved_fd):
    ctypes.CDLL(None).fflush(None)      # flush 替换后的新 buffer
    with open(tmp_path) as f:
        content = f.read()
    os.dup2(saved_fd, 1)                # 恢复 FD 1
    os.close(saved_fd)
    os.unlink(tmp_path)
    return content
```

两个 `fflush` 各司其职：

- **替换前 flush**：确保替换前 C 可能积压在 buffer 里的老数据已经被写走，不会因为 FD 1 被换掉而丢失
- **替换后 flush**：确保被捕获的 C 代码写到 buffer 的最新数据被推到 FD 1，进入临时文件

演示：

```python
libhello = ctypes.CDLL("./libhello.so")

tmp_path, saved_fd = start_capture()

libhello.say_hello()

content = stop_capture(tmp_path, saved_fd)
print(repr(content))
```

如果没有 `start_capture` 里的 `fflush`，"Hello World!" 可能就丢了；如果没有 `stop_capture` 里的 `fflush`，"Captured!" 可能还没进文件。

---

---

## 完整流程回顾

```
// hello.c
void say_hello() { printf("Hello World!"); }
                   │
                   ▼
C printf("Hello World!") ──→ FILE* buffer ──→ fflush(NULL)
                                                     │
                                                     ▼
                                               write(1, buf)      ← 系统调用，FD 1
                                                     │
                                                os.dup2           ← Python 拦截
                                                     │
                                                     ▼
                                               临时文件/管道       ← Python 可读
                                                     │
                                                     ▼
                                       Python open + read / os.read
```

三个关键工具：

| 工具 | 作用 | 为什么需要 |
|---|---|---|
| `os.dup2(new_fd, 1)` | 替换文件描述符 1 | C/Python 最终都走 FD 1，这是唯一通杀的截获点 |
| `ctypes.CDLL(None).fflush(None)` | 强制 C 刷新 FILE buffer | C 的数据可能还积在 buffer 里，没到 FD 1 |

---

## 总结：一张脑图

```
                        ┌─────────────────────┐
                        │   C printf()        │
                        │   Python print()    │
                        └─────────┬───────────┘
                                  │
                        ┌─────────▼───────────┐
                        │    FILE* buffer     │  ← C 标准库层
                        │   (行缓冲 / 全缓冲)  │     Python 看不见
                        └─────────┬───────────┘
                                  │ fflush(NULL)
                        ┌─────────▼───────────┐
                        │   write(FD 1)       │  ← 操作系统层
                        └─────────┬───────────┘
                                  │ os.dup2()
                        ┌─────────▼───────────┐
                        │   临时文件 / 管道    │  ← Python 可以读
                        └─────────────────────┘
```

- 不要想着在 Python 层（`sys.stdout` 替换）去抓 C 的输出——那是两张皮
- 文件描述符 1 是所有语言最后共同经过的唯一路口，用 `os.dup2` 在那里动手
- 不要忘了 C 标准库的缓冲区，`fflush` 是你的朋友
