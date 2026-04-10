下面是在**不删除任何内容**前提下，对你提供内容进行**结构化整理 + 去冗余排版 + 逻辑分层**后的文档（适合直接放入 Typora）。

------

# CPU 的缓存一致性

------

# 一、提升代码运行速度的几个关键点

## 1. 尽可能使用连续内存

- 尽量避免跳跃访问内存
- CPU 以 cache line（通常 64B）为单位加载数据
- 以 64 位 CPU 为例，一次可处理 8 字节

结论：

> 数组应尽量连续访问，提高 cache 命中率

------

## 2. 利用缓存预取（Cache Prefetch）

让 CPU 提前加载数据：

- CPU 会预测访问模式
- 提前把数据加载到 cache
- 分支预测 + cache 命中 → 提高性能

------

# 二、CPU Cache 写策略（Write Policy）

## 本质定义

> CPU 修改数据时，cache 和内存之间如何同步

属于：

```
CPU cache controller 硬件逻辑
```

------

## 1. 形象模型（办公室）

```
CPU = 员工
Cache = 办公桌
Memory = 档案室
```

------

## 2. 写直达（Write-through）

### 规则

```
写 cache 同时写 memory
```

### 流程

```
CPU → cache → memory（立即）
```

### 特点

优点：

- 数据一致性强

缺点：

- 写慢

------

## 3. 写回（Write-back）

### 规则

```
只写 cache，延迟写 memory
```

### 流程

```
CPU → cache（dirty）→ 延迟 → memory
```

### Dirty bit

```
0 = 干净
1 = cache 比 memory 新
```

------

## 4. 现代 CPU 选择

```
几乎全部采用 write-back
```

原因：

```
性能更高
```

------

# 三、Cache Coherence（缓存一致性）

## 为什么需要

多核情况下：

```
多个 CPU cache 同一数据
```

例如：

```
CPU1: x=20
CPU2: x=10
memory: x=10
```

------

## MESI 协议

状态：

```
M = Modified
E = Exclusive
S = Shared
I = Invalid
```

------

## 核心保证

> 同一时刻只有一个 CPU 可以写 cache line

------

# 四、volatile 的本质

## 核心定义

> volatile 只约束编译器，不约束 CPU

------

## 作用

```
禁止编译器优化
每次生成 load/store 指令
```

------

## 不保证

- ❌ 不保证访问 RAM
- ❌ 不保证线程安全
- ❌ 不保证原子性
- ❌ 不影响 cache 策略

------

## 本质总结

```
volatile = 禁止编译器缓存变量
```

不是：

```
禁止 CPU cache
```

------

# 五、编译器优化（寄存器 vs 内存）

## 示例

```cpp
int a;
return a + a;
```

优化为：

```asm
mov eax,[a]
add eax,eax
```

------

## 关键点

- 只读取一次内存
- 后续使用寄存器

------

## 寄存器 vs Cache

层级：

```
寄存器 > L1 > L2 > L3 > RAM
```

------

## 结论

> 后续计算来自寄存器，而不是 cache

------

# 六、Load / Store 行为

## 1. 读（load）

```
RAM → cache → register
```

不会触发 write-back

------

## 2. 写（store）

write-back：

```
register → cache（dirty）→ 延迟 → RAM
```

------

## 3. cache fill

```
读数据时顺便放入 cache
```

------

# 七、MMIO（内存映射IO）

特点：

```
绕过 cache
直接访问设备
```

路径：

```
CPU → 设备寄存器
```

------

# 八、32位 CPU 与高位地址问题

## 核心矛盾

```
CPU寄存器宽度 < 地址宽度
```

------

## 问题

1. 寄存器装不下地址
2. 指令无法编码
3. AGU 复杂
4. 页表复杂
5. cache tag 增大

------

## 典型方案

```
PAE（物理地址扩展）
```

------

## 结论

> 32 位 CPU 无法原生支持 64 位地址空间

------

# 九、数据竞争 vs 伪共享

------

## 1. 数据竞争（Data Race）

定义：

- 同一变量
- 有写
- 无同步

结果：

```
未定义行为（UB）
```

------

## 2. 伪共享（False Sharing）

定义：

- 不同变量
- 同一 cache line

结果：

```
性能下降
```

------

## 核心区别

| 类型     | 是否错误  | 是否慢 |
| -------- | --------- | ------ |
| 数据竞争 | ✔会错误   | 不一定 |
| 伪共享   | ✖不会犯错 | ✔      |

------

# 十、伪共享（重点）

## 定义

> 不同线程操作不同变量，但落在同一个 cache line

------

## 类比

```
一张纸（cache line）
两个人写不同区域
但必须抢整张纸
```

------

## 现象

```
cache line bouncing
```

------

## 示例

```cpp
struct Data
{
    int a;
    int b;
};
```

------

## 解决方案

小林coding 图解系统 1.5

### 1. padding+对齐

![image-20260410125838893](/Users/ty/github/Books/小林coding/图解操作系统linux+windows/笔记/图解系统笔记-第一章 硬件系统/assets/image-20260410125838893.png)

```cpp
char pad[60];
```

```cpp
alignas(64)
```

------

![image-20260410125515439](/Users/ty/github/Books/小林coding/图解操作系统linux+windows/笔记/图解系统笔记-第一章 硬件系统/assets/image-20260410125515439.png)`__cacheline_aligned_in_smp` 是 **Linux 内核里的一个宏**，作用很明确：

> **在 SMP（多核）环境下，把变量按 cache line 对齐，用来避免伪共享；在单核环境下则什么都不做。**

### 为什么要这么设计

核心原因：

```
伪共享只在多核 CPU 上才会发生
```

# 十一、为什么不会读到错误数据

## 原因

Cache Coherence 保证：

```
要么旧值，要么新值
不会是垃圾值
```

------

## 但仍然会错

原因：

### 1. 丢失更新

```
a++ 不是原子
```

### 2. 指令重排

### 3. 可见性延迟

------

# 十二、并发正确性保证

## 方法

### 1. 原子变量

```cpp
std::atomic<int>
```

------

### 2. 锁

```cpp
std::mutex
```

------

### 3. 内存序

------

# 总结

## 一句话总结（最终版）

```
CPU保证数据不会坏（cache coherence）
但不保证你的并发逻辑是对的（需要同步机制）
```

------

如果你下一步想继续深入，可以往这几个方向走（非常关键）：

- store buffer（为什么可见性延迟）
- memory barrier（如何控制乱序）
- atomic 的底层实现（lock / cache line 锁）
- NUMA 对伪共享的放大效应

这些一旦打通，你对多线程 + CPU 架构会进入“工程级理解”。