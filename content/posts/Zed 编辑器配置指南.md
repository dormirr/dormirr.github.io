---
title: Zed 编辑器配置指南
tags:
  - 生活指南
  - 编辑器
  - 博客
  - Zed
生活场景: 编辑器
share: true
cover:
    image: ../../images/Zed 编辑器配置指南.webp
date: 2026-02-25T17:32:00
lastmod: 2026-08-31T01:05:00
categories: 生活指南
---

# 前言

如果你写代码的年头够长，大概率经历过几轮编辑器的迁徙。每一次切换都伴随着插件配置、快捷键记忆、工作流重构——折腾一圈下来，真正写代码的时间没剩多少，全花在调编辑器上了。

Zed 的出现，某种程度上就是在回应这种疲惫感。

它的团队来自 Atom，但这一次他们放弃了 Electron，用 Rust 从头写了一套 GPUI 框架。这意味着 Zed 不需要在功能丰富和响应流畅之间做取舍：你可以拥有接近 JetBrains 的语言智能（LSP 深度集成、多光标编辑、语法树感知），同时保持甚至超越 VS Code 的启动速度和滚动帧率。打开一个几千行的 Rust 文件，滚动依然跟手；Java 项目加载时，jdt.ls 在后台静默启动，你甚至不会感觉到编辑器界面被阻塞。

更值得留意的是它对 AI 的态度。Zed 没有像其他编辑器那样自建一套订阅制的 AI 服务，而是采用了 BYOK（Bring Your Own Key）模式。你可以在设置里填入 OpenAI、Anthropic 或任何兼容 OpenAI 协议的 API 密钥，甚至通过 Ollama 接入本地模型。想用 Claude 写代码，还是用 DeepSeek 做解释，全凭自己决定。只为实际调用的 tokens 付费，不为厂商的套餐买单。

当然，默认状态的 Zed 已经足够好用。但足够好用和真正顺手之间，往往隔着几十项细节调整。Tab 宽度是 2 还是 4？保存时自动格式化要不要开？语言服务器用哪个实现？这些偏好没有标准答案，只有你的习惯说了算。

这篇文章不打算教你入门 Zed。开箱即用的部分，打开编辑器就能体验到。我想做的是另一件事：拆解它的配置逻辑，然后给出一套可以直接拿走的配置方案，覆盖 Java 和 Rust 两个主力语言。复制进去，就能得到一个贴合日常习惯的开发环境，不用从零开始摸索。

工具理应用来忘记，而不是用来折腾。希望这篇配置指南，能帮你更快达到那个状态。

# 基本配置

Zed 版本：1.0.1

