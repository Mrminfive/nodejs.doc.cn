<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [命令行选项](#%E5%91%BD%E4%BB%A4%E8%A1%8C%E9%80%89%E9%A1%B9)
  - [简介](#%E7%AE%80%E4%BB%8B)
  - [选项](#%E9%80%89%E9%A1%B9)
    - [`-v`, `--version`](#-v---version)
    - [`-h`, `--help`](#-h---help)
    - [`-e`, `--eval "script"`](#-e---eval-script)
    - [`-p`, `--print "script"`](#-p---print-script)
    - [`-c`, `--check`](#-c---check)
    - [`-i`, `--interactive`](#-i---interactive)
    - [`-r`, `--require module`](#-r---require-module)
    - [`--no-deprecation`](#--no-deprecation)
    - [`--trace-deprecation`](#--trace-deprecation)
    - [`--throw-deprecation`](#--throw-deprecation)
    - [`--no-warnings`](#--no-warnings)
    - [`--trace-warnings`](#--trace-warnings)
    - [`--trace-sync-io`](#--trace-sync-io)
    - [`zero-fill-buffers`](#zero-fill-buffers)
    - [`preserve-symlinks`](#preserve-symlinks)
    - [`--track-heap-objects`](#--track-heap-objects)
    - [`--prof-process`](#--prof-process)
    - [`--v8-options`](#--v8-options)
    - [`--tls-cipher-list=list`](#--tls-cipher-listlist)
    - [`--ebavke-fips`](#--ebavke-fips)
    - [`--force-fips`](#--force-fips)
    - [`--openssl-config=file`](#--openssl-configfile)
    - [`--icu-data-dir=file`](#--icu-data-dirfile)
  - [环境变量](#%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
    - [`NODE_DEBUG=module[,...]`](#node_debugmodule)
    - [`NODE_PATH=path[:...]`](#node_pathpath)
    - [`NODE_DISABLE_COLORS=1`](#node_disable_colors1)
    - [`NODE_ICU_DATA=file`](#node_icu_datafile)
    - [`NODE_PRESERVE_SYMLINKS=1`](#node_preserve_symlinks1)
    - [`NODE_REPL_HISTORY=file`](#node_repl_historyfile)
    - [`NODE_TIY_UNSAFE_ASYNC=1`](#node_tiy_unsafe_async1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# 命令行选项

Node.js带有各种CLI选项。这些选项提供了内置调试、多种执行脚本方式和其它有用的运行选项。

在终端中运行 `man node` 可以查看此手册。

## 简介

`node [options] [v8 options] [script.js | -e "scirpt"] [arguments]`

`node debug [script.js | -e "script" | <host>:<port>] ...`

`node --v8-options`

执行无参数启动 [REPL](#d)

## 选项

### `-v`, `--version`

打印 Node.js 的版本号。

### `-h`, `--help`

打印 Node.js 的命令行选项。当前文档内容比该输出文档详细

### `-e`, `--eval "script"`

执行参数代码，如 JavasSript。在 REPL 中预定义的模块也可以在 `script` 代码块中使用。

> 译者注：`node -e "var log = 'Hello World'; console.log(log)"`

### `-p`, `--print "script"`

与 `-e` 相同，但输出结果。

### `-c`, `--check`

在不执行的情况下进行脚本语法检查。

### `-i`, `--interactive`

TODO

### `-r`, `--require module`

在启动时预加载指定模块。

遵循 `require()` 的模块解析规则。`module` 可以是一个文件的路径，也可以时一个 Node.js 的模块名称。

### `--no-deprecation`

静默废弃警告

### `--trace-deprecation`

打印废弃的堆栈跟踪

### `--throw-deprecation`

抛出废弃的错误

### `--no-warnings`

静默所有进程警告（包括弃用）

### `--trace-warnings`

打印所有堆栈跟踪过程中的警告（包括弃用）

### `--trace-sync-io`

每当在事件循环的第一帧之后检测到同步 I/O 时，打印堆栈跟踪。

### `zero-fill-buffers`

> TODO

自动填充所有新分配的 [Buffer](#) 和 [SlowBuffer](#) 实例。

### `preserve-symlinks`

指定模块加载解析和缓存时保留符号链接。

默认情况下，当 Node.js 的加载模块从符号链接到不同的磁盘路径上的位置时，Node.js 将取消应用链接并使用该模块在磁盘上的“真实路径”，即一个标识符为以根路径做定位的依赖模块。在大多数情况下，这种默认行为是正常的在可接受范围内。但是，当使用存在同级依赖的符号链接时，如下面的例子所以，如果 moduleA 需要 moduleB做为同级依赖，则默认行为会导致抛出异常。

```text
{appDir}
 ├── app
 │   ├── index.js
 │   └── node_modules
 │       ├── moduleA -> {appDir}/moduleA
 │       └── moduleB
 │           ├── index.js
 │           └── package.json
 └── moduleA
     ├── index.js
     └── package.json
```

`--preserve-symlinks` 该命令行标识指定 Node.js 使用符号链接路径下的模块，而不是真实（当前）的路径，从而可以查找到与符号链接同级下的依赖模块。

注意，使用 `--preserve-symlinks` 可能会有其它副作用。具体来说，使用符号链接时，如果依赖树中的多个地方依赖 **本地** 模块，加载时将会抛出异常（Node.js会将它们看作两个单独的模块，并尝试多次加载该模块，从而导致抛出异常）。

### `--track-heap-objects`

跟踪为堆快照分配的堆对象

### `--prof-process`

使用 v8 选项 --prof 处理 v8 引擎生成的输出。

### `--v8-options`

打印 V8 命令行选项。

注：v8选项允许使用双破折号（-）或下划线（_）分开。

例如, `--stack-trace-limit` 就等同于 `--stack_trace_limit`。

### `--tls-cipher-list=list`

指定备用的默认 TLS 加密列表。（需要使用支持的加密构建 Node.js 。（默认））

### `--ebavke-fips`

启动时启用符合 FIPS 标准的加密。（需要使用 ./configure --openssl-fips 构建 Node.js。）

### `--force-fips`

强制使用 FIPS 标准的加密。（无法在脚本代码中被禁用）（与 `--enable-fips` 要求相同）

### `--openssl-config=file`

启动时加载OpenSSL的配置文件。如果 Node.js 使用 `./configure --openssl-fips` 构建，则该命令会启用符合 FIPS 的加密。

### `--icu-data-dir=file`

指定 ICU 数据的加载路径。（覆盖 NODE_ICU_DATA）


## 环境变量

### `NODE_DEBUG=module[,...]`

用 `,` 分隔的应该打印调试信息的核心模块列表


### `NODE_PATH=path[:...]`



### `NODE_DISABLE_COLORS=1`

设置为 `1` 时，在REPL中将不会使用颜色。

### `NODE_ICU_DATA=file`

ICU（Intl 对象）数据的数据路径。将在编译时使用 `small-icu` 支持的扩展链接的数据。

### `NODE_PRESERVE_SYMLINKS=1`

设置为 `1` 时，指示模块加载器解析和缓存模块时保留符号链接。

### `NODE_REPL_HISTORY=file`

用于存储持久性的 REPL 历史记录的文件的路径。它的默认路径是 ~/.node_repl_history，该变量覆盖该值。将值设置为空字符串（"" 或 " "）会禁用持久性的 REPL 历史记录。

### `NODE_TIY_UNSAFE_ASYNC=1`

设置为1时，在支持异步 `stdio` 的平台上输出到 TTY 时，写入 `stdout` 和 `stderr` 将是非阻塞和异步的。设置这将无法保证 `stdio` 不会在程序退出时发生交错或者丢失。 **不建议使用此模式**。