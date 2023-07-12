---
title: Zephyr 多级中断
image: https://yun.weenas.com:8006/i/2023/06/04/647c0b7345785.jpg
date: 2019-01-17
tags:
description: "采用一种很巧妙的方式实现多级中断，为中断设计提供参考。"
categories:
 - Zephyr
keywords:
 - zephyr
 - interrupt
---

理解中断子系统有利于我们更好的理解操作系统的工作原理，每个OS实现中断子系统的方式各不相同。 Linux一般采用动态注册的方式可以在系统运行过程中把中断Handler注册给OS，而Zephyr系统为了减小image大小采用静态注册的方式，在编译过程中把所有IRQ_CONNECT解析为isr table。

<!--more-->

## 一级中断

如果系统只需要一级中断，生成的isr table（中断向量表）看起来比较简单，编译后生成的路径在`zephyr/isr_tables.c`

```c
struct _isr_table_entry {
    void *arg;
    void (*isr)(void *);
};

struct _isr_table_entry __sw_isr_table _sw_isr_table[160] = {
    {(void *)0x1979e0, (void *)0x114fc1},
    {(void *)0x1979ec, (void *)0x114fc1},
    {(void *)0x2, (void *)0x109e01},
    {(void *)0x3, (void *)0x109e01},
    {(void *)0x4, (void *)0x109e01},
    {(void *)0x5, (void *)0x109e01}
};
```

可以看到，在32位系统中isr table的每一个entry占用8 bytes，前4 bytes为参数，后4 bytes为handler。isr table的大小由`CONFIG_NUM_IRQS`决定，可以根据需要调整。

Zephyr系统启动时会把isr table地址设置给CPU的VTOR寄存器(以CM4为例)，如果有中断触发，CPU可以根据中断号进入相应的中断服务程序处理。

## 多级中断

CPU的中断控制器能处理的中断数量往往有限制，ARM Cortex-M的中断控制器一般能外接64个IRQ和64个FIQ，对于外设较多的芯片可能会不够用。这时需要级连额外的中断控制器来扩展中断数量，这样新增的中断控制器（INTC）就称为2级中断控制器。如果一个2级中断控制器还不能满足系统需求，还可以使用多个2级中断控制器或者增加3级中断、4级中断。当然多级因为需要软件分发中断，级数越多效率也会越低。

### 相关宏定义

