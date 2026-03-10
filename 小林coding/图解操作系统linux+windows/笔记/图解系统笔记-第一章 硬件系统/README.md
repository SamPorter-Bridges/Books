# 提升代码运行速度的几个关键点

1.尽可能使用连续的内存空间，不要跳跃使用空间

因为内存是连续加载的，以64位cpu为例，一次能够加载8个字节的空间，所以数组要尽可能连续用

2.让cpu的缓存提前知道使用的空间内都有什么，这相当于预加载一次数据，这样再遇到if条件分支的时候，cpu的分支预测器会直接从cache里拉数据

# 写策略writeThrogh和write-back

可以把 **CPU cache 写策略（write policy）** 理解成：

> **当 CPU 修改一个变量时，数据如何在 cache 和内存之间传播的规则。**

它确实属于 **CPU 硬件实现的一部分（cache controller 的底层逻辑）**。
程序员通常无法直接控制，只是 CPU 设计的一部分。

下面用一个更直观的例子说明。

------

## 1 一个形象模型：办公室文件柜

假设：

- **CPU** = 办公室员工
- **L1/L2 cache** = 办公桌抽屉
- **内存（RAM）** = 档案室文件柜

员工为了工作效率，会把文件先放在桌子抽屉里，而不是每次都跑去档案室。

```
CPU(员工)
   ↓
L1 Cache（桌子抽屉）
   ↓
L2/L3 Cache（办公室柜子）
   ↓
Memory（档案室）
```

------

## 2 读数据（为什么有 cache）

如果员工每次查资料都跑档案室：

```
员工 → 档案室
```

会很慢。

所以会先把文件复制一份到桌子：

```
员工 → 桌子抽屉
```

下次直接拿。

------

## 3 写策略（重点）

当员工 **修改文件** 时，有两种管理方式。

------

## 4 写直达（Write-through）

规则：

> **每次修改文件，同时更新档案室。**

流程：

```
CPU 修改数据
      ↓
写入 cache
      ↓
同时写入 memory
```

用例子表示：

员工改了桌上的文件：

```
桌子抽屉：工资=10000
```

必须 **立刻跑档案室更新**：

```
档案室：工资=10000
```

所以始终：

```
cache == memory
```

优点：

- 数据始终一致
- 简单可靠

缺点：

- 写操作很慢
- 每次都要访问内存

------

## 5 写回（Write-back）

规则：

> **只修改 cache，等合适时机再写回内存。**

流程：

```
CPU 修改数据
      ↓
只写入 cache
      ↓
标记 dirty
      ↓
以后再写回 memory
```

例子：

员工改了桌上的文件：

```
桌子抽屉：工资=10000
档案室：工资=8000
```

这时候 **档案室还是旧数据**。

但桌子抽屉会贴一个标签：

```
DIRTY
```

表示：

```
这个数据被修改过
```

什么时候写回？

通常：

1. cache 被替换
2. cache line eviction
3. 系统同步

此时：

```
cache → memory
```

------

## 6 真实 CPU 几乎全部使用 Write-back

现代 CPU（x86 / ARM）：

L1 / L2 / L3 cache

基本都是：

```
write-back
```

因为性能更高。

否则写操作太慢。

------

## 7 Dirty bit（关键机制）

每个 cache line 有一个标记：

```
dirty bit
```

含义：

```
0 = cache 和 memory 一致
1 = cache 比 memory 新
```

写回策略流程：

```
CPU写数据
     ↓
cache更新
     ↓
dirty=1
     ↓
替换cache line
     ↓
写回memory
```

------

## 8 写策略属于 CPU 底层逻辑吗

是的。

更准确说属于：

```
CPU cache controller
```

结构：

```
CPU Core
   │
   ├── L1 Cache
   │       │
   │   Cache Controller
   │       │
   │   Write Policy
   │
   ├── L2 Cache
   │
   └── Memory Controller
```

写策略是 **cache controller 的实现逻辑**。

程序基本无法改变。

------

## 9 和 volatile 没关系

即使变量是：

```
volatile int x;
```

CPU 仍然可能：

```
CPU
 ↓
L1 cache
 ↓
稍后
memory
```

因为写策略仍然是：

```
write-back
```

------

## 10 一个真实系统流程

假设代码：

```cpp
x = 10;
```

CPU执行：

```
store x
```

可能发生：

```
CPU register
     ↓
L1 cache (dirty)
     ↓
L2 cache
     ↓
L3 cache
     ↓
memory (later)
```

------

## 11 再给你一个非常关键的现实问题

