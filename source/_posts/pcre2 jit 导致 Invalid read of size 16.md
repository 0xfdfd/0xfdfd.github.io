---
title: pcre2 jit 导致 Invalid read of size 16
date: 2022-11-19T14:02:36+08:00
---

在使用 [pcre2jit](https://www.pcre.org/current/doc/html/pcre2jit.html) 后，使用 valgrind 运行程序可能出现如下错误：

```
==331814== Invalid read of size 16
==331814==    at 0x4863C84: ???
==331814==    by 0x4B54DFB: ???
```

最开始以为是程序代码问题，经过搜索之后发现这是特意为之，不会造成任何问题（除了 valgrind 告警），细节见如下 Bug: [[pcre-dev] [Bug 2540] New: Valgrind errors in PCRE2 JIT code](https://www.mail-archive.com/pcre-dev@exim.org/msg06407.html)。

<!-- more -->

[关键信息](https://www.mail-archive.com/pcre-dev@exim.org/msg06413.html)摘录如下：

```
--- Comment #5 from Zoltan Herczeg <hzmes...@freemail.hu> ---
> Do you mean that this kind of reads past the end of the buffer is expected
> from PCRE2+SIMD JIT ?

Exactly. To understand it, you need to know about how virtual memory mapping is
working. You can read about it here:

https://en.wikipedia.org/wiki/Virtual_memory

The CPU maps virtual addresses to physical addresses by replacing the upper
bits of the address, but always keeps the lower n bits. Usually n is at least
12 (that means 4K pages). As far as I remember some architectures support 1K
pages (n = 10), but I am not 100% sure.

The point is: if you have a p pointer, which points to a valid memory byte
(available to the current process), reading 16 byte from (p & ~(16 - 1)) is
always valid. The (p & ~(16 - 1)) is called aligned memory address which means
the lower 4 bits of p is zeroed. Therefore we can safely read data before the
start and after the end of any buffer as long as the pointer is aligned, and
the covered memory area contains at least 1 byte of the buffer. This is not
limited to 16 bytes: any n where n is power of 2, and lower or equal than 1024
should work.

SIMD works best with large amount of data, so JIT may read data before and
after the subject buffer. However this should never cause any problem (except
for valgrind).

Let me know if you need more detailed explanation.
```

简单翻译一下：

> 绝大部分架构使用 4k 内存页（少部分使用 1k），而内存页的地址是对齐的 （比如 4k 内存页的首地址和尾地址均能够被 4096 整除），这使得对于任意有效地址 `p`，访问地址 `(p & ~(16 - 1))` 也总是有效的。
>
> 之所以 PCRE2JIT 要这么做的原因，在于 SIMD 擅长于处理大量数据，所以 PCRE2JIT 也许会将所需匹配的地址前后数据都读入，以便 SIMD 发挥最大能力。