Zephyr系统的[多级中断](https://docs.zephyrproject.org/latest/kernel/other/interrupts.html)因为是采用静态生成的方式，实现起来会稍微复杂一点。首先，我们需要理解几个关于多级中断宏的含义：

- CONFIG_MULTI_LEVEL_INTERRUPTS
多级中断总开关
- CONFIG_2ND_LEVEL_INTERRUPTS
2级中断开关
- CONFIG_3RD_LEVEL_INTERRUPTS
3级中断开关
- CONFIG_2ND_LVL_ISR_TBL_OFFSET
2级中断向量表的偏移，即1级中断的数量，因为2级中断向量表是放在1级中断向量表的后面
- CONFIG_NUM_2ND_LEVEL_AGGREGATORS
2级中断控制器的数量
- CONFIG_MAX_IRQ_PER_AGGREGATOR
多级中断控制器上最多有多少个中断项，如果每个中断控制器的中断数量不一样就取最大值
- CONFIG_2ND_LVL_INTR_00_OFFSET
第一个2级中断控制器的中断号
- CONFIG_2ND_LVL_INTR_01_OFFSET
第二个2级中断控制器的中断号

这些宏用于配置多级中断，编译脚本会根据以上的宏生成中断向量表。

### 定义中断号

如果是一级中断，调用`IRQ_CONNECT`接口时传入的IRQ NUM即是相应的实际中断号。多级中断涉及到每一级中断的中断号，Zephyr系统采用每一级中断号占用1个byte的方式将多级中断号定义在一个DWORD中。

下图是官方文档的示例，用于展示如何定义中断号：

```
         9             2   0
   _ _ _ _ _ _ _ _ _ _ _ _ _          (LEVEL 1) 
         |         |   |
  5      |         A   |
_ _ _ _ _|_ _         _|_ _ _ _ _ _   (LEVEL 2)
  |   |                       |
  |   C                       B
 _|_ _ _ _ _ _                        (LEVEL 3)
         |
         D
```

根据示例，ABCD四个设备定义的中断号如下，可以看到除了第一级中断的起始数是从0开始外，从第二级开始所有中断号都是从1开始编号，因为0用于代表没有设备。

```
A -> 0x00000004
B -> 0x00000302
C -> 0x00000409
D -> 0x00030609
```

### 生成中断向量表

现假设上述宏的定义如下

```c
#define CONFIG_MULTI_LEVEL_INTERRUPTS 1
#define CONFIG_2ND_LEVEL_INTERRUPTS 1
#define CONFIG_3ND_LEVEL_INTERRUPTS 1
#define CONFIG_2ND_LVL_ISR_TBL_OFFSET 64
#define CONFIG_NUM_2ND_LEVEL_AGGREGATORS 2
#define CONFIG_MAX_IRQ_PER_AGGREGATOR 32
#define CONFIG_2ND_LVL_INTR_00_OFFSET 2
#define CONFIG_2ND_LVL_INTR_01_OFFSET 9
```

系统将生成中断向量表

```
isr_table
     +---------+----------+
     |  isr0   |          |
     |  isr1   |          |
     |  isr2   |          |
     |  isr3   | LEVEL 1  |
=> A |  isr4   |          |
     |   ...   |          |
     |  isr63  |          |
     +---------+----------+
     |  isr64  |          |
     |  isr65  |          |
=> B |  isr66  | LEVEL 2  |
     |   ...   |          |
     |  isr95  |          |
     +---------+----------+
     |  isr96  |          |
     |  isr97  |          |
     |  isr98  |          |
=> C |  isr99  | LEVEL 2  |
     |   ...   |          |
     |  isr127 |          |
     +---------+----------+
     |  isr128 |          |
     |  isr129 |          |
=> D |  isr130 | LEVEL 3  |
     |   ...   |          |
     |  isr159 |          |
     +---------+----------+

```

### 中断注册

定义了中断号后，注册多级中断和注册普通中断一样，例如我们要注册上面设备C的中断：

```c
IRQ_CONNECT(0x00000409, DEVICE_C_IRQ_PRIO, uwp_ictl_isr, 0, 0);
irq_enable(0x00000409);
```

这里只是示例，所以直接写的数字，在实际代码中，硬件相关的参数应该都定义在device tree，再根据生成的宏定义来写代码。

### 中断处理

多级中断服务程序被触发后，需要根据中断状态寄存器对中断进行分发。分发函数需要当前的中断状态以及当前中断控制器在isr_table的起始位置，起始位置可以根据上图获取到，分发时按中断状态的每一位依次分发，保证每一个中断都可以被处理到。

```c
static ALWAYS_INLINE void uwp_dispatch_child_isrs(u32_t intr_status,
                              u32_t isr_base_offset)
{
    u32_t intr_bitpos, intr_offset;

    /* Dispatch lower level ISRs depending upon the bit set */
    while (intr_status) {
        intr_bitpos = find_lsb_set(intr_status) - 1;
        intr_status &= ~(1 << intr_bitpos);
        intr_offset = isr_base_offset + intr_bitpos;
        _sw_isr_table[intr_offset].isr(
            _sw_isr_table[intr_offset].arg);
    }
}

static void uwp_ictl_isr(void *arg)
{
    struct device *dev = arg;

    struct uwp_ictl_config *config = DEV_CFG(dev);

    volatile struct uwp_intc *intc = INTC_STRUCT(dev);

    uwp_dispatch_child_isrs(uwp_intc_status(intc),
            config->isr_table_offset);
}
```

## 结束

通过对Zephyr系统多级中断流程的分析，不难看出其实Zephyr系统已经封闭的大部份的工作，只需要理清系统中中断控制器的拓扑结构，实现少量代码即可控制多级中断。