为什么多核 CPU 会有 **cache coherence protocol**（MESI）？

原因正是：

```
write-back
```

因为多个 CPU 可能都有同一数据的 cache。

例如：

```
CPU1 cache: x=10
CPU2 cache: x=10
memory: x=10
```

CPU1 改：

```
CPU1 cache: x=20
CPU2 cache: x=10
memory: x=10
```

这就不一致了。

所以 CPU 引入：

```
MESI protocol
```

保证：

```
cache coherence
```

------

如果继续深入，其实整个体系是这样一套完整结构：

```
CPU
 ├─ store buffer
 ├─ L1 cache
 ├─ L2 cache
 ├─ L3 cache
 ├─ cache coherence (MESI)
 └─ memory
```

理解 **store buffer + write-back + MESI** 之后，你就会彻底理解：

为什么会出现

```
内存重排序
memory barrier
atomic
```

这些并发问题。

# 不要把**volatile** 和 **CPU cache 写策略（write-through / write-back）** 混在一起了

你这里其实把 **`volatile`（不是 volidate）** 和 **CPU cache 写策略（write-through / write-back）** 混在一起了。这两者属于完全不同层面的机制：

- `volatile`：**编译器层面的约束（compiler semantics）**
- cache 写策略：**CPU硬件层面的缓存一致性策略（hardware cache policy）**

`volatile` **不会改变 CPU cache 的写策略，也不会强制 CPU 直接访问内存。**

下面分层解释。

------

## 1 `volatile` 的真实语义（C/C++）

`volatile` 的作用只有一条：

> **禁止编译器对该变量的访问进行优化，每一次读写都必须生成真实的内存访问指令。**

具体表现为：

### 读取

编译器不能把变量缓存到寄存器里。

例如：

```cpp
int flag = 0;

while(flag == 0)
{
}
```

编译器可能优化成：

```asm
mov eax, flag
loop:
cmp eax,0
je loop
```

因为它认为 `flag` 不会被别人改。

如果是：

```cpp
volatile int flag = 0;
```

则必须每次重新读取：

```asm
loop:
mov eax, flag
cmp eax,0
je loop
```

------

### 写入

写入必须真实生成 store 指令：

```cpp
volatile int x;
x = 10;
```

编译器不能消除这次写操作。

------

## 2 `volatile` 不保证的事情

很多人误解 `volatile` 可以：

- ❌ 强制从 **内存读取**
- ❌ 强制写入 **内存**
- ❌ 禁止 CPU cache
- ❌ 保证线程同步
- ❌ 保证原子性

这些 **全部不是 volatile 的职责**。

CPU 仍然可能：

```
寄存器
   ↓
L1 cache
   ↓
L2 cache
   ↓
L3 cache
   ↓
Memory
```

`volatile` 不会改变这个路径。

------

## 3 `volatile` 与 CPU cache 的真实关系

当执行：

```cpp
volatile int x;
x = 5;
```

生成的机器指令大概是：

```
mov [x], 5
```

之后发生什么：

取决于 **CPU cache policy**

例如：

### write-back（主流 CPU）

```
CPU store
   ↓
L1 cache
   ↓ (稍后)
Memory
```

不会立即写入内存。

------

### write-through

```
CPU store
   ↓
L1 cache
   ↓
Memory (立即)
```

但这是 **cache controller 的策略**，和 `volatile` 无关。

------

## 4 那 `volatile` 到底解决什么问题？

它主要用于 **三种情况**：

------

### 1 硬件寄存器（最典型）

嵌入式：

```cpp
#define UART_STATUS (*(volatile unsigned int*)0x40001000)

while((UART_STATUS & 0x1) == 0)
{
}
```

原因：

**硬件寄存器可能被 外设修改。**

必须每次重新读取。



------

### 2 中断修改变量

```cpp
volatile int flag;

void ISR()
{
    flag = 1;
}

int main()
{
    while(flag == 0);
}
```

中断可能改变 `flag`。

------

### 3 多线程（历史用途）

早期代码：

```cpp
volatile bool stop;
```

但在现代 C++ 中：

正确方法是

```
std::atomic
```

因为 `volatile` **不能保证内存序**。

------

## 5 `volatile` 不等于 memory barrier

如果你想要 **CPU 级别的内存同步**，需要：

```
memory barrier
```

例如：

x86

```
mfence
lfence
sfence
```

C++ 对应：

```
std::atomic
std::atomic_thread_fence
```

------

## 6 `volatile` 与 write-back / write-through 没关系

你问的这个：

> 这是不是从写直达切换为写回？

