# nvim

### nvim利用ccls进行代码补全
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
      "rootPatterns": [".ccls-root", "compile_commands.json", ".git/", ".hg/", ".vscode", ".vim/"],
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
> 注：里面不能使用执行命令
然后每次打开nvim，ccls都会检查这个文件，并且根据这个文件进行补全配置。

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

**工程的情况下使用compile_commands.json**

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
