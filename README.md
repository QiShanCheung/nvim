# nvim

- [ccls配置](#nvim利用ccls进行代码补全)
- [vimspector](#Vimspector)

## nvim利用ccls进行代码补全
&emsp;coc.vim补全有三种lsp：clangd、ccls、cquery。

### ccls的配置
#### 1. 下载ccls
**MacOS**

``
brew install ccls
``

#### 2. 配置ccls为补全插件
&emsp;ccls官方提供了很多方法，这里说明如何在coc.vim中进行补全。打开nvim，输入`:CocConfig`打开配置文件

~~~json
{
  "languageserver": {
    "ccls": {
      "command": "ccls",
      "filetypes": ["c", "cc","cpp", "c++", "cuda", "objc", "objcpp"],
      "rootPatterns": [".ccls-root", "compile_commands.json", ".git/", ".hg/", ".vscode", ".vim/"], # 指定搜索项目根目录的模式。这些模式可以是文件名、文件夹名或正则表达式。
      "initializationOptions": {
        "highlight": {"lsRanges": true},
        "cache": {
          "directory": ".ccls-cache" # 缓冲文件所在路径
        },
        "clang": {  # 编译器，如果你使用的是gdb编译器，那么可以按照https://github.com/neoclide/coc.nvim/wiki/Language-servers#ccobjective-c进行配置
          "resourceDir": "/usr/lib/clang/13.0.0"
        },
        "client": {
          "snippetSupport": true
        },
        "compilationDatabaseDirectory": ".vscode/"
      }
    }
  }
}
~~~
&emsp;上面的`"compilationDatabaseDirectory": ".vscode/"`代表到.vscode目录下查找compile_commands.json文件，根据compile_commands.json就可以查找到头文件和库文件所在目录，而compile_commands.json是由cMake或make生成的。

**cMake生成compile_commands.json：**
~~~markdown
cmake -DCMAKE_BUILD_TYPE=debug -DCMAKE_EXPORT_COMPILE_COMMANDS=YES -S . -B .vscode
-S  指定源文件根目录，必须包含一个CMakeLists.txt文件
-B  指定构建目录，构建生成的中间文件和目标文件的生成路径  

在当前目录的.vscode目录下生成compile_commands.json

平常cmake只会生成CMakeCache.txt、CMakeFiles、cmake_install.cmake、Makefile。上面的cmake多生成了compile_commands.json等文件。
在coc的配置文件中，我们还写了一个generate_compile_commands函数。有了generate_compile_commands函数，我们就可以通过输入:Gcmake来执行上面的cmake命令。
~~~


#### 3. 小程序的情况下使用.ccls
&emsp;如果你的程序足够小，可以通过在工程目录下编写.ccls文件来让ccls找到自己的头文件。.ccls中的每一行都是一个编译指令：

~~~markdown
-Iinclude

-std=c++17
~~~
> 注：这是 ccls 的配置文件，用于指定编译器标志和头文件目录。

**MacOS特殊情况**

&emsp;在MacOS下，ccls没办法找到系统头文件，这个时候你必须自己编写.ccls文件，在文件中加入如下内容帮助ccls找到头文件:
~~~
-isystem
/usr/local/include
-isystem
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include/c++/v1
-isystem
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/clang/10.0.1/include
-isystem
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include
-isystem
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.14.sdk/usr/include
~~~

#### 4. 工程的情况下使用compile_commands.json

&emsp;如果你使用的是大工程的话，可以考虑编写compile_commands.json文件：

~~~json
[
    {
    "arguments": ["c++", "-Iinclude", "-std=c++11", "main.cpp"],
    "file": "main.cpp"
    }
]
~~~
- arguments
  - 通过将编译指令各个部分拆开称数组。
- file
  - 则指定了你要编译的文件。

但是每次都编写compile_commands.json也很烦，有一些工具可以帮助你自动生成。

**CMake**

&emsp;如果你使用的是CMake，可以加上构建选项`-DCMAKE_EXPORT_COMPILE_COMMANDS=YES`让cmkae自动生成。

**bear**

&emsp;bear工具可以帮助你生成，在MacOS上使用`brew install bear`安装之后即可使用。 bear可以根据多个构建工具来帮助你生成，如果你使用的是make，那么可以使用: `bear make`来自动生成。

**其它的构建工具**

&emsp;[这里](https://github.com/MaskRay/ccls/wiki/Project-Setup#compile_commandsjson)


## Vimspector
[vimspector](https://github.com/puremourning/vimspector)

### 1.安装插件
1. Plugin 'puremourning/vimspector'
2. 安装'gadgets'(debug adapter) - see [here for installation commands](https://github.com/puremourning/vimspector#install-some-gadgets) and [select gadgets to install](https://github.com/puremourning/vimspector#supported-languages)

### 2.下载gadgets
&emsp;进入vimspector的安装目录，执行：
~~~BASH
./install_gadget.py <language-name>
~~~
&emsp;`install_gadget.py`会自动下载`<language-name>`所需的调式适配器并进行相应配置，`--help`可以查看`vimspector`所支持的全部语言。
&emsp;以在Linux环境上打开C/C++支持为例：
~~~BASH
./install_gadget.py --enable-c
~~~
&emsp;`vimspector会`自动下载微软开发的调试适配器`cpptools-linux.vsix`到`your-vimspector-path/gadgets/linux/download/vscode-cpptools/0.27.0/`中。如果是在mac上,linux会被改成mac。

&emsp;如果只想使用脚本添加一个新适配器而不破坏现有适配器，添加 `--update-gadget-config`，如下所示：
~~~BASH
./install_gadget.py --enable-tcl
./install_gadget.py --enable-rust --update-gadget-config
./install_gadget.py --enable-java --update-gadget-config
~~~

&emsp;**gadget目录**

默认情况下，Vimspector使用以下目录查找名为`.gadgets.json`的文件：`<path-to-vimspector>/gadgets/<os>`。此路径作为vimspector变量`${gadgetDir}`。
格式与`.vimspector.json`相同，but only the adapters key is used:

Example:
~~~json
{
  "adapters": {
    "lldb-vscode": {
      "variables": {
        "LLVM": {
          "shell": "brew --prefix llvm"
        }
      },
      "attach": {
        "pidProperty": "pid",
        "pidSelect": "ask"
      },
      "command": [
        "${LLVM}/bin/lldb-vscode"
      ],
      "env": {
        "LLDB_LAUNCH_FLAG_LAUNCH_IN_TTY": "YES"
      },
      "name": "lldb"
    },
    "vscode-cpptools": {
      "attach": {
        "pidProperty": "processId",
        "pidSelect": "ask"
      },
      "command": [
        "${gadgetDir}/vscode-cpptools/debugAdapters/bin/OpenDebugAD7"
      ],
      "name": "cppdbg"
    }
  }
}
~~~

Vimspector的配置可以存在于以下文件中：
~~~
1. your-path-to-vimspector/gadgets/<os>/.gadgets.d/*.json ：这些文件是用户自定义的。
2. 在 Vim 工作目录向父目录递归搜索到的第一个.gadgets.json。
3. .vimspector.json 中定义的adapters。
~~~
编号代表配置文件的优先级，编号越大优先级越高，高优先级的配置文件将覆盖低优先级的配置文件中的的adapters。


### 3.调试适配器配置
&emsp;Vimspector有两类配置：<br/>
- 调试适配器的配置
  - 如何启动或连接到调试适配器
  - 如何 attach 到某进程
  - 如何设置远程调试
- 调式会话的配置
  - 使用哪个调试适配器
  - launch 或 attach 到进程
  - 是否预先设置断点，在何处设置断点
  
**调试适配器配置**

&emsp;该配置就是上述介绍gadget目录中的`.gadgets.json`文件，这个配置在打开 vimspector 对某语言的支持时就已经自动设置好了。

**调试会话配置**

&emsp;项目的调试会话的文件位于以下两个位置：
~~~
1. <your-path-to-vimspector>/configurations/<os>/<filetype>/*.json
2. 项目根目录中的 .vimspector.json
~~~

&emsp;在 macOS 上，我强烈建议对 C 和 C++ 项目使用 CodeLLDB。它真的很棒，依赖性更少，并且不会在另一个终端窗口中打开控制台应用程序。每当打开一个新的调试会话时，vimspector 都会在当前目录向父目录递归搜索，如果查找到了 .vimspector.json，则使用其中的配置，`并将其所在的目录设定为项目根目录`;如果未查找到，则使用 vimspector 安装目录中的配置文件，将打开的文件的目录设置为项目根目录。

**配置选项说明**

`configurations`主要包含以下字段：
- `configuration`：调试器适配器的配置信息，具体格式取决于所使用的适配器。比如在使用 gdb 调试器时，可以通过该参数设置 gdb 的命令行参数。
  - `request`：调试请求的类型。通常为 "launch"，表示启动一个新的进程并开始调试。也可以为 "attach"，表示连接到一个已经运行的进程并开始调试。
  - `program`：要调试的程序的路径。
  - `args`：传递给程序的命令行参数。
  - `cwd`：程序运行的当前工作目录。
  - `externalConsole`：是否使用外部控制台窗口。如果设置为 true，则会在调试过程中使用一个新的终端窗口来显示程序的输出，否则会将输出显示在 vimspector 插件的窗口中。



### 可能遇到的问题
**1. NeoVim: vimspector unavailable: Requires Vim compiled with +python3**
```
pip3 install neovim -i https://pypi.tuna.tsinghua.edu.cn/simple/
-i参数指定pip源
```