答案是：

**完全没有关系。**

| 机制           | 层级      |
| -------------- | --------- |
| volatile       | 编译器    |
| write-back     | CPU cache |
| write-through  | CPU cache |
| memory barrier | CPU       |

------

## 7 一个更准确的理解方式

可以这样理解：

```
volatile = 禁止编译器缓存变量
```

而不是：

```
volatile = 禁止CPU缓存变量
```

------

## 8 一张完整流程图

正常变量：

```
source code
    ↓
compiler优化
    ↓
寄存器缓存
    ↓
CPU cache
    ↓
memory
```

volatile变量：

```
source code
    ↓
禁止编译器优化
    ↓
每次生成load/store
    ↓
CPU cache
    ↓
memory
```

区别只在 **编译器阶段**。

------

## 9 一个非常经典的总结

一句话：

> **volatile 是给编译器看的，不是给 CPU cache 看的。**

------

如果你愿意，我可以再给你讲一个 **很多资深 C++ 程序员都搞错的问题**：

**为什么 Linux 内核大量使用 `volatile`，但 C++ 多线程却不推荐使用 `volatile`。**

这里涉及 **memory model / cache coherency / CPU reorder**，理解后你对 **CPU cache + 编译器优化 + 并发模型**会一下子通透。

# “Volatile禁止编译器对该变量的访问进行优化，每一次读写都必须生成真实的内存访问指令”是什么意思

这句话如果严格解释，需要区分 **编译器行为** 和 **CPU行为**。这里说的“真实的内存访问指令”，指的是 **编译器必须生成 `load/store` 指令访问内存地址，而不能用寄存器或优化掉访问**。它并不是说一定访问 **RAM**，而是访问 **该变量的内存地址**。

换一个更具体的方式说明。

------

## 1 普通变量时编译器会做什么优化

假设代码：

```cpp
int a = 10;

int func()
{
    int b = a;
    int c = a;
    return b + c;
}
```

编译器会分析：

- `a` 在这个函数里 **没有被修改**
- 两次读取 `a` **结果一样**

所以编译器可能生成类似汇编：

```asm
mov eax, [a]   ; 读取一次
add eax, eax   ; eax = eax + eax
```

原本代码里 **读了两次 a**，但编译器只读了一次。

甚至在某些情况下，编译器可能把变量直接放到寄存器：

```asm
mov eax, 10
add eax, eax
```

完全不再访问内存。

------

## 2 如果变量是 volatile

代码：

```cpp
volatile int a = 10;

int func()
{
    int b = a;
    int c = a;
    return b + c;
}
```

编译器必须认为：

> **每一次读取都可能得到不同结果**

因此必须生成：

```asm
mov eax, [a]   ; 第一次读取
mov edx, [a]   ; 第二次读取
add eax, edx
```

注意这里：

```
[a]
```

表示：

```
访问变量所在的内存地址
```

这就是所谓：

```
真实的内存访问指令
```

------

## 3 再看写操作

普通变量：

```cpp
int x;
x = 1;
x = 2;
```

编译器会优化：

```
x = 2
```

因为第一次写没有意义。

但如果是：

```cpp
volatile int x;
x = 1;
x = 2;
```

编译器必须生成两次写：

```asm
mov [x], 1
mov [x], 2
```

不能优化掉第一次。

------

## 4 为什么需要这样

因为有些变量 **可能被程序外部修改**。

例如：

### 1 硬件寄存器

```cpp
#define UART_STATUS (*(volatile int*)0x40001000)

while((UART_STATUS & 1) == 0)
{
}
```

如果没有 volatile：

编译器可能优化成：

```asm
mov eax, [0x40001000]
test eax,1
je loop
```

只读一次。

但真实情况是：

```
硬件设备会修改这个寄存器
```

所以必须每次读取。

------

### 2 中断修改

```cpp
volatile int flag = 0;

while(flag == 0)
{
}
```

中断可能改变 `flag`。

如果不使用 volatile：

编译器可能把 `flag` 放寄存器：

```
while(1)
```

程序就死循环了。

------

## 5 “真实内存访问”不要误解

很多人误解为：

```
volatile → 强制访问RAM
```

这是错误的。

真实情况是：

```
CPU
 ↓
寄存器
 ↓
L1 cache
 ↓
L2 cache
 ↓
L3 cache
 ↓
RAM
```

访问 `[a]` 时：

CPU **通常访问的是 cache**，不是直接访问 RAM。

所以：

```
volatile 只影响编译器
不会影响 CPU cache
```

------

