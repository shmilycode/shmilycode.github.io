## 从零开始？

你是否需要从零开始配置 `WebAssembly` 开发环境？不好意思，标题可能骗了你，你可以不需要，你只需要直接安装 `Emscripten SDK` 就可以了，下面会对这种方式做介绍。但我比较折腾，所以自己下代码编译然后配置环境，因为我有一种....精神（好吧，其实是因为我系统版本较低，且下载 `SDK` 慢的我想撞墙），下面我会分别介绍这两种做法。

### 安装 Emscripten SDK

#### 1. 环境准备

在安装 `Emscripten SDK` 前， 你需要确保你的系统环境下有这些工具，且版本也是对的：

* **Git**，这个系统应该默认都有，在Windows上需要在这里[安装Git](https://git-scm.com/)

* **CMake**，可以直接使用 `apt-get` 或者 `brew` 安装，如果是 Windows系统，戳[这里](https://cmake.org/download/)，版本至少为3.4.3。

* **编译工具**， Linux上为 `GCC` ，OS X上，安装XCode，Windows上安装[Visual Studio 2015 Community with Update 3](https://visualstudio.microsoft.com/downloads/)或更高的版本。

* **Python2.7.x**，这个基本系统自带的。

除了上面这些工具外，其实还需要 `LLVM` ，但 `LLVM` 会在安装 `SDK` 时被自动编译和安装，所以这里不用管他。

#### 2. 编译Emscripten

接照下面的命令进行编译安装：

``` shell
$ git clone https://github.com/juj/emsdk.git
$ cd emsdk
$ ./emsdk install sdk-incoming-64bit binaryen-master-64bit
$ ./emsdk activate sdk-incoming-64bit binaryen-master-64bit
```

安装完成后，将 `Emscripten` 的环境变量配置到当前的命令行窗口下：

``` shell
$ source ./emsdk_env.sh
```

在 Windows中，`./emsdk` 使用 `emsdk` 代替，`source ./emsdk_env.sh` 使用 `emsdk_env` 代替。

#### 3. 验证

使用C++写一个简单的`hello_world.cpp`代码，这里代码就不贴了吧，然后使用emcc进行编译。

* 在使用 emcc 命令时，要带着 -s WASM=1 参数（不然，默认将会编译成asm.js）。
* 如果我们想让 Emscripten 生成一个我们所写程序的HTML页面，并带有 wasm 和 JavaScript 文件，我们需要给输出的文件名加 .html 后缀名。
* 最后，当我们运行程序的时候，我们不能直接在浏览器中打开 HTML 文件，因为跨域请求是不支持 file 协议的。我们需要将我们的输出文件运行在HTTP协议上。

``` shell
$ emcc hello_world.cpp -s WASM=1 -o hello_world.html
```

使用 emrun 命令来创建一个 http 协议的 web server 来展示我们编译后的文件：

``` shell
$ emrun --no_browser --port 8080 .
```

使用浏览器打开 http://localhost:8080能看到由 `Emscripten` 自动生成的一个html页面，并在页面下方的模拟终端里看到"hello,world!"输出。

如果上面的都安装完成了，那就请忽略文章下面的内容。

### 从源码安装 Emscripten

#### 1. 系统环境说明

**系统**：Unbuntu14.04
**gcc版本**：6.5.0
**python版本**：2.7.6

自己编译的话系统环境不会有太大的关系，因为版本不对的话我们可以自己下源码编译。

#### 2. 安装必要工具

可以先更新下源，虽然可能没什么用：

``` shell
sudo apt-get update
```

**1.** 如果系统没有python2.7，需要自己安装：

``` shell
$ sudo apt-get install python2.7
```

**2.** 安装node.js，尝试直接使用包安装：

``` shell
$ sudo apt-get install nodejs
```

在最后运行 `Emscripten` 的时候，发现安装的版本太低（远古版本0.10.37），导致没办法用，所以需要自己编译 `node.js` 模块。

``` shell
// 卸载之前已对安装的低版本nodejs，如果没有则忽略
$ sudo apt-get --purge remove nodejs
// 下载源码包
$ wget https://nodejs.org/dist/v10.15.3/node-v10.15.3-linux-x64.tar.gz
// 解压
$ tar -zxvf node-v10.15.3-linux-x64.tar.gz
// 编译安装
$ cd node-v10.15.3
$ sudo ./configure
$ make
$ sudo make install
//查看当前安装的node版本，应该已经变为v10.15.3
$ node -v
```

**3.** 安装 `gcc` 和 `cmake`

``` shell
$ sudo apt-get install build-essential
$ sudo apt-get install cmake
```

好吧，安装完后面运行时发现cmake版本又是太低了，所以自己下源码(v3.4.3)编一个

``` shell
// 卸载老版本，如果有的话
$ sudo apt-get --purge remove cmake
$ sudo wget https://cmake.org/files/v3.4/cmake-3.4.3.tar.gz
$ tar -zxvf cmake-3.4.3.tar.gz
$ cd cmake-3.4.3
$ ./configure
$ make
$ sudo make install
// 查看版本
$ cmake --version
```

**4.** 安装 `git-core` 

``` shell
$ sudo apt-get install git-core
```

**5.** 安装 `Java`

``` shell
sudo apt-get install default-jre
```

**6.** 从源码开始编译 `Fastcomp (LLVM + Clang)`，这个没有其他选择。

* 创建目录，用于在存放代码和编译中间文件，放哪里都可以，在最后 `Emscripten` 会通过配置文件 `~/.emscripten` 指定 `Fastcomp` 的位置。

  ``` shell
  $ mkdir fastcomp
  $ cd fastcomp
  ```

* 下载 `fastcomp LLVM` 仓库代码

  ``` shell
  $ git clone https://github.com/emscripten-core/emscripten-fastcomp
  ```

* 下载 `emscripten-core/emscripten-fastcomp-clang` 仓库代码，必须下载到 `emscripten-fastcomp` 的`tools/clang`目录，如下。

  ``` shell
  $ cd emscripten-fastcomp
  $ git clone https://github.com/emscripten-core/emscripten-fastcomp-clang tools/clang
  ```

* 创建编译目录并编译

  ``` shell
  $ mkdir build
  $ cd build
  // 这里CMake的版本必须是3.4.3及以上
  $ cmake .. -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="host;JSBackend" -DLLVM_INCLUDE_EXAMPLES=OFF -DLLVM_INCLUDE_TESTS=OFF -DCLANG_INCLUDE_TESTS=OFF
  // 编译
  $ make -j3
  ```

  编译成功后就 `Fastcomp` 相关的就算完成了。

**7.** 拉取 `emscripten-core/emscripten` 仓库代码，可以拉到一个新目录下。

``` shell
$ git clone https://github.com/emscripten-core/emscripten.git
```

**8.** 配置 `Emscripten`

* 进入下载的 `emscripten` 仓库
* 运行命令：

  ``` shell
  ./emcc --help
  ```

  你应该能看到 `Welcome to Emscripten!` 或者类似的信息，这表示已经在你的用户根目录下已经成功生成 `.emscripten` 配置文件。

* 编辑 `.emscripten` 文件，在Linux和macOS上，他应该在 `~/.emscripten`，在Windows系统上，他在 `C:/Users/yourusername_000/.emscripten.`。

  ``` shell
  $ vi ~/.emscripten
  ```

  这里需要将相关的变量路径改为他们真实存在的路径，最可能需要修改的路由是`LLVM_ROOT`这个变量的路径，其他的保持默认一般是不会错的。在我的机器上，`LLVM_ROOT`的路径为 `/home/walle/Code/emscripten/fastcomp/emscripten-fastcomp/build/bin`，也就是我们刚才在步骤6下载和编译生成的可执行文件的路径。

  ``` shell
  LLVM_ROOT = os.path.expanduser(os.getenv('LLVM', '/home/walle/Code/emscripten/fastcomp/emscripten-fastcomp/build/bin')) # directory
  ```

全部做完后就OK了，最后使用SDK的那种验证方法进行验证就可以了。

参考：
1. https://emscripten.org/docs/building_from_source/building_emscripten_from_source_on_linux.html#building-emscripten-on-linux
2. http://webassembly.org.cn/getting-started/developers-guide/