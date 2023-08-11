electron 复合型依赖包，npm 包中的 `postinstall` 脚本（npm 包安装后会自动执行）会去下载 electron 可执行文件以及二进制资源。

## chromium

chromiun 多进程架构资源占用大，也是 electron 应用资源消耗大的原因。

node

## 主要目录

- build 构建 electron 工程相关的脚本，buildflags 编译配置文件。
- chromium_src chromium 代码。
- lib 存放 ts 文件，提供 electron 的 js API，但是这些 API 的原生支持是放在 shell 目录的。
- script 编译所需的工具脚本所在目录。比如执行补丁代码的脚本。
- shell 目录存放 cpp 文件，是 electron Js API 的原生支持。
- spec / spec-main 渲染进程和主进程的测试代码。
- patches 存放的补丁代码，编译 electron 源码前，补丁会修改 node、chromium 的代码，进行集成。

lib 转换为 cpp（shell 里面的原生支持），然后通过 node 的 linkbinding 将 cpp 绑定到 v8 的对象

electron-builder 将 html、css、js 等资源打包成 asar 文件。node require 方法耗费性能

渲染进程 node 环境

在编译器判断执行环境，避免运行时性能损耗。

主进程和渲染进程使用 mojo 框架。

electron 的页面事件，通过 chromium 的对应时机的方法调用实现。

### electron-builder

前端构建
html、css、js生成 asar
准备配置对应的二进制文件
应用签名
压缩 7z压缩工具
生成卸载程序 nsis / squirrel.mac
生成安装包 nsis / squirrel.mac

### electron-updater

文件哈希校验
签名验证
nsis / squirrel.mac 不同的方式安装更新
