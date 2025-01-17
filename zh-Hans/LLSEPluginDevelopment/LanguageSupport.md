# 📋 脚本引擎 - 特定脚本语言开发须知

## 🌏 开发语言支持情况

通过 [ScriptX](https://github.com/Tencent/ScriptX) 项目的支持，脚本引擎使用同一套源代码，对多种开发语言进行了适配。    
同时，由于API保持一致，使各种语言得以共享一份开发文档，大大降低了维护难度。

目前，脚本引擎支持使用如下语言编写插件：

| 语言后端           | 备注                                             |
| ------------------ | ---------------------------------------------- |
| `JavaScript`       | 使用 QuickJS 引擎运行插件，支持 ES Module 机制      |
| `Lua`              | 使用 CLua 引擎运行插件                            |
| `Node.js`          | 改造 Node.js 使其适合嵌入工作，支持 npm 包管理       |
| `Python`   | 使用 CPython 引擎运行插件，支持 pip 包管理          |

> [!INFO]
>
> 如果需要使用 C++、Go、.NET 等编译式语言编写插件，请移步 [主页](zh_CN/) 查看其他语言文档

## JavaScript 语言支持说明

- 使用 QuickJS 引擎对简单的 JavaScript 插件提供支持，轻量级引擎资源占用较少
- QuickJS 当前版本支持到 ES2020 语言特性，同时原生支持 ESModule 模块机制，可以方便开发者进行项目管理
- 暂不支持包管理机制，如果需要请使用 Node.js 进行插件开发，使用 npm 包管理
- 在BDS控制台中使用`jsdebug`命令进入和退出 QuickJs 交互式命令行环境。此功能便于编写插件时进行一些简单的测试

## Lua 语言支持说明

- 使用 CLua 引擎，支持使用 require 进行简单的项目管理
- 由于 Rocks 包管理机制需要引入编译器，因此暂不提供相关实现。如果需要依赖扩展可以手动编译后引入项目进行使用（如 SQLite 等常用库）
- 在BDS控制台中使用`luadebug`命令进入和退出 Lua 交互式命令行环境。此功能便于编写插件时进行一些简单的测试

## Node.js 支持说明

- 脚本引擎 通过自行实现 Node.js 启动代码，使其可以在嵌入式模式下工作，并实现了不同插件的执行环境隔离
- 自行编写接口实现了对 npm 的 programmic 支持。支持通过 package.json 安装第三方扩展依赖包

##### ⭐ **Node.js 插件打包 & 部署方法**

- 在插件编写完成之后，请将 `package.json` 以及所有插件源码打包为一个zip压缩包，并**将文件名后缀修改为 .llplugin**
- `node_modules` 目录请勿打包在压缩包之中
- 将 **.llplugin** 文件作为插件分发，安装插件时直接将此文件放置到 plugins 目录即可
- 在开服时，脚本引擎会自动识别 **.llplugin** 文件，将其解压到`plugins/nodejs/插件名`目录，并在目录中自动执行 `npm install`安装依赖包，整个过程无需人工干预

## Python 语言支持说明

- 使用 CPython 引擎，Python版本为3.10.9。支持使用 pip 包管理机制为插件安装第三方扩展依赖包，支持多文件插件开发和 import，支持现代项目管理机制
- 在BDS控制台中使用`pydebug`命令进入和退出 Python 交互式命令行环境。此功能便于编写插件时进行一些简单的测试

**Python 单文件插件开发方法**

- 为了方便不熟悉Python的开发者快速上手，我们提供了一种简单的 Python 插件支持：单文件插件。
- 单文件插件类似QuickJs和Lua插件：单个.py源码文件，直接放置到 plugins目录中，在开服时将被加载运行
- 单文件插件缺点：不支持插件元数据储存、不支持源码分文件、不支持pip第三方包。仅用于开发者熟悉LLSE的Python环境，或者开发非常简易的插件而使用。

⭐ **Python 多文件插件开发方法**

- 对于大型、正式的Python插件，强烈建议使用此方法进行开发。相比于单文件插件，此完整方案支持完善的元数据储存、支持源码分文件编写、以及最重要的：第三方pip包支持。
- 首先，你需要创建一个新的目录，在其中初始化和项目相关的元数据信息。LLSE 要求 使用`pyproject.toml` 项目文件进行元数据储存，类似 Node.Js 的`package.json`。`pyproject.toml` 项目文件将通过下面的包管理器工具自动生成：
- 推荐使用支持现代项目特性的 PDM 包管理器（[pdm-project/pdm](https://github.com/pdm-project/pdm)）进行插件项目的创建和维护。PDM起到的的作用类似于 Node.Js 中的npm。使用方法：
  - 首先，使用`pip install --user pdm`命令安装 pdm
  - 安装完成之后，在你想要创建 Python 项目的目录启动命令行执行`pdm init`命令。根据pdm工具的提示创建新项目，填写项目的相关信息。
  - 如果需要安装依赖包，在项目目录执行`pdm add <依赖包名>`即可
  - 所有的项目元数据和依赖数据都会被自动储存在`pyproject.toml` 中，无需手动编写。你也可以打开此文件修改版本号、描述等元数据信息
  - 另外，你也可以直接把项目依赖手动写在`requirements.txt`当中。在安装插件时，`pyproject.toml`和`requirements.txt`中描述的依赖都将被处理并自动安装。
- 接下来编写代码。**注意：Python插件使用`__init__.py`文件作为入口点**。加载插件时，Python解释器会读取并执行这个文件。在`__init__.py`中编写插件的启动代码。
- 如果有需要，可以创建多个源码文件，并且通过import进行互相调用。

##### ⭐ **Python 多文件插件打包 & 部署方法**

- 在插件编写完成之后，请将 `pyproject.toml`以及所有插件源码打包为一个zip压缩包，并**将文件名后缀修改为 .llplugin**
- `__pycache__` 和`__pypackages__` 等目录请勿打包在压缩包之中
- 将 **.llplugin** 文件作为插件分发，安装插件时直接将此文件放置到 plugins 目录即可
- 在开服时，脚本引擎会自动识别 **.llplugin** 文件，将其解压到`plugins/python/插件名`目录，并在目录中自动执行 `pip install`安装依赖包，整个过程无需人工干预
