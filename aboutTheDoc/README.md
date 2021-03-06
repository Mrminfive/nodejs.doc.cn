<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [关于此文档](#%E5%85%B3%E4%BA%8E%E6%AD%A4%E6%96%87%E6%A1%A3)
- [稳定度](#%E7%A8%B3%E5%AE%9A%E5%BA%A6)
    - [稳定性指数如下：](#%E7%A8%B3%E5%AE%9A%E6%80%A7%E6%8C%87%E6%95%B0%E5%A6%82%E4%B8%8B)
- [JSON格式输出](#json%E6%A0%BC%E5%BC%8F%E8%BE%93%E5%87%BA)
- [系统调用和手册](#%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8%E5%92%8C%E6%89%8B%E5%86%8C)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

------

## 关于此文档

无论是从参考还是概念的角度来看，本文档的目的都是为了更全面的解释Node.js的API。每个部分都会介绍一个内置模块或者更高层次的概念。

属性类型、方法参数和提供给事件处理程序的参数在各种情况下的不同使用方式，在相应主题下的列表中都会详细说明。

每个 `.html` 文档都有一个相应的 `.json` 文档，以结构化的方式呈现相同的信息。这个特性是实验性的，希望能够为一些需要对文档进行程序化操作的 *IDE* 或者其他工具提供帮助。

每个 `.html` 和 `.json` 文件都是基于源代码 `doc/api/` 目录下的 `.md` 文件生成的。本文档使用 `tools/doc/generate.js` 这个程序生成。 HTML 模板位于 `doc/template.html`。

## 稳定度

在整个文档中，你可能会看到某个部分有稳定性提示。随着 Node.js 的逐步完善，它的API仍然会有一些小的变化，以确保某些部分更加可靠。一些接受过严格验证，被大量依赖的 API 不会轻易做改动。一些是新增和实验性的，或已知具有危险性将会被重新设计。

### 稳定性指数如下：

> 稳定性：**0** - 已弃用
这是一个存在问题的特性，目前正计划修改。*请不要使用该特性，*使用该功能可能会导致警告，不要指望该特性会向后兼容

> 稳定性：**1** - 实验性
此功能可能随时更改，我们会在命令行中给出相应提示。它可能会在将来的版本中更改或删除。

> 稳定性：**2** - 稳定
该API已被证明是令人满意的。在 npm 中已经有了很高的使用率，我们在没有绝对必要的情况下，不会对其做任何修改。

> 稳定性：**3** - 已锁定
我们只接受对其进行的安全性、性能或修复 BUG 的建议。请不要在修改 API 方面给出提案，因为我们会拒绝这类建议。


## JSON格式输出

> 稳定性：**1** - 实验性

每个通过 markdown 生成的 HTML 文件都对应于一个具有相同数据结构的 JSON 文件。

该特性是在 Node.js v0.6.12 中引入的，目前仍是实验性功能。


## 系统调用和手册

系统调用定义了用户程序和底层操作系统之间的接口，例如 [open(2)](http://man7.org/linux/man-pages/man2/open.2.html) 和 [read(2)](http://man7.org/linux/man-pages/man2/read.2.html) 。Node.js 的函数只是简单的包装了系统调用，就像文档中的 fs.open()。该文档链接到相应的手册页（以下简称手册页），其中描述了该系统调用的工作方式。

*警告：*一些系统调用，例如 [lchown(2)](http://man7.org/linux/man-pages/man2/lchown.2.html) ，是特定于 BSD 系统。这就意味着 fs.chown() 只适用于 Mac OS X 和其他的 BSD 派生系统，在 Linux 上是不可用的。

Windows 环境下的大多数系统回调和 Unix 环境下的等效，但有一些可能与 Linux 和 MAC OS X 不同。以一种微妙的关系为例，Windows 环境下有时不可能找到某些 Unix 系统回调的替代方案，详见 [Node issue 4760](https://github.com/nodejs/node/issues/4760) 。