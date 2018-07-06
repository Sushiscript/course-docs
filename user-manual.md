## 序言

sushiscript，su同时是sub和super的前缀，sh代表shell，script代表着重与脚本而不是命令行交互。本软件旨在提供一种更好的命令式语言的解决方案。

## 安装与配置

### 安装环境：

  + linux或类unix系统
  + python's conan
  + cmake(minimum version 3.5)

### 系统软硬件安装与参考配置：

  + 软件参考：ubuntu18.04 LTS
  + 硬件参考：CPU I5 、 RAM 8GB、 ROM 1GB

### 安装的步骤

1. 下载源代码，打开系统的命令行交互程序并导航到源代码目录
2. 输入以下指令
```
mkdir build && cd build
conan remote add bincrafters https://api.bintray.com/conan/bincrafters/public-conan
conan install ..  -s compiler.libcxx=libstdc++11 --build=missing
cmake .. && make install
```
### 安装成功的测试方法

输入以下指令`sushi -v`，可以查看到当前版本号。

### 常见问题解决方法

1. 如何安装python及conan
  建议您访问python与conan的相关文章寻找安装引导。
2. make install提示需要root权限
  请改为键入以下指令
  `sudo make install`
  并在提示输入密码的时候键入您的管理员密码。
3. make install报错
  请仔细检查安装中的第二步是否是按照顺序逐行输入，检查中途有无错误提示。

## 语言参考

- [language specification](https://github.com/Sushiscript/sushiscript/blob/master/docs/language-specification.md)

### 整合与命令行接口

命令行接口部分工作总体难度较低，可以划分为
1. 制定命令行参数标准。这里参考了`python`，`javac`，`dotnet`等的参数设计，最终敲定。
2. 实现命令行参数处理部分并添加测试样例。这个就是简单的字符序列的处理，测试也比较容易。
3. 调用前几层的工作转换`sushiscript`为`shell script`。这是整个程序的核心部分，中间需要对前几个层次的异常做出处理。
4. 将各个`pipeline`部分串联起来，比如使用一个`config`来传递参数，`run`和`build`公用相同的核心处理函数。

## 附录

[Software Requirements Specification](https://github.com/Sushiscript/course-docs/blob/master/Software%20Requirements%20Specification.md)

[Installation-deployment-instructions](https://github.com/Sushiscript/course-docs/blob/master/installation-deployment-instructions.md)

[Software-design](https://github.com/Sushiscript/course-docs/blob/master/software-design.md)
