# WASM

## 介绍

```md
WebAssembly/wasm WebAssembly 或者 wasm 是一个可移植、体积小、加载快并且兼容 Web 的全新格式。
```

wasm 是一种可在虚拟环境中执行的二进制文件格式，当前主要被用于解决 js 脚本在计算密集型应用中的执行效率问题。
wasm 格式文件目前支持在主流浏览器中执行，同时支持在 `WASI` 虚拟环境中执行。运行效率官方宣称接近原生。

## 原理

### 工作原理

wasm 实际上是通过 `LLVM` 将目标程序转换为可以在特定虚拟环境上运行的类汇编代码。理论上所有使用 `LLVM` 架构编译的语言都可以编译成 `wasm` 格式；同时， wasm 也有一个可读的编译格式： `wat` ，可以使用 [wasm2wat](https://webassembly.github.io/wabt/demo/wasm2wat/) 将 wasm 文件转换为 wat 文件。

```wat
(module
(type $t0 (func (param i32 i32) (result i32)))
(type $t1 (func (param i32 i32 i32) (result i32)))
(type $t2 (func (param i32 i32)))
(type $t3 (func (param i32)))
(type $t4 (func (param i32 i32 i32)))
(type $t5 (func (param i32) (result i32)))
(type $t6 (func))
(type $t7 (func (param i32) (result i64)))
(type $t8 (func (result i32)))
(type $t9 (func (param i64 i32) (result i32)))
(import "./wasm_game_of_life_bg.js" "__wbindgen_throw" (func $./wasm_game_of_life_bg.js.__wbindgen_throw (type $t2)))
(func $f1 (type $t5) (param $p0 i32) (result i32)
(local $l1 i32) (local $l2 i32) (local $l3 i32) (local $l4 i32) (local $l5 i32) (local $l6 i32) (local $l7 i32) (local $l8 i32) (local $l9 i64)
(block $B0
(block $B1
(block $B2
(block $B3
(block $B4
(if $I5
(i32.ge_u
(local.get $p0)
(i32.const 245))
(then
(br_if $B1
(i32.ge_u
(local.get $p0)
(i32.const -65587)))
...
```

上述是一个编译完成的 wasm 文件解析为 wat 的结果。从编译结果可以看出以下结论：

1. 与前端场景不同，后端场景中使用 wasm 后，程序将运行在一个非原生系统环境的运行时环境中，对于 `rust` 这种不含运行时的语言而言，效率必定存在下降；
2. wasm 编译后的程序只能接收有限类型的几个参数。这些类型包括:C 语言语义的 `i32` `i64` `f32` `f64` `v128`，包含除此之外的输入值、输出值的函数都不能被直接编译为 wasm 函数。在前端场景中，当前 `rust` 语言可以使用 `wasm_bindgen` 工具帮助我们自动将非标准函数转换为标准函数，同时这样也会引入运行时序列化消耗；除此之外，只能通过在 wasm 内直接操作连续内存的方式进行处理。在后端 rust 场景下就要求代码中加入大量 `unsafe` 代码，同时要求开发者手动管理内存，代码安全性会受到影响。

示例中通过 `wasm-pack` 生成的 js 和 ts 代码，主要负责将 js 语言中的字符串等进行编码，并解码 `wasm` 返回。在另一个示例 [生命游戏](https://github.com/rustwasm/wasm_game_of_life) 中， `wasm` 通过直接操作 js `canvas` 内存的方式进行数据交换，性能更强，但更不适用于后端场景。

### wasm-bindgen 场景（前端）

wasm 提供了供 rust 使用的 `wasm-bindgen` 工具，该工具会生成辅助代码，帮助前端场景调用 `wasm` 代码。

## WASI

`wasi` 是一种为了脱离浏览器实现的 `wasm` 解释器。 `wasi` 是一个跨平台的 runtime，可以支持在任意平台脱离浏览器运行 `wasm` 文件，并支持系统级操作（规划中）。`wasi` 仍在快速发展中，当前主流的 `wasi` 解释器包括：

- [WASI](https://github.com/WebAssembly/WASI)
- [wasmtime](https://github.com/bytecodealliance/wasmtime)
- [wasmer](https://github.com/wasmerio/wasmer)
- [wasm3](https://github.com/wasm3/wasm3)

目前，此类 `wasi` 的实现尚不完善，文档不完整，部分仓库示例 demo 无法运行，实际使用与 demo 场景差别很大，非标准类型场景下只能通过手动内存操作方式实现。

未来 `wasi` 官方表示，将扩充汇编指令，支持接口类型返回值，方便开发者调用。但是，实现后仍旧不能支持 rust 原生类型返回。

## 后端使用场景

当前后端使用 `wasm` 的主要场景是 Serverless 和 FaaS 场景。可以参考 [secondstate](https://www.secondstate.io/faas/) 公司的 FaaS 场景。此场景也是使用 `node.js` 实现。但是，此种方式的实现存在限制：

- 函数的返回值被限制在基础类型以及由基础类型组成的数组、字符串，其他类型不被支持；
- 函数只适用于计算密集型场景，每个函数执行有限的计算任务，在 i/o 密集场景反而会造成性能瓶颈。
- FaaS 适用于无服务器架构的 Serverless 场景

## 优势

- 对于前端开发者而言，wasm 提供了一种较为简便的性能优化方式，计算密集型场景下可以绕过 js jit 优化，更快的优化性能；同时可以让前端部分场景直接使用现有的 c 或者其他语言代码，使得大部分性能优化场景不需要使用 js 重复造轮子。
- wasm 程序虚拟化水平低，可以在虚拟环境下以接近原生的性能运行，同时兼具跨平台特性，不受实际硬件环境的影响。

## 劣势

- 注入值、返回值受限。根据 `wasm` 未来规划，将提供接口方式进行参数的返回，但是返回值仍旧被限制在一定范围内，必定存在格式转换问题。
- 后端场景下，wasm 相较于原生语言必然存在性能损耗。使用 rust 作为后端开发语言时，必须通过 rust 执行一个 wasm 虚拟机，在虚拟机内执行由 rust 转换的 wasm 代码。
- wasm 构建的程序本身存在限制，不能使用诸如端口、控制台等系统资源。此问题官方已经列入 todo 列表中，目前的解决方案只能是由调用方实现相关功能，提供调用接口给被调用的 wasm 程序使用。
- wasm 不支持多线程。官方将于后续提供多线程的支持。
- wasm 不支持文件读写，不支持系统资源控制，理论上只有带有 `#[no_std]` 的库可以无损编译为 wasm 格式。未来基于 `wasi` 的 wasm 可能支持相关功能。
- wasm 只能直接操作有限的数据类型，传递大变量或者不连续内存引用的时候会出现性能瓶颈；当前没有有效手段直接转换 rust 中的各种类型变量为 wasm 可支持的变量类型，只能手动转换；手动转换存在类型转换消耗，性能会进一步下降。
- rust 当前可以通过 `wasmer` 模拟执行 wasm 程序，但是没有对应的 `bindgen` 工具可以辅助使用；同时，直接用 `wasm-bindgen` 生成的 wasm 源码可以看到：

```wat
(import "./bindgen_bg.js" "__wbg_alert_c50838567b4e234b" (func $bindgen::alert::__wbg_alert_c50838567b4e234b::hb1923449dce4a4af (type $t5)))
```

这种外部注入严格依赖于 js 环境，仅适用于 js 调用场景，如果需要在 wasm 程序内调用外部 rust 函数会出现错误。

## 结论

`wasm` 当前主要适用于解决 js JIT 优化困难、快速提高计算密集型 js 应用性能的场景。后端场景中更适合在 Serverless 这种无服务器架构场景应用。

## 参考文献

1. <https://webassembly.github.io/wabt/demo/wasm2wat/>
2. <https://juejin.cn/user/3984285872430536>
3. <https://developer.mozilla.org/en-US/docs/WebAssembly/Rust_to_wasm>
4. <https://imweb.io/topic/5c06b8b0611a25cc7bf1d7d5>
5. <https://docs.wasmer.io/>
6. <https://rustwasm.github.io/docs/wasm-pack/introduction.html>
7. <http://llever.com/rustwasm-book/introduction.zh.html>
8. <https://zhuanlan.zhihu.com/p/297753460>
9. <https://www.secondstate.io/articles/getting-started-with-rust-function/>
