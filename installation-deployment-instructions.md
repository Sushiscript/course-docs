# Installation Deployment Instructions
`sushicript` `安装部署说明`

## 版本修改历史

+ 版本号: v0.0.1
  + 修改内容: 实现sushicript最基本最核心的内容。
  + 修改日期: 2018年7月8日

## 引言 
+ 编写目的：本说明文档希望对sushicript的安装部署过程加以说明，为用户提供帮助。
+ 系统背景：本软件旨在提供一种更好的命令式语言的解决方案，背景是现有的方案有一定的缺陷性。
+ 适应人群：本软件适用于熟悉命令行交互、对命令行交互有一定使用需求的、具有编程背景的人。
+ 术语定义：
  + sushiscript：一门基于系统先有命令解释器支持语法的新语言。
  + sushi: sushiscript执行文件。

## 安装环境：
  + linux或类unix系统
  + python's conan
  + cmake(minimum version 3.5)
## 系统软硬件安装与参考配置：
  + 软件参考：ubuntu18.04 LTS
  + 硬件参考：CPU I5 、 RAM 8GB、 ROM 1GB

## 安装的步骤
1. 下载源代码，打开系统的命令行交互程序并导航到源代码目录
2. 输入以下指令
```
mkdir build && cd build
conan remote add bincrafters https://api.bintray.com/conan/bincrafters/public-conan
conan install ..  -s compiler.libcxx=libstdc++11 --build=missing
cmake .. && make install
```
## 安装成功的测试方法
输入以下指令`sushi -v`，可以查看到当前版本号。

## 常见问题解决方法
1. 如何安装python及conan
  建议您访问python与conan的相关文章寻找安装引导。
2. make install提示需要root权限
  请改为键入以下指令
  `sudo make install`
  并在提示输入密码的时候键入您的管理员密码。
3. make install报错
  请仔细检查安装中的第二步是否是按照顺序逐行输入，检查中途有无错误提示。