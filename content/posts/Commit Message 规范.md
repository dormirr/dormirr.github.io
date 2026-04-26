---
title: 如何贡献有价值的反馈
tags:
  - 生活指南
  - Git
  - 博客
生活场景: Git
share: true
cover:
    image: ../../images/Commit Message 规范.webp
date: 2020-10-03T11:14:00
lastmod: 2026-04-26T13:31:00
categories: 生活指南
---

# 前言

Git 提交记录写的足够清晰，可以让人们清晰地看到你所更改的内容。

它是个很重要的东西，规范的提交记录可以让人们快速了解项目最近的改动。

下面介绍一下我个人使用的规范。

# 格式

```
<type>(<scope>): <subject>
<空行>
<body>
<空行>
<footer>
```

其中，`<type>(<scope>): <subject>` 为必填内容，其他为选填。

## type

表示本次的提交更改类型，有以下几个可选内容：

1. feat: 新增新功能；
2. fix: 修复问题；
3. docs: 只修改了文档；
4. style: 不影响代码含义的更改（空白，格式，缺少分号等）；
5. refactor: 既不修复 bug 也不添加功能的代码更改（重构）；
6. perf: 改进性能的代码更改；
7. test: 添加缺失的测试或纠正现有测试；
8. build: 影响构建系统或外部依赖的变化；
9. ci: 修改 Cl 配置文件和脚本；
10. chore: 其他不修改代码或测试文件的更改；
11. revert: 恢复一个以前的提交；

## scope

表示本次变更影响的范围。

如果描述不出范围可以用 `*` 表示。

## subject

简短描述本次提交的内容，不带句号。

如果是一次 revert 提交，则 subject 要和 revert 的 Header 内容一致。

## body

详细描述本次提交的内容。包括为什么修改以及和以前的对比。

如果是一次 revert 提交，则写：`回滚 <hash>。` `<hash>` 是被恢复提交的 SHA。

## footer

分为两种情况。

如果当前代码与上一个版本不兼容，则 footer 部分以 `BREAKING CHANGE` 开头，后面是对变动的描述、以及变动理由和迁移方法。

如果当前 commit 针对某些 issue，那么可以在 footer 部分关闭这些 issue 。

# 举例

```
feat($compile): simplify isolate scope bindings

Changed the isolate scope binding options to:
  - @attr - attribute binding (including interpolation)
  - =model - by-directional model binding
  - &expr - expression execution binding

This change simplifies the terminology as well as
number of choices available to the developer. It
also supports local name aliasing from the parent.

BREAKING CHANGE: isolate scope bindings definition has changed and
the inject option for the directive controller injection was removed.

To migrate the code follow the example below:

Before:

scope: {
  myAttr: 'attribute',
  myBind: 'bind',
  myExpression: 'expression',
  myEval: 'evaluate',
  myAccessor: 'accessor'
}

After:

scope: {
  myAttr: '@',
  myBind: '@',
  myExpression: '&',
  // myEval - usually not useful, but in cases where the expression is assignable, you can use '='
  myAccessor: '=' // in directive's template change myAccessor() to myAccessor
}

The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

# 最后

规范并非一成不变，最终目的就是生成简单明了的提交记录，让他人快速了解最近的项目变更内容。

我的规范仅供参考。每家大公司也有自己的规范，如 gitemoji 使用 emoji 表示提交内容等，各有各的优势，很有趣。我们可以取长补短，整理出自己认为最舒服的一套规范，一直遵守下去即可。
