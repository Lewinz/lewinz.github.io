---
layout: post
title: WebAssembly
categories: [webassembly]
description: WebAssembly
keywords: webassembly
---

自从引入计算机以来，本地应用程序的性能有了巨大的提高。相比之下，web 应用程序相当慢，因为 JS 一开始并不是为了速度而构建的。但是由于浏览器之间的激烈竞争以及 JS 引擎如 V8 的快速开发，使得 JS 能够在机器上快速运行。但是它仍然不能超过本机应用程序的性能。这主要是因为 JS 代码必须经历几个进程才能生成机器码。

![webassembly_1](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/webassembly_1.png)

随着 WebAssembly 的引入，现代 Web 发生革命性的变化，这项技术非常快。 让我们看一下什么是 WebAssembly，以及如何与 JS 集成以构建快速的应用程序。

## 什么是 WebAssembly？
在了解 WebAssembly 之前，让我们看一下什么是 Assembly。

Assembly (汇编) 是一种低级编程语言，它与体系结构的机器级指令有着非常密切的联系。换句话说，它只需一个进程就可以转换为机器可以理解的代码，即 机器代码。此转换过程称为汇编。

WebAssembly 可以简称为 Web 的汇编。 它是一种类似于汇编语言的低级语言，具有紧凑的二进制格式，使您能够以类似本机的速度运行 Web 应用程序。 它还为 C，C ++ 和 Rust 等语言提供了编译目标，从而使客户端应用程序能够以接近本地的性能在 Web 上运行。

此外，WebAssembly 的出现是与 JS 一起运行，而不是取代 JS。使用 WebAssembly JavaScript API，你可以交替地运行来自任一种语言的代码，来回没有任何问题。这为我们提供了利用 WebAssembly 的强大功能和性能以及 JS 的通用性和适应性的应用程序。这为 web 应用程序打开了一个全新的世界，它可以运行最初并不打算用于 web 的代码和功能。

## 有什么区别
Lin Clark 预测，2017 年 WebAssembly 的引入可能会引发 web 开发生命中的一个新的拐点。早期的另一个拐点 生在引入 JITs 编译的时候，JIT 编译使 JS 的速度提高了近 10 倍。

![webassembly_2](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/webassembly_2.png)

如果将 WebAssembly 的编译过程与 JS 的编译过程进行比较，会注意到几个过程已被剥离，其余过程已被修剪，如下所示：

JIT 是使 JavaScript 运行更快的一种手段，通过监视代码的运行状态，把 hot 代码（重复执行多次的代码）进行优化。通过这种方式，可以使 JavaScript 应用的性能提升很多倍。

