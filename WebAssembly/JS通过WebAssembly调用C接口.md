## 调用`WebAssembly` 模块
 
有两种方法可以实现对 `WebAssembly` 模块的调用：如果是使用 `C/C++` 编写的模块，可以直接使用 `Emscripten` 提供的 `ccall()` 函数；另一种是通过加载 `WebAssembly` 代码从而进行调用，这是一种更通用的作法，因为他不要求加载的模块必须是由 `C/C++` 编写的，只要为一个正常的 `WebAssembly` 模块就能被调用，但调用的过程也更复杂。

接下来我会对这两种调用方法进行说明，使用的都是通过 `C/C++` 编写的模块。

## 使用`ccall()` 调用 

1. 先写一段`C++`代码如下，保存为`hello_world_ccall.cpp`。

``` cpp
#include <iostream>
#include <emscripten/emscripten.h>
const char* hello="hello,world!";
#ifdef __cplusplus
extern "C" {
#endif
const char* EMSCRIPTEN_KEEPALIVE myFunction(int argc, char ** argv) {
  std::cout << "walle: my function called" << std::endl;
  return hello;
}
#ifdef __cplusplus
}
#endif
```

对于导出的函数需要注意两点，一是需要使用 `EMSCRIPTEN_KEEPALIVE` 声明，他会将你的函数添加到导出函数列表中，其次最好使用 `extern"C"` 对接口进行导出，否则C++编译器会对函数名进行修改，导致在JS中使用的函数名跟这里定义的不同。

默认情况下，`Emscripten`生成的代码只会调用 `main` 函数，其它函数被视为无用代码，所以在函数名之前使用 `EMSCRIPTEN_KEEPALIVE` 可以防止这样的事情发生。

2. 接着运行以下命令编译：

``` shell
$ em++ -o hello_world_ccall.html hello_world_ccall.cpp -s WASM=1 -s "EXTRA_EXPORTED_RUNTIME_METHODS=['ccall']"
```

这里使用了 `em++` 命令，如果编译的是 `C` 代码，则使用 `emcc`。还需要注意我们添加了指定的参数 `EXTRA_EXPORTED_RUNTIME_METHODS` ，用于指定保留运行时函数。运行完该命令后，会生成有3个文件，分别是 `html`，`JS` 和 `wasm` 文件。

3. 对接口的调用，使用下面的 HTML，保存在同一目录下，并命名为 `index.html`。

``` html
html>
  <body>
    <button class="mybutton">Run my function</button>
    <canvas id="canvas_id">
      result
    </canvas>
  </body>

  <script type="text/javascript" src="hello_world_ccall.js"></script>
  <script>
  document.querySelector('.mybutton').addEventListener('click', function(){
    //Core code
    var result = Module.ccall('myFunction', // name of C function
    'string', // return type
    null, // argument types
    null); // arguments
    //Show result
    var canvas_box = document.getElementById('canvas_id').getContext('2d')
    canvas_box.font="48px serif";
    canvas_box.strokeText(result, 10, 50);
  });
  </script>

</html>
```

可以注意到其实我们在html中对生成的js文件进行了加载，然后调用了在其中定义好的 `Module.ccall` 函数，以实现对我们在 `C++` 中导出的函数的调用。

运行 `emrun --no_browser --port 8080 .`，然后通过浏览器访问 `http://localhost:8080` 然后按出现的按键， `canvas` 中就会显示 `hello,world!` 字符串了。

到这里 `ccall` 的使用方法就讲完了，相对比较简单。

## 通过对WebAssembly模块加载以运行

1. 首先还是先写一段 `C++` 代码，命名为 `hello_world.cpp`。

``` cpp
const char* hello="hello,world!";
#ifdef __cplusplus
extern "C" {
#endif
const char* myFunction() {
  return hello;
}
#ifdef __cplusplus
}
#endif
```

可以注意到，我把 `iostream` 和 `std::out` 调用去掉了，因为加入这两个会引入极大的复杂度，所以我们先从最简单的开始，同样使用 `extern "C"`。

2. 对代码进行编译

``` shell
$ em++ -O2 hello_world.cpp -s WASM=1 -s EXPORTED_FUNCTIONS='["_myFunction"]' -o hello_world.js
```

这里，我们使用 `EXPORTED_FUNCTIONS` 对我们想要的函数进行导出，注意导出的函数名需要加 `_` 前缀，不然编译会报错。

我们还需要对生成的 `wasm` 进行进一步的处理，因为如果直接使用现在生成的 `wasm`，在浏览器加载 `wasm` 的时候会一直报类似下面的错误：

``` js
index.html:1 Uncaught (in promise) LinkError: WebAssembly Instantiation: Import #0 module="env" function="___setErrNo" error: function import requires a callable
```

