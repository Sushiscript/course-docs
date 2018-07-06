## 引言

### 目的

本需求分析说明书对本项目第一阶段的内容进行分析，对需求细节和实现方式进行了较为详细的阐述。本需求说明书供软件需求提供人员、
软件的概要设计人员、软件的开发人员、软件的测试人员使用，并作为产品验收确认的依据。

### 范围

`sushiscript`是一个以bash为后端的静态类型脚本语言，目前已完成基础的语言框架，包括parsing、简单的分析和代码生成等部分。

### 定义、简写和缩略语

sushiscript：一门基于系统先有命令解释器支持语法的新语言。

### 参考文献

GB/T 9385-2008 《计算机软件需求说明编制指南》  
ASTM E1340-1996 《计算机系统快速原型法标准指南》


## 总体描述

### 产品描述

和简化的传统编译器架构类似，`sushiscript`实现分为三大部分：编译器前端，分析与检查，以及代码生成。但为了实现简单和分工方便的考虑：前端部分目前细分为**词法分析**（`sushi/lexer`）和**语法分析**（`sushi/parser`）两阶段，分析与检查又分为**作用域分析**（`sushi/scope`）以及**类型检查**（`sushi/type-system/type-check`）两阶段。每个阶段在实现上均假设输入的各种性质完全正确，并且各自产生自己的错误，顶层的**流水线**（`sushi/pipeline`）按顺序执行每一个阶段，并把上一阶段的输出传递给下一阶段，如果在任何一个阶段遇到错误，则将错误输出并退出。

### 运行环境

安装环境：
  linux或类unix系统
  python's conan
  cmake(minimum version 3.5)
  
系统软硬件安装与参考配置：
  软件参考：ubuntu18.04 LTS
  硬件参考：CPU I5 、 RAM 8GB、 ROM 1GB

### 产品功能

支持文件名替换（通配符匹配）、管道、here文档、命令替换、变量，以及条件判断和循环遍历。

### 用户特点

熟悉命令行交互
对命令行交互有一定使用需求
具有编程背景

### 软件设计约束及有关说明

软件的设计和开发过程需要严格按照规范要求，根据软件的设计方案来进行。
软件开发过程应遵循软件工程规范，对过程和版本进行管理和控制。

### 需求分配

暂无

### 接口

#### 外部接口

外部接口的用户界面部分按Linux或类uxic应用软件用户界面的规范来设计，界面设计风格与Linux环境保持一致，便于用户使用。

#### 软件接口

python's conan
cmake(minimum version 3.5)

#### 硬件接口

CPU I5 、 RAM 8GB、 ROM 1GB

#### 控制和操作

sushiscript的最终交付形式为代码包，控制该软件运行的方式为命令行打开。

sushiscript提供文件名替换（通配符匹配）、管道、here文档等功能，各个功能项的设置及使用应符合程序员使用shell的操作习惯，通过常用的鼠标点击，键盘输入来完成启动和使用软件的过程，控制信号均由鼠标和键盘进行输入。

## 具体需求

### 性能需求

#### 精度需求

要按照严格的数据格式输入，对符合数据格式要求的输入进行提示。 

#### 时间特性需求

软件启动时间：小于1s。

系统实时响应时间：软件使用过程中，对用户在各个功能模块的鼠标点击、键盘输入等操作事件的响应时间需在用户能够容忍的范围之内，要求小于1秒。  
数据的转换和传送时间：对软件不同模块间的数据交互，要求数据的转换和传送时间不得超过500ms。  
数据更新时间：500ms。  

### 灵活性需求

sushiscript能够支持鼠标、键盘等多种操作方式的使用。软件的设计和实现需要考虑到运行环境的变化，并能够在运行环境变化的情况下正常使用。同时，软件需要兼容其他软件接口的变化，以保证在不同运行环境，不同软件接口的情况下的正常使用。

具体要求如下：
操作方式：软件支持多种操作方式，鼠标、键盘和菜单等。
运行环境：软件的设计和实现需要考虑其运行环境的变化，并能对不同的运行环境提供支持。具体而言，软件应支持ubuntu16.04 LTS及以上版本的操作系统，支持Linux/类Unix环境。
同其他软件接口：当其他软件的接口发生变化时，sushiscript应能够适应接口的变化。
精度和有效时限：灵活性要求软件能够方便的适应精度和有效时限的变化。

### 功能需求

经过多次需求调研，提出sushiscript软件的功能需求：

sushiscript实现分为三大部分：编译器前端，分析与检查，以及代码生成。

前端部分目前细分为**词法分析**（`sushi/lexer`）和**语法分析**（`sushi/parser`）两阶段，分析与检查又分为**作用域分析**（`sushi/scope`）以及**类型检查**（`sushi/type-system/type-check`）两阶段。

每个阶段在实现上均假设输入的各种性质完全正确，并且各自产生自己的错误，顶层的**流水线**（`sushi/pipeline`）按顺序执行每一个阶段，并把上一阶段的输出传递给下一阶段，如果在任何一个阶段遇到错误，则将错误输出并退出。

### 数据需求

#### 数据采集的要求

输入源：键盘输入
输入介质和设备：键盘 鼠标

#### 数据输出需求

输出介质和设备：显示器 文件

## 故障处理需求

### 软件运行故障

在使用软件的过程中，当出现计算机断电，计算机内存不足等情况时，sushiscript将出现运行故障。运行故障发生时，sushiscript的各个功能模块将无法正常使用，启动相关功能按钮都无法进行正常的操作。

对由于计算机断电引发的软件运行故障，用户在重新给计算机供电后，可以通过重新启动计算机，并启动sushiscript的方式恢复软件的正常运行与使用。
对由于计算机内存不足引发的软件运行故障，建议暂时关闭软件。用户应检查并解决计算机内存不足的问题，内存使用情况正常后，sushiscript将恢复正常的运行与使用。
出现软件运行故障并进行修复后，应确保sushiscript功能的完整性，不能发生因软件运行故障而导致工具无法继续使用的情况。

### 软件使用故障

在软件的使用过程中，如果出现软件使用故障，应当具有报警信息提示。 

1)	当软件依赖的文件损毁或丢失时，软件以对话框的形式进行提示，报告损毁或丢失的文件等相关错误，以帮助用户及时修复软件的正常功能。

2)	对软件需要用户输入项的情况，如果发生缺少输入项、输入项格式错误或不符合规则等情况，软件应以合理的方式予以提示。

3)	为了防止用户由于未及时保存而导致信息丢失的情况，软件提供定时保存机制，每隔一定时间自动对信息进行保存，从而保证用户数据的安全。

## 质量需求

1)	软件的功能实现必须符合常用的主流shell软件的使用方法和操作习惯。

2)	要求可配置型强，便于使用者对工具的使用以及定制。

3)	采用可行、合理、高效的方式进行开放性的设计和实现。

4)	软件具有很强的适应能力，并且便于维护，不仅能很好的满足当前的需求，而且应当为后期可能的开发的工作提供很好的扩展和维护接口。