```json
{
  // 用于 UI 的 Zed 主题名称。
  //
  // `mode` 可为以下值之一：
  // - "system"：使用与系统外观对应的主题
  // - "light"：使用 "light" 字段指定的主题
  // - "dark"：使用 "dark" 字段指定的主题
  "theme": {
    "mode": "system",
    "light": "Vercel Light",
    "dark": "Vercel Dark",
  },
  "icon_theme": "Colored Zed Icons Theme Light",
  // 要使用的基础键位绑定方案名称。
  // 此设置可取六个值，每个值分别以另一款文本编辑器命名：
  //
  // 1. "VSCode"
  // 2. "Atom"
  // 3. "JetBrains"
  // 4. "None"
  // 5. "SublimeText"
  // 6. "TextMate"
  "base_keymap": "JetBrains",
  // 编辑器中用于渲染文本的字体名称。
  // 当前 ".ZedMono" 是 Lilex 的别名，
  // 但将来可能会改变。
  "buffer_font_family": "Maple Mono NF CN",
  // 编辑器中文本启用的 OpenType 特性。
  "buffer_font_features": {
    // 关闭连字：
    "calt": true,
  },
  // 用于 UI 文本渲染的字体名称。
  // 你可以将其设置为 ".SystemUIFont" 以使用系统字体。
  // 当前 ".ZedSans" 是 "IBM Plex Sans" 的别名，但将来可能会改变。
  "ui_font_family": ".SystemUIFont",
  // UI 文本启用的 OpenType 特性。
  "ui_font_features": {
    // 关闭连字：
    "calt": true,
  },
  // 是否在编辑器中用彩色标记括号。
  // （也称为"彩虹括号"）
  //
  // 不同缩进级别使用的颜色由主题（主题键：`accents`）定义。
  // 可通过主题覆盖进行自定义。
  "colorize_brackets": true,
  // 当光标在括号内时，是否在编辑器中显示方法签名。
  "auto_signature_help": true,
  // 是否在补全或插入括号对后显示签名帮助。
  // 如果启用了 `auto_signature_help`，此设置也将被视为启用。
  "show_signature_help_after_edits": true,
  "indent_guides": {
    // 决定缩进辅助线的着色方式。
    // 此设置可取以下三个值：
    //
    // 1. "disabled"
    // 2. "fixed"
    // 3. "indent_aware"
    "coloring": "indent_aware",
  },
  // 启用时，根据查询内容自动调整搜索的大小写敏感性。
  // 如果搜索查询包含任何大写字母，搜索将区分大小写；
  // 如果仅包含小写字母，搜索将不区分大小写。
  "use_smartcase_search": true,
  // 内联提示相关设置。
  "inlay_hints": {
    // 全局开关，用于打开或关闭提示，默认关闭。
    "enabled": true,
  },
  "collaboration_panel": {
    // 是否在状态栏显示协作面板按钮。
    "button": false,
  },
  "git_panel": {
    // 是否在 Git 面板中显示文件图标。
    //
    // 默认值：false
    "file_icons": true,
    /// 在面板中是以树状视图还是平铺视图显示条目。
    ///
    /// 默认值：false
    "tree_view": true,
  },
  "agent": {
    "default_profile": "minimal",
    // 创建新线程时使用的默认模型。
    "default_model": {
      // 要使用的提供商。
      "provider": "deepseek",
      // 要使用的模型。
      "model": "deepseek-reasoner",
      // 是否启用思考。
      "enable_thinking": true,
    },
    "favorite_models": [],
    "model_parameters": [],
  },
  // 控制语言服务器的语义标记如何用于语法高亮。
  //
  // 选项：
  // - "off"：不向语言服务器请求语义标记。
  // - "combined"：将 LSP 语义标记与 tree-sitter 高亮一起作为基础使用。
  // - "full"：仅使用 LSP 语义标记对文本进行高亮，关闭 tree-sitter 语法高亮。
  //
  // 可能需要重启语言服务器才能正确应用。
  "semantic_tokens": "combined",

  // 控制是否使用语言服务器的折叠范围代替 tree-sitter 和基于缩进的折叠。
  //
  // 选项：
  // - "off"：使用 tree-sitter 和基于缩进的折叠（默认）。
  // - "on"：尽可能使用 LSP 折叠，当服务器未返回结果时回退至 tree-sitter 和基于缩进的折叠。
  "document_folding_ranges": "on",

  // 控制用于大纲和面包屑导航的文档符号来源。
  //
  // 选项：
  // - "off"：使用 tree-sitter 查询计算文档符号（默认）。
  // - "on"：使用语言服务器的 `textDocument/documentSymbol` LSP 响应。启用后，不再使用 tree-sitter 获取文档符号。
  "document_symbols": "on",

  // 何时自动保存已编辑的缓冲区。此设置可取四个值。
  //
  // 1. 从不自动保存：
  //     "autosave": "off",
  // 2. 当焦点从 Zed 窗口移开时保存：
  //     "autosave": "on_window_change",
  // 3. 当焦点从特定缓冲区移开时保存：
  //     "autosave": "on_focus_change",
  // 4. 当空闲一定时间后保存：
  //     "autosave": { "after_delay": {"milliseconds": 500} },
  "autosave": "on_focus_change",
  // 如何软换行长文本。
  // 可能的值：
  //
  // 1. 通常首选单行，除非遇到非常长的行。
  //      "soft_wrap": "none",
  //      "soft_wrap": "prefer_line", // （已弃用，与 "none" 相同）
  // 2. 软换行超出编辑器宽度的行。
  //      "soft_wrap": "editor_width",
  // 3. 在首选行长度或编辑器宽度中较小者处软换行。
  //      "soft_wrap": "bounded",
  "soft_wrap": "bounded",
  // 对于启用了软换行的缓冲区，软换行的列位置。
  "preferred_line_length": 120,
  // 控制 Zed 收集哪些信息。
  "telemetry": {
    // 发送如崩溃报告等调试信息。
    "diagnostics": false,
    // 发送匿名使用数据，如你正使用 Zed 与哪些语言。
    "metrics": false,
  },
  // 诊断配置。
  "diagnostics": {
    // 内联诊断设置。
    "inline": {
      // 是否内联显示诊断信息。
      "enabled": true,
    },
  },
  "session": {
    // 是否跳过工作树信任检查。
    // 当受信任时，项目设置会自动同步，
    // 语言和 MCP 服务器会自动下载和启动。
    //
    // 默认值：false
    "trust_all_worktrees": true,
  },
}
```