这是什么原因导致的呢，我们可以看下我们的 `wasm` 生成了哪些东西，这里需要使用一个 [wabt](https://github.com/WebAssembly/wabt) 库。把库下载完后编译配置(如何编译配置可见最后)，会得到几个有用的工具，我们这里用到的是 `wasm2wat` 工具，看名字就知道他是用于将 `wasm` 转换成 `wat`格式，后者是一种 [S-expressions](https://developer.mozilla.org/en-US/docs/WebAssembly/Understanding_the_text_format)，他和`wasm`的关系就类似于二进制和汇编的关系，我们很难从二进制文件看出什么有用的信息，但汇编就好读多了，所以使用命令:

``` shell
$ wasm2wat hello_world.wasm -o hello_world.wat
```

将 `hello_world.wasm` 反汇编成 `hello_world.wat`，查看后者，发现其包含了很多需要 `import` 的参数：

``` js
  (import "env" "___setErrNo" (func (;0;) (type 0)))
  (import "env" "_emscripten_get_heap_size" (func (;1;) (type 1)))
  (import "env" "_emscripten_memcpy_big" (func (;2;) (type 2)))
  (import "env" "_emscripten_resize_heap" (func (;3;) (type 3)))
  (import "env" "abortOnCannotGrowMemory" (func (;4;) (type 3)))
  (import "env" "DYNAMICTOP_PTR" (global (;0;) i32))
  (import "env" "memory" (memory (;0;) 256 256))
```

这里面有一些是我们根本就没有用到的，如 `___setErrNo` 函数，所以我们最好对他们做一次清理，所以又需要另一个工具库，叫做 [Binaryen](https://github.com/WebAssembly/binaryen.git)，同样的下载，编译和配置后（如何编译配置可见最后），我们又得到了几个工具，擦汗，我们需要用到他的 `wasm-opt` 工具对我们的 `wasm` 进行瘦身，具体使用命令：

``` shell
$ wasm-opt --remove-imports -Oz hello_world.wasm -o hello_world.wasm
```

从命令可以看出，我将他没用到的 `import` 移除掉了，并且使用了优化参数 `-Oz`，最后还是生成为 `hello_world.wasm`，我们再次通过 `wasm2wat` 查看下优化的效果：

``` js
  (import "env" "DYNAMICTOP_PTR" (global (;0;) i32))
  (import "env" "memory" (memory (;0;) 256 256))
```

会发现现在就只剩下两个 `import` 了，我们在JS调用的时候就方便多了。

3. 使用 js 对 `wasm` 进行加载和实例化，使用的 `html` 部分代码如下：

``` html
  <script>
  function fetchAndInstantiate(url,imports) {
    return fetch(url).then(response =>
      response.arrayBuffer()
    ).then(bytes =>
      WebAssembly.instantiate(bytes,imports)
    ).then(results =>
      results.instance
    );
  }

  var imports={
    env:{
      memory: new WebAssembly.Memory({ initial: 256, maximum:256 }),
      DYNAMICTOP_PTR: 0
    }
  }

  buffer=new Uint8Array(imports.env.memory.buffer);

  fetchAndInstantiate('hello_world.wasm',imports).then(function(instance) {
      document.querySelector('.mybutton').addEventListener('click', function(){
        var result = instance.exports._myFunction();
        let result_str="";
        for(let i = result; i < result+12; ++i)
            result_str+=String.fromCharCode(buffer[i])
        console.log(result_str)
        var canvas_box = document.getElementById('canvas_id').getContext('2d')
        canvas_box.font="48px serif";
        canvas_box.strokeText(result_str, 10+100*Math.random(), 50+100*Math.random());
      });
  })

```

可以注意到我们对 `wasm` 先 `fetch` 然后 `instantiate` 进行实例化，最后通过调用实例化出来的 `instance` 中导出的接口达到我们的目的：

``` js
var result = instance.exports._myFunction();
```

可能会觉得奇怪，我们返回的明明是一个字符串，为什么不能直接用，而是需要用 `buffer`做处理呢，原因是 `WebAssembly` 现在还非常弱，所以他的参数只能传 int 和 float 型的数据，返回值也是同样的，所以你猜下他返回的 `result` 是什么鬼东西，呃，是字符串的指针，好吧，这个指针有什么用呢，他指向了结果在 `memory.buffer`中存放的位置，所以我们需要把结果逐个的从 `buffer`中读出来。

还有需要说明的就是 `imports` 这个参数，他是用于 `initantiate` 实例化用的，他需要的参数就是我们前面在 `wat` 中看到的 `import` 语句，如果在加载 `wasm` 时看到终端有报我在上面提到的错误，那就是因为有哪些 `imports` 没有被加进去，你只需要加进去就行，所以我们的 `imports` 中就中加了 `DYNAMICTOP_PTR` 和 `memory` 。

总体感觉 `Webassembly` 的使用步骤比繁琐，且功能也不是很强大，除了第一种提到的 `ccall` 比较方便外，第二种官方推荐的做法有大量的坑，无论是传值，还是编译都会踩坑，只能期望他越做越好用吧。

demo下载地址：https://github.com/shmilycode/WebAssemblySample

------------------------
关于 `wabt` 和 `Binaryen` 的编译配置步骤是完全相同的，我们这里只讲 `wabt`:

1. 先 `git clone` 代码下来
2. 在代码路径下创建 `build` 文件夹，然后使用 `cmake`并编译安装

``` shell
$ mkdir build
$ cd build
$ cmake ..
$ make
$ sudo make install
```

完成。