![webassembly_3](https://cdn.jsdelivr.net/gh/Lewinz/lewinz.github.io@master/images/posts/webassembly_3.png)

仔细比较上图，注意到，重新参与 WebAssembly 已经完全被剥夺掉了。这主要是因为编译器不需要对 WebAssembly 代码做任何假设，因为诸如数据类型是在代码中明确提及。

但是 JS 不是这样的，因为 JIT 应该做一些假设来运行代码，如果假设失败，它需要重新优化它的代码。

## 如何获取 WebAssembly 代码
WebAssembly 是一项伟大的技术，我们需要如何利用 WebAssembly 的强大功能呢？

有几种方法：
- 不推荐从头编写 WebAssembly 代码，除非你非常了解基本知识
- 从 C 编译为 WebAssembly
- 从 C++ 编译为 WebAssembly
- 从 Rust 编译为 WebAssembly
- 使用 AssemblyScript 将 Typescript 编译为 WebAssembly。 对于不熟悉 C/C ++ 或 Rust 的 Web 开发人员来说，这是一个不错的选择
- 支持更多的语言选项。
此外，还有 Emscripten 和 WebAssembly Studio 之类的工具可以帮助您完成上述过程。

## JS 的 WebAssembly API
为了充分利用 WebAssembly 的特性，我们必须将其与 JS 代码集成在一起，这可以在 JavaScript WebAssembly API 的帮助下完成。

## 模块编译和实例化
WebAssembly 代 码驻留在.wasm 文件中。这个文件应该被编译成特定于它所运行的机器的机器码。我们可以使用 WebAssembly.compile 方法来编译 WebAssembly 模块。

WebAssembly.instantiate 方法实例化已编译模块。 另外，我们也可以从.wasm 文件获得的数组缓冲区传递到 WebAssembly.instantiate 方法中。 这也适用，因为实例化方法有两个重载。
``` js
let exports

fetch('sample.wasm').then(response =>
  response.arrayBuffer()
).then(bytes =>
  WebAssembly.instantiate(bytes)
).then(results => {
  exports = results.instance.exports
})
```
上述方法的缺点之一是这些方法不能直接访问字节码，因此在编译 / 实例化 wasm 模块之前，需要采取额外的步骤将响应转换为 ArrayBuffer。

相反，我们可以使用 WebAssembly.compileStreaming / WebAssembly.instantiateStreaming 方法来实现与上述相同的功能，其优点是可以直接访问字节码，而无需将响应转换为 ArrayBuffer。
``` js
let exports

WebAssembly.instantiateStreaming(fetch('sample.wasm'))
.then(obj => {
  exports = obj.instance.exports
})
```
注意，WebAssembly.instantiate 和 WebAssembly.instantiateStreaming 会返回实例以及已编译的模块，它们可用于快速启动模块的实例。
``` js
let exports;
let compiledModule;

WebAssembly.instantiateStreaming(fetch('sample.wasm'))
.then(obj => {
  exports = obj.instance.exports;
  //access compiled module
  compiledModule = obj.module;
})
```
## 导入对象
实例化 WebAssembly 模块实例时，可以选择传递一个导入对象，该对象将包含要导入到新创建的模块实例中的值，有 4 种类型：
- global values
- functions
- memory
- tables
可以将导入对象视为提供给模块实例的工具，以帮助它实现其任务。如果没有提供导入对象，编译器将分配默认值。

## Global
WebAssembly.Global 对象表示一个全局变量实例，可以被 JavaScript 和 importable/exportable 访问，跨越一个或多个 WebAssembly.Module 实例。他允许被多个 modules 动态连接.

可以使用 WebAssembly.Global() 构造函数创建全局实例。
``` js
const global = new WebAssembly.Global({
    value: 'i64',
    mutable: true
}, 20)
```
### 语法

var myGlobal = new WebAssembly.Global(descriptor, value)
global 构造函数接受两个参数。

### descriptor
GlobalDescriptor 包含 2 个属性的表:
- value: A USVString 表示全局变量的数据类型。可以是 i32, i64, f32, 或 f64
- mutable: 布尔值决定是否可以修改。默认是 false

### value
可以是任意变量值，需要其类型与变量类型匹配。如果变量没有定义，使用 0 代替
``` js
const global = new WebAssembly.Global({
    value: 'i64',
    mutable: true
}, 20);

let importObject = {
    js: {
        global
    }
};

WebAssembly.instantiateStreaming(fetch('global.wasm'), importObject)
```

全局实例应该传递给 importObject，以便在 WebAssembly 模块实例中可以访问它。

## Memory
当 WebAssembly 模块被实例化时，它需要一个 memory 对象。你可以创建一个新的 WebAssembly.Memory 并传递该对象。如果没有创建 memory 对象，在模块实例化的时候将会自动创建，并且传递给实例。

JS 引擎创建一个 ArrayBuffer 来做这件事情。ArrayBuffer 是 JS 引用的 JavaScript 对象。JS 为你分配内存。你告诉它需要多少内存，它会创建一个对应大小的 ArrayBuffer

ArrayBuffer 做了两件事情，一件是做 WebAssembly 的内存，另外一件是做 JavaScript 的对象。

它使 JS 和 WebAssembly 之间传递内容更方便。
使内存管理更安全。
## Table
WebAssembly.Table() 构造函数根据给定的大小和元素类型创建一个 Table 对象。

这是一个包装了 WebAssemble Table 的 Javascript 包装对象，具有类数组结构，存储了多个函数引用。在 JS 或者 WebAssemble 中创建 Table 对象可以同时被 JS 或 WebAssemble 访问和更改。

引入 Table 的主要原因是提高了安全性。我们可以使用 set()、grow() 和 get() 方法来操作表。

## 事例
为了演示，我将使用 WebAssembly Studio 应用程序将 C 文件编译为.wasm。

我已经在 wasm 文件中创建了一个函数来计算一个数字的幂。我将必要的值传递给函数，然后用 JavaScript 接收输出。

同样，我在 wasm 中进行了一些字符串操作。 需要注意，wasm 没有字符串类型。 因此，它将使用 ASCII 值。 返回到 JS 的值将指向存储输出的内存位置。 由于内存对象是 ArrayBuffer，因此我要进行迭代，直到收到字符串中的所有字符为止。

### JavaScript 文件
``` js
let exports;
let buffer;
(async() => {
  let response = await fetch('../out/main.wasm');
  let results = await WebAssembly.instantiate(await response.arrayBuffer());
  //or
  // let results = await WebAssembly.instantiateStreaming(fetch('../out/main.wasm'));
  let instance = results.instance;
  exports = instance.exports;
  buffer = new Uint8Array(exports.memory.buffer);

  findPower(5,3);
  
  printHelloWorld();
  
})();

const findPower = (base = 0, power = 0) => {
  console.log(exports.power(base,power));
}

const printHelloWorld = () => {
  let pointer = exports.helloWorld();
  let str = "";
  for(let i = pointer;buffer[i];i++){
    str += String.fromCharCode(buffer[i]);
  }
  console.log(str);
}
```
### C 文件
``` c
#define WASM_EXPORT __attribute__((visibility("default")))
#include <math.h>


WASM_EXPORT
double power(double number,double power_value) {
  return pow(number,power_value);
}

WASM_EXPORT
char* helloWorld(){
  return "hello world";
}
```
## 应用
WebAssembly 更适合用于写模块，承接各种复杂的计算，如图像处理、3D 运算、语音识别、视音频编码解码这种工作，主体程序还是要用 javascript 来写的。