## 6 一句话总结

这句话真正的含义是：

```
volatile 变量
每一次读写都必须生成
load/store 指令访问该变量的内存地址
不能被编译器优化掉或缓存到寄存器
```

但：

```
是否访问RAM
是否写回RAM
由CPU cache机制决定
```

------

如果你继续深入，其实还有一个 **很多工程师第一次学 volatile 都会误解的问题**：

> 为什么 `volatile` 在 **多线程同步** 中通常是错误的做法？

这个问题涉及 **CPU乱序执行 + store buffer + cache一致性协议**，理解之后你会彻底明白 `atomic` 为什么存在。

# 海目星现场volatile 外设io修改变量值的真正原因

算，但需要区分 **两种不同情况**。很多现场工程师其实把这两种情况混在一起了。

------

## 1 情况一：变量映射到硬件寄存器（典型 `volatile` 用途）

例如 MCU / FPGA / IO 控制器寄存器。

代码类似：

```cpp
#define GPIO_STATUS (*(volatile unsigned int*)0x40001000)
```

或者

```cpp
volatile unsigned int* gpio = (unsigned int*)0x40001000;
```

假设这个地址对应 **IO输入寄存器**。

硬件流程是：

```
外设IO电平变化
      ↓
IO控制器更新寄存器
      ↓
CPU读取该寄存器地址
```

此时寄存器里的值 **可能随时变化**：

```
0x40001000

时间t1 = 0
时间t2 = 1
时间t3 = 0
```

如果不用 `volatile`：

编译器可能只读取一次：

```asm
mov eax, [0x40001000]
```

然后循环使用这个值。

这样程序就 **永远看不到IO变化**。

所以必须：

```
volatile
```

保证：

```
每次循环都重新读寄存器
```

这种情况 **完全符合你说的现场问题**。

------

## 2 情况二：中断 / DMA 修改内存变量

另一种常见情况是：

```cpp
volatile int flag = 0;
```

中断里：

```cpp
void ISR()
{
    flag = 1;
}
```

主线程：

```cpp
while(flag == 0)
{
}
```

这里修改变量的是：

```
CPU中断
```

不是外设寄存器。

但对编译器来说是一样的问题：

```
变量可能被程序外部修改
```

所以也需要 `volatile`。

------

## 3 现场 IO 修改变量的真实结构

很多工业现场其实是这种结构：

```
传感器
   ↓
IO模块
   ↓
PLC / FPGA
   ↓
寄存器
   ↓
CPU读取
```

CPU代码：

```cpp
while(!(GPIO_STATUS & 1))
{
}
```

如果没有 `volatile`：

编译器可能生成：

```
读一次寄存器
然后死循环
```

你就会看到：

```
IO已经变化
程序却没反应
```

很多人现场调试半天。

------

## 4 Linux 驱动里为什么大量使用 volatile

例如：

```cpp
#define REG_STATUS (*(volatile uint32_t*)0xF0000000)
```

驱动代码：

```cpp
while(!(REG_STATUS & READY))
{
}
```

因为：

```
硬件设备会异步修改寄存器
```

------

## 5 但有一个更专业的做法

在 **现代系统（Linux内核 / ARM）** 中，其实更推荐：

```
memory-mapped IO
```

配合：

```
readl()
writel()
```

例如：

```c
val = readl(reg);
```

原因是：

除了防止编译器优化，还要处理：

```
CPU乱序执行
cache
memory barrier
```

------

## 6 一个现场工程常见 bug

代码：

```cpp
int flag = 0;

while(flag == 0)
{
}
```

另一个线程：

```
flag = 1
```

结果：

```
程序死循环
```

原因：

编译器优化：

```
mov eax, flag
while(eax == 0)
```

而不是不断读内存。

------

## 7 一句话判断是否需要 volatile

如果变量可能被 **当前执行流之外的东西修改**：

```
硬件
中断
DMA
另一个线程
```

就可能需要 `volatile`。

------

最后补充一个很多做 **嵌入式 / 工控 / 驱动的人第一次知道都会很震惊的事实**：

> **很多 CPU 访问硬件寄存器时，cache 是被完全绕过的。**

也就是说：

```
CPU → cache → memory
```

在 **MMIO（memory mapped IO）** 时会变成：

```
CPU → 设备寄存器
```

根本不走 cache。

这也是为什么硬件寄存器可以实时变化。

如果你愿意，我可以给你画一个 **CPU cache / MMIO / 外设寄存器 的完整硬件路径图**，这对理解工业控制板卡、FPGA、PCIe设备非常关键。