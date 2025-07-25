---
title: Kconfig 简介
slug: kconfig_usage
image: https://yun.weenas.com:8006/8l3G1D.png
date: 2022-06-05
tags:
description: "通过Kconfig为软件工程增加可配置功能，模块编译的基础。"
categories:
  - 工具
keywords:
  - kconfig
  - kconfiglib
---

## 简介

Kconfig来源于Linux Kernel，用于在项目中对各种选项进行配置，以达到仅修改config文件就可以控制开关Module（Feature）的目的。也就是说，使用同一份源代码配合不同的config文件，可以编译出功能各异的bin文件。

## Kconfig实现方式

早期的Kconfig只能在Kernel中使用，现在由于它的易用性被广泛使用于各种大型项目，包括NuttX，buildroot，crosstool-NG，uClibc，openWRT，Zephyr等项目。目前主流的Kconfig有两种实现方式：kconfig-frontends和Kconfiglib

### kconfig-frontends

kconfig-frontends是C语言实现的版本，使用的是Kernel的Kconfig源码，独立于Kernel进行管理。
优点
- 兼容性最好

缺点
- 不支持跨平台
- 默认只能支持输出.config

### Kconfiglib

Kconfiglib是用Python实现的一个库，兼容Kconfig的所有语法。
优点
- 跨平台使用
- 能同时输出.config和config.h

> .config和config.h的内容完全一致，只是为了给不同的文件使用而采用不同的表达方式
> .config用于把配置输出给Makefile或者CMake: CONFIG_XXX=100
> config.h用于把配置输出给源代码: #define CONFIG_XXX 100

## 常用语法

官方说明文档：https://www.kernel.org/doc/html/latest/kbuild/kconfig-language.html

### config

config是Kconfig最基础的语法，每个config会生成一个CONFIG_XXX配置

```sh
config XXX
    bool "Enable XXX"
    default y
    help
        Enable XXX module.
```

config支持不同的配置类型，常用的有bool，string，int，hex四种类型

### menu

menu可以生成二级菜单，多个menu同时使用可以生成多级菜单。
menu不会生成config配置，作用仅是为了组织config结构

```sh
menu "2nd Menu"
endmenu
```

> 注意：menu和endmenu需要配对使用

### menuconfig

menuconfig是menu和config的结合体，不仅生成菜单，同时也会生成config配置。
和menu不同的是只有Enable这个配置才能进入一下级菜单，一般用于模块和模块内部选项配置

```sh
menuconfig MENU_XXX
    bool "MENU XXX"
    default y
    help
        Menu XXX
```

### choice

choice提供单选功能，当只能在多个配置中选择一个时可以使用，比如选择平台使用VDK，FPGA或者SOC时，使用choice可以避免人为的配置错误发生。

```sh
choice
    bool "Choice Sample"
    default CHOICE_C
config CHOICE_A
    bool "Choice A"
config CHOICE_B
    bool "Choice B"
config CHOICE_C
    bool "Choice C"
endchoice
```

> 注意：choice和endchoice需要配对使用

### if, depends on, select

这些是Kconfig的条件用法，可以从配置上去控制模块之间的依赖关系

```sh
config MODULE_A
    bool "module a"
if MODULE_A
config MODULE_B
    bool "module b"
endif
config MODULE_C
    bool "module c"
    depends on MODULE_A
config MODULE_D
    bool "module d"
    select MODULE_E
config MODULE_E
    bool "module e"
```

### source

source的作用是包含下级Kconfig文件，方便开发时把Kconfig设计为树形结构，在模块内部添加编写config配置。不管config在哪个Kconfig文件，所有的config都是全局可见的，所以if，depends on和select可以引用其它模块的config配置。

## 示例

### Kconfig文件

```sh
mainmenu "Sample Project"
config AUTHOR
    string "Author"
    default "Sample"
    help
        Author of this project.
config MAJOR_VER
    int "Major version"
    default 1
    help
        Major version of this project.
config SUB_VER
    int "Sub version"
    default 12
    help
        Sub version of this project.
menu "Config Samples"
config BOOL_XXX
    bool "Enable BOOL XXX"
    default y
    help
    Enable BOOL XXX.
config STRING_XXX
    string "STRING XXX"
    default "string_xxx"
    help
    Set string
config INT_XXX
    int "INT XXX"
    default 100
    help
    Set int
config HEX_XXX
    hex "HEX XXX"
    default 0x100
    help
    set hex
endmenu
menu "Menu Sample"
menu "2nd Menu"
menu "3rd Menu"
menu "4th Menu"
endmenu
endmenu
endmenu
endmenu
menu "Menuconfig Sample"
menuconfig MENU_XXX
    bool "MENU XXX"
    default y
    help
    Menu XXX
if MENU_XXX
config MENU_SUB1_XXX
    bool "MENU SUB1 XXX"
config MENU_SUB2_XXX
    bool "MENU SUB2 XXX"
    default y
endif
endmenu
choice
    bool "Choice Sample"
    default CHOICE_C
config CHOICE_A
    bool "Choice A"
config CHOICE_B
    bool "Choice B"
config CHOICE_C
    bool "Choice C"
endchoice
menu "Condition Sample"
config MODULE_A
    bool "module a"
if MODULE_A
config MODULE_B
    bool "module b"
endif
config MODULE_C
    bool "module c"
    depends on MODULE_A
config MODULE_D
    bool "module d"
    select MODULE_E
config MODULE_E
    bool "module e"
endmenu
```

### 运行

下载Kconfiglib: https://github.com/ulfalizer/Kconfiglib

```sh
python Kconfiglib/menuconfig.py Kconfig
```
