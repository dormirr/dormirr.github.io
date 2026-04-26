---
title: Zed 编辑器配置指南
tags:
  - 生活指南
  - 编辑器
  - 博客
生活场景: 编辑器
share: true
cover:
    image: ../../images/搜索引擎使用教程封面.webp
date: 2026-02-25T17:32:00
lastmod: 2026-02-25T18:51:00
categories: 生活指南
---

# 前言

如果你写代码的年头够长，大概率经历过几轮编辑器的迁徙。每一次切换都伴随着插件配置、快捷键记忆、工作流重构，折腾一圈下来，真正写代码的时间没多少，调编辑器的时间倒是占了大半。

Zed 的出现，某种程度上是在回应这种疲惫感。

它的团队来自 Atom，但这一次他们放弃了 Electron，用 Rust 从头写了一套 GPUI 框架。这意味着 Zed 不需要在功能丰富和响应流畅之间做取舍——你可以拥有接近 JetBrains 的语言智能，包括 LSP 深度集成、多光标编辑、语法树感知，同时保持甚至超越 VS Code 的启动速度和滚动帧率。打开一个几千行的 Rust 文件，滚动依然跟手；Java 项目加载时，jdt.ls 在后台静默启动，你不会感觉到编辑器界面被阻塞。

更值得留意的是它对 AI 的态度。Zed 没有像其他编辑器那样自建一套订阅制的 AI 服务，而是采用 BYOK（Bring Your Own Key）模式。你可以在设置里填入 OpenAI、Anthropic 或任何兼容 OpenAI 协议的 API 密钥，甚至通过 Ollama 接入本地模型。想用 Claude 写代码，还是用 DeepSeek 做解释，自己说了算——你只为实际调用的 tokens 付费，不为厂商的套餐买单。

当然，默认状态的 Zed 已经足够好用，但足够好用和真正顺手之间，往往隔着几十项细节调整。Tab 宽度是 2 还是 4？保存时自动格式化要不要开？语言服务器用哪个实现？这些偏好没有标准答案，只有你自己的习惯说了算。

这篇文章不打算教你怎么入门 Zed，开箱即用的部分你打开编辑器就能体验到。我想做的是另一件事：拆解它的配置逻辑，然后给出一套可以直接拿去用的配置方案，覆盖 Java 和 Rust 两个主力语言。你复制进去，就能得到一个贴合自己习惯的环境，不用从零开始摸索。

工具理应用来忘记，而不是用来折腾。希望这篇基于 Zed 0.224.11 的配置指南能帮你更快达到那个状态。

# 开箱即用配置方案

Zed 版本：0.224.11

```json
// Zed settings
//
// For information on how to configure Zed, see the Zed
// documentation: https://zed.dev/docs/configuring-zed
//
// To see all of Zed's default settings without changing your
// custom settings, run `zed: open default settings` from the
// command palette (cmd-shift-p / ctrl-shift-p)
{
  "session": {
    // 是否跳过工作树信任检查。
    // 当受信任时，项目设置会自动同步，
    // 语言和 MCP 服务器会自动下载并启动。
    //
    // 默认值：false
    "trust_all_worktrees": true,
  },

  // 用于 UI 的 Zed 主题名称。
  //
  // `mode` 可以是以下之一：
  // - "system": 使用与系统外观对应的主题
  // - "light": 使用 "light" 字段指定的主题
  // - "dark": 使用 "dark" 字段指定的主题
  "theme": {
    "mode": "system",
    "light": "One Light",
    "dark": "One Dark",
  },
  "icon_theme": "Zed (Default)",

  // 要使用的基本键绑定集名称。
  // 此设置可取六个值，每个值以另一个文本编辑器命名：
  //
  // 1. "VSCode"
  // 2. "Atom"
  // 3. "JetBrains"
  // 4. "None"
  // 5. "SublimeText"
  // 6. "TextMate"
  "base_keymap": "JetBrains",

  // 用于在编辑器中呈现文本的字体名称
  // ".ZedMono" 当前别名指向 Lilex
  // 但将来可能会更改。
  "buffer_font_family": "Cascadia Code",
  // 设置缓冲区文本的字体后备列表，这将与平台的默认后备列表合并。
  "buffer_font_fallbacks": ["霞鹜文楷等宽"],
  // 为编辑器中的文本启用的 OpenType 特性。
  "buffer_font_features": {
    // 启用连字：
    "calt": true,
  },

  // 是否在编辑器中为括号着色。
  // （也称为“彩虹括号”）
  //
  // 用于不同缩进级别的颜色在主题中定义（主题键：`accents`）。
  // 可以通过主题覆盖进行自定义。
  "colorize_brackets": true,

  // 何时在补全菜单中显示滚动条。
  // 此设置可取四个值：
  //
  // 1. 如果有重要信息则显示滚动条，或遵循系统的配置行为
  //   "auto"
  // 2. 匹配系统的配置行为：
  //    "system"
  // 3. 始终显示滚动条：
  //    "always"
  // 4. 从不显示滚动条：
  //    "never"（默认）
  "completion_menu_scrollbar": "auto",

  // 在编辑器中，当位于括号内时显示方法签名。
  "auto_signature_help": true,
  // 补全后或插入括号对后是否显示签名帮助。
  // 如果启用了 `auto_signature_help`，此设置也将被视为启用。
  "show_signature_help_after_edits": true,

  // 标题栏相关设置
  "title_bar": {
    // 是否在标题栏的分支切换器旁边显示分支图标。
    "show_branch_icon": true,
    // 是否在标题栏中显示分支名称按钮。
    "show_branch_name": true,
    // 是否在标题栏中显示项目主机和名称。
    "show_project_items": true,
    // 是否在标题栏中显示入门指南横幅。
    "show_onboarding_banner": true,
    // 是否在标题栏中显示用户头像。
    "show_user_picture": true,
    // 是否在标题栏中显示用户菜单。
    "show_user_menu": true,
    // 是否在标题栏中显示登录按钮。
    "show_sign_in": true,
    // 是否在标题栏中显示菜单。
    "show_menus": false,
  },

  "inlay_hints": {
    // 全局开关，用于切换提示的开启和关闭，默认关闭。
    "enabled": true,
    // 切换某些类型提示的开启和关闭，默认全部开启。
    "show_type_hints": true,
    "show_parameter_hints": true,
    "show_value_hints": true,
    // 对应于 null/None 的 LSP 提示类型值。
    "show_other_hints": true,
    // 是否为内嵌提示显示背景。
    //
    // 如果设置为 `true`，背景将使用当前主题中的 `hint.background` 颜色。
    "show_background": false,
    // 编辑缓冲区后等待请求提示的时间，设置为 0 以禁用去抖。
    "edit_debounce_ms": 700,
    // 滚动缓冲区后等待请求提示的时间，设置为 0 以禁用去抖。
    "scroll_debounce_ms": 50,
    // 一组修饰键，当按下时会切换内嵌提示的可见性。
    // 如果该集合为空或未按下所有指定的修饰键，则不会切换内嵌提示。
    "toggle_on_modifiers_press": {
      "control": false,
      "shift": false,
      "alt": false,
      "platform": false,
      "function": false,
    },
  },

  // 控制如何将语言服务器的语义标记用于语法高亮显示。
  //
  // 选项：
  // - "off": 不从语言服务器请求语义标记。
  // - "combined": 将 LSP 语义标记与 tree-sitter 高亮显示一起用作基础。
  // - "full": 仅使用 LSP 语义标记来高亮显示文本，tree-sitter 语法高亮显示关闭。
  //
  // 可能需要重启语言服务器才能正确应用。
  "semantic_tokens": "combined",

  // 控制是否使用来自语言服务器的折叠范围而不是 tree-sitter 和基于缩进的折叠。
  //
  // 选项：
  // - "off": 使用 tree-sitter 和基于缩进的折叠（默认）。
  // - "on": 尽可能使用 LSP 折叠，当服务器未返回结果时回退到 tree-sitter 和基于缩进的折叠。
  "document_folding_ranges": "on",

  // 何时自动保存编辑的缓冲区。此设置可取四个值。
  //
  // 1. 从不自动保存：
  //     "autosave": "off",
  // 2. 当焦点从 Zed 窗口移开时保存：
  //     "autosave": "on_window_change",
  // 3. 当焦点从特定缓冲区移开时保存：
  //     "autosave": "on_focus_change",
  // 4. 空闲一段时间后保存：
  //     "autosave": { "after_delay": {"milliseconds": 500} },
  "autosave": "on_window_change",

  // 如何软换行长行文本。
  // 可能的值：
  //
  // 1. 通常首选单行，除非遇到过长的行。
  //      "soft_wrap": "none",
  //      "soft_wrap": "prefer_line", // （已弃用，与 "none" 相同）
  // 2. 对超出编辑器宽度的行进行软换行。
  //      "soft_wrap": "editor_width",
  // 3. 在首选行长度处软换行。
  //      "soft_wrap": "preferred_line_length",
  // 4. 在首选行长度或编辑器宽度（取较小者）处软换行。
  //      "soft_wrap": "bounded",
  "soft_wrap": "editor_width",

  // 控制 Zed 收集哪些信息。
  "telemetry": {
    // 发送调试信息，如崩溃报告。
    "diagnostics": false,
    // 发送匿名使用数据，如您使用 Zed 的语言。
    "metrics": false,
  },
}
```

# 系统默认配置解析

```json
{
  "$schema": "zed://schemas/settings",
  /// 此项目的显示名称。如果未设置或为 null，则显示根目录名称。
  "project_name": null,
  // 用于 UI 的 Zed 主题名称。
  //
  // `mode` 可以是以下之一：
  // - "system": 使用与系统外观对应的主题
  // - "light": 使用 "light" 字段指定的主题
  // - "dark": 使用 "dark" 字段指定的主题
  "theme": {
    "mode": "system",
    "light": "One Light",
    "dark": "One Dark",
  },
  "icon_theme": "Zed (Default)",
  // 要使用的基本键绑定集名称。
  // 此设置可取六个值，每个值以另一个文本编辑器命名：
  //
  // 1. "VSCode"
  // 2. "Atom"
  // 3. "JetBrains"
  // 4. "None"
  // 5. "SublimeText"
  // 6. "TextMate"
  "base_keymap": "VSCode",
  // 用于在编辑器中呈现文本的字体名称
  // ".ZedMono" 当前别名指向 Lilex
  // 但将来可能会更改。
  "buffer_font_family": ".ZedMono",
  // 设置缓冲区文本的字体后备列表，这将与平台的默认后备列表合并。
  "buffer_font_fallbacks": null,
  // 为编辑器中的文本启用的 OpenType 特性。
  "buffer_font_features": {
    // 禁用连字：
    // "calt": false
  },
  // 编辑器中文本的默认字体大小
  "buffer_font_size": 15,
  // 编辑器字体的粗细，采用标准 CSS 单位，范围从 100 到 900。
  "buffer_font_weight": 400,
  // 设置缓冲区的行高。
  // 可取三个值：
  //  1. 使用适合阅读的行高 (1.618)
  //         "buffer_line_height": "comfortable"
  //  2. 使用标准行高 (1.3)
  //         "buffer_line_height": "standard",
  //  3. 使用自定义行高
  //         "buffer_line_height": {
  //           "custom": 2
  //         },
  "buffer_line_height": "comfortable",
  // 用于在 UI 中呈现文本的字体名称
  // 您可以将其设置为 ".SystemUIFont" 以使用系统字体
  // ".ZedSans" 当前别名指向 "IBM Plex Sans"，但将来可能会更改
  "ui_font_family": ".ZedSans",
  // 设置 UI 的字体后备列表，这将与平台的默认字体后备列表合并。
  "ui_font_fallbacks": null,
  // 为 UI 中的文本启用的 OpenType 特性
  "ui_font_features": {
    // 禁用连字：
    "calt": false,
  },
  // UI 字体的粗细，采用标准 CSS 单位，范围从 100 到 900。
  "ui_font_weight": 400,
  // UI 中文本的默认字体大小
  "ui_font_size": 16,
  // 代理面板中代理响应的默认字体大小。如果未设置，则回退到 UI 字体大小。
  "agent_ui_font_size": null,
  // 代理面板中用户消息的默认字体大小。
  "agent_buffer_font_size": 12,
  // 未使用代码的淡出程度。
  "unnecessary_code_fade": 0.3,
  // 活动窗格样式设置。
  "active_pane_modifiers": {
    // 活动窗格的嵌入边框大小（像素）。
    "border_size": 0.0,
    // 非活动窗格的不透明度。0 表示透明，1 表示不透明。
    // 值将被限制在 [0.0, 1.0] 范围内。
    "inactive_opacity": 1.0,
  },
  // 底部停靠栏的布局模式。默认为 "contained"
  //   选项：contained, full, left_aligned, right_aligned
  "bottom_dock_layout": "contained",
  // 水平拆分窗格的方向。默认为 "down"
  "pane_split_direction_horizontal": "down",
  // 垂直拆分窗格的方向。默认为 "right"
  "pane_split_direction_vertical": "right",
  // 居中布局相关设置。
  "centered_layout": {
    // 使用居中布局时，工作区中央窗格左侧填充的相对宽度。
    "left_padding": 0.2,
    // 使用居中布局时，工作区中央窗格右侧填充的相对宽度。
    "right_padding": 0.2,
  },
  // 图片查看器设置
  "image_viewer": {
    // 图片文件大小的单位："binary"（KiB, MiB）或 decimal（KB, MB）
    "unit": "binary",
  },
  // 确定使用鼠标添加多个光标时要使用的修饰键。打开悬停链接的鼠标手势将进行调整，使其不与多光标修饰键冲突。
  //
  // 1. 在 Linux 和 Windows 上映射为 `Alt`，在 macOS 上映射为 `Option`：
  //    "alt"
  // 2. 在 Linux 和 Windows 上映射为 `Control`，在 macOS 上映射为 `Command`：
  //    "cmd_or_ctrl" (别名："cmd", "ctrl")
  "multi_cursor_modifier": "alt",
  // 是否启用 vim 模式和键绑定。
  "vim_mode": false,
  // 是否启用 helix 模式和键绑定。
  // 启用此模式将自动启用 vim 模式。
  "helix_mode": false,
  // 当鼠标悬停在编辑器中的符号上时，是否显示信息性悬停框。
  "hover_popover_enabled": true,
  // 显示信息性悬停框前的等待时间（毫秒）。
  // 当启用 `auto_signature_help` 时，此延迟也适用于自动签名帮助。
  "hover_popover_delay": 300,
  // 退出 Zed 前是否确认。
  "confirm_quit": false,
  // 当打开新的 Zed 实例时，是否恢复上次关闭的项目。
  // 可取三个值：
  //  1. 上次会话中打开的所有工作区
  //         "restore_on_startup": "last_session"
  //  2. 上次打开的工作区
  //         "restore_on_startup": "last_workspace",
  //  3. 不恢复之前的工作区
  //         "restore_on_startup": "none",
  "restore_on_startup": "last_session",
  // 是否在再次打开文件时尝试恢复之前文件的状态。
  // 状态按窗格存储。
  // 禁用时，将应用默认值而不是恢复的状态。
  //
  // 例如，对于编辑器，如果同一个文件被关闭，稍后在同一个窗格中再次打开，将恢复选择、折叠和滚动位置。
  // 禁用时，默认使用文件开头的一个选择、零滚动位置和无折叠状态。
  //
  // 默认值：true
  "restore_on_file_reopen": true,
  // 是否自动关闭已在磁盘上删除的文件。
  "close_on_file_delete": false,
  // 编辑器中放置目标的大小相对值，该目标将放置的文件作为拆分窗格打开（0-0.5）
  // 例如：0.25 == 如果您拖放到窗格的上/下四分之一处，将使用新的垂直拆分
  //              如果您拖放到窗格的左/右四分之一处，将使用新的水平拆分
  "drop_target_size": 0.2,
  // 当使用“关闭活动项”关闭没有标签页的窗口时，是否应关闭窗口。
  // 可取三个值：
  //  1. 使用当前平台的惯例
  //         "when_closing_with_no_tabs": "platform_default"
  //  2. 始终关闭窗口：
  //         "when_closing_with_no_tabs": "close_window",
  //  3. 从不关闭窗口
  //         "when_closing_with_no_tabs": "keep_window_open",
  "when_closing_with_no_tabs": "platform_default",
  // 当最后一个窗口关闭时要执行的操作。
  // 可取两个值：
  //  1. 使用当前平台的惯例
  //         "on_last_window_closed": "platform_default"
  //  2. 始终退出应用程序
  //         "on_last_window_closed": "quit_app",
  "on_last_window_closed": "platform_default",
  // 要使用的文本渲染模式。
  // 可取三个值：
  //  1. 使用平台默认行为：
  //         "text_rendering_mode": "platform_default"
  //  2. 使用子像素（ClearType风格）文本渲染：
  //         "text_rendering_mode": "subpixel"
  //  3. 使用灰度文本渲染：
  //         "text_rendering_mode": "grayscale"
  "text_rendering_mode": "platform_default",
  // 是否显示缩放面板的填充。
  // 启用时，缩放的中央面板（例如代码编辑器）将在四周有填充，
  // 而缩放的底部/左侧/右侧面板将分别在顶部/右侧/左侧有填充。
  //
  // 默认值：true
  "zoomed_padding": true,
  // 绘制 Zed 窗口装饰（标题栏）的方式：
  // 1. 客户端应用程序（Zed）绘制自己的窗口装饰
  //    "client"
  // 2. 显示服务器绘制窗口装饰。GNOME Wayland 不支持。
  //    "server"
  //
  // 更改此设置需要重启 Zed 才能生效。
  //
  // 默认值："client"
  "window_decorations": "client",
  // 是否使用系统提供的打开和另存为对话框。
  // 设置为 false 时，Zed 将使用内置的键盘优先选择器。
  "use_system_path_prompts": true,
  // 是否使用系统提供的提示对话框，例如确认提示。
  // 设置为 false 时，Zed 将使用其内置提示。请注意，在 Linux 上，
  // 此选项被忽略，Zed 将始终使用内置提示。
  "use_system_prompts": true,
  // 编辑器中的光标是否闪烁。
  "cursor_blink": true,
  // 默认编辑器的光标形状。
  //  1. 垂直条
  //     "bar"
  //  2. 包围后续字符的块
  //     "block"
  //  3. 沿后续字符的下划线
  //     "underline"
  //  4. 包围后续字符的框
  //     "hollow"
  //
  // 默认值："bar"
  "cursor_shape": "bar",
  // 确定何时在编辑器或输入框中隐藏鼠标光标。
  //
  // 1. 从不隐藏鼠标光标：
  //    "never"
  // 2. 仅在键入时隐藏：
  //    "on_typing"
  // 3. 在键入和光标移动时隐藏：
  //    "on_typing_and_movement"
  "hide_mouse": "on_typing_and_movement",
  // 确定代码片段相对于其他补全项的排序方式。
  //
  // 1. 将代码片段放在补全列表顶部：
  //    "top"
  // 2. 正常放置代码片段，无特殊偏好：
  //    "inline"
  // 3. 将代码片段放在补全列表底部：
  //    "bottom"
  // 4. 不在补全列表中显示代码片段：
  //    "none"
  "snippet_sort_order": "inline",
  // 如何在编辑器中突出显示当前行。
  //
  // 1. 不突出显示当前行：
  //    "none"
  // 2. 突出显示装订线区域：
  //    "gutter"
  // 3. 突出显示编辑器区域：
  //    "line"
  // 4. 突出显示整行（默认）：
  //    "all"
  "current_line_highlight": "all",
  // 是否在编辑器中突出显示所选文本的所有出现位置。
  "selection_highlight": true,
  // 文本选择是否应有圆角。
  "rounded_selection": true,
  // 根据当前光标位置从语言服务器查询高亮显示的去抖延迟。
  "lsp_highlight_debounce": 75,
  // 前景色和背景色之间的最小 APCA 感知对比度。
  // APCA（可访问感知对比度算法）比 WCAG 2.x 更准确，尤其是在暗色模式下。值范围从 0 到 106。
  //
  // 基于 APCA 可读性标准（ARC）青铜简单模式：
  // https://readtech.org/ARC/tests/bronze-simple-mode/
  // - 0：无对比度调整
  // - 45：大型流畅文本的最小值（36px+）
  // - 60：其他内容文本的最小值
  // - 75：正文文本的最小值
  // - 90：正文文本的首选值
  //
  // 这仅影响在编辑器高亮背景上绘制的文本。
  "minimum_contrast_for_highlights": 45,
  // 在编辑器中键入时，是否自动弹出补全菜单，无需显式请求。
  "show_completions_on_input": true,
  // 是否在补全菜单中显示内联和并排的文档信息。
  "show_completion_documentation": true,
  // 是否在编辑器中为括号着色。
  // （也称为“彩虹括号”）
  //
  // 用于不同缩进级别的颜色在主题中定义（主题键：`accents`）。
  // 可以通过主题覆盖进行自定义。
  "colorize_brackets": false,
  // 何时在补全菜单中显示滚动条。
  // 此设置可取四个值：
  //
  // 1. 如果有重要信息则显示滚动条，或遵循系统的配置行为
  //   "auto"
  // 2. 匹配系统的配置行为：
  //    "system"
  // 3. 始终显示滚动条：
  //    "always"
  // 4. 从不显示滚动条：
  //    "never"（默认）
  "completion_menu_scrollbar": "never",
  // 是否将代码补全上下文菜单中的详细信息文本向左或向右对齐。
  "completion_detail_alignment": "left",
  // 如何在编辑器中显示差异。
  //
  // 默认值：split
  "diff_view_style": "split",
  // 在编辑器中，当位于括号内时显示方法签名。
  "auto_signature_help": false,
  // 补全后或插入括号对后是否显示签名帮助。
  // 如果启用了 `auto_signature_help`，此设置也将被视为启用。
  "show_signature_help_after_edits": false,
  // 是否在缓冲区行开头显示代码操作按钮。
  "inline_code_actions": true,
  // 是否允许在缓冲区中拖放文本选择。
  "drag_and_drop_selection": {
    // 如果为 true，则允许在缓冲区中拖放文本选择。
    "enabled": true,
    // 允许拖放之前必须经过的延迟（毫秒）。否则，将创建新的文本选择。
    "delay": 300,
  },
  // 当转到定义没有结果时要执行的操作。
  //
  // 1. 不执行任何操作：`none`
  // 2. 查找同一符号的引用：`find_all_references`（默认）
  "go_to_definition_fallback": "find_all_references",
  // 用于过滤编辑器中显示的诊断信息的级别。
  //
  // 仅影响编辑器渲染，不会中断诊断获取和项目诊断编辑器的功能。
  // 要在标签页中标记哪些包含诊断错误/警告的文件。
  // 仅当文件图标也处于活动状态时，才会显示诊断信息。
  // 此设置仅适用于以下三个值：
  //
  // 要在滚动条中显示哪些诊断指示器，其级别应大于或等于指定的严重性级别。
  // 可能的值：
  //  - "off" — 不允许任何诊断
  //  - "error"
  //  - "warning"
  //  - "info"
  //  - "hint"
  //  - "all" — 允许所有诊断（默认）
  "diagnostics_max_severity": "all",
  // 是否在编辑器中显示换行参考线（垂直标尺）。
  // 如果 'soft_wrap' 设置为 'preferred_line_length'，则将此设置为 true 将在 'preferred_line_length' 值处显示一条参考线，并将按 'wrap_guides' 设置显示任何其他参考线。
  "show_wrap_guides": true,
  // 在编辑器中显示换行参考线的字符计数。
  "wrap_guides": [],
  // 在私有文件中隐藏变量的视觉显示值。
  "redact_private_values": false,
  // 在多缓冲区中展开摘录的默认行数。
  "expand_excerpt_lines": 5,
  // 在多缓冲区摘录中显示的默认上下文行数。
  "excerpt_context_lines": 2,
  // 与文件路径匹配的 glob，用于确定文件是否为私有。
  "private_files": [
    "**/.env*",
    "**/*.pem",
    "**/*.key",
    "**/*.cert",
    "**/*.crt",
    "**/secrets.yml",
  ],
  // 是否在每次“触发”符号输入后使用额外的 LSP 查询来格式化（并修改）代码，该触发符号由 LSP 服务器能力定义。
  "use_on_type_format": true,
  // 是否在键入开括号、方括号、大括号、单引号或双引号字符时自动添加匹配的关闭字符。
  // 例如，当您键入 '(' 时，Zed 将在正确位置添加一个关闭的 ')'。
  "use_autoclose": true,
  // 是否在键入开括号、方括号、大括号、单引号或双引号字符时自动包围所选文本。
  // 例如，当您选择文本并键入 '(' 时，Zed 将用 () 包围该文本。
  "use_auto_surround": true,
  // 是否在键入时根据上下文调整缩进。
  "auto_indent": true,
  // 是否根据上下文调整粘贴内容的缩进。
  "auto_indent_on_paste": true,
  // 控制编辑器如何处理自动关闭的字符。
  // 当设置为 `false`（默认）时，跳过和自动移除关闭字符仅适用于自动插入的字符。
  // 否则（当 `true` 时），无论关闭字符是如何插入的，始终会跳过和自动移除它们。
  "always_treat_brackets_as_autoclosed": false,
  // 控制在当前语言作用域中允许 `editor::Rewrap` 操作的位置。
  //
  // 此设置可取三个值：
  //
  // 1. 仅允许在注释中重新换行：
  //    "in_comments"
  // 2. 仅允许在当前选择中重新换行：
  //    "in_selections"
  // 3. 允许在任何位置重新换行：
  //    "anywhere"
  //
  // 当使用除 `in_comments` 之外的值时，重新换行可能会产生语法上无效的代码。选择要使用的行为时请记住这一点。
  //
  // 注意：此设置在 Vim 模式下无效，因为重新换行已允许在任何位置。
  "allow_rewrap": "in_comments",
  // 控制编辑预测是立即显示（true）还是通过触发 `editor::ShowEditPrediction` 手动显示（false）。
  "show_edit_predictions": true,
  // 控制在哪些语言作用域中禁用编辑预测。
  // 示例：["string", "comment"]
  "edit_predictions_disabled_in": [],
  // 是否在编辑器中显示制表符和空格。
  // 此设置可取四个值：
  //
  // 1. 仅为所选文本绘制制表符和空格（默认）：
  //    "selection"
  // 2. 不绘制任何制表符或空格：
  //    "none"
  // 3. 绘制所有不可见符号：
  //    "all"
  // 4. 仅在边界处绘制空白：
  //    "boundary"
  // 5. 仅绘制非空白字符后的空白：
  //    "trailing"
  // 空白被视为在边界处，需满足以下任一条件：
  // - 它是制表符
  // - 它相邻于边缘（开始或结束）
  // - 它相邻于空白（左侧或右侧）
  "show_whitespaces": "selection",
  // 当启用 show_whitespaces 时，用于渲染空白的可见字符。
  "whitespace_map": {
    "space": "•",
    "tab": "→",
  },
  // Zed 中通话相关的设置
  "calls": {
    // 加入通话时默认将麦克风静音
    "mute_on_join": false,
    // 当您是第一个加入频道的人时共享您的项目
    "share_on_join": false,
  },
  // 工具栏相关设置
  "toolbar": {
    // 是否显示面包屑。
    "breadcrumbs": true,
    // 是否显示快速操作按钮。
    "quick_actions": true,
    // 是否在编辑器工具栏中显示选择菜单。
    "selections_menu": true,
    // 是否在编辑器工具栏中显示代理审查按钮。
    "agent_review": true,
    // 是否在编辑器工具栏中显示代码操作按钮。
    "code_actions": false,
  },
  // 是否允许窗口根据用户的标签页偏好合并标签页（仅限 macOS）。
  "use_system_window_tabs": false,
  // 标题栏相关设置
  "title_bar": {
    // 是否在标题栏的分支切换器旁边显示分支图标。
    "show_branch_icon": false,
    // 是否在标题栏中显示分支名称按钮。
    "show_branch_name": true,
    // 是否在标题栏中显示项目主机和名称。
    "show_project_items": true,
    // 是否在标题栏中显示入门指南横幅。
    "show_onboarding_banner": true,
    // 是否在标题栏中显示用户头像。
    "show_user_picture": true,
    // 是否在标题栏中显示用户菜单。
    "show_user_menu": true,
    // 是否在标题栏中显示登录按钮。
    "show_sign_in": true,
    // 是否在标题栏中显示菜单。
    "show_menus": false,
  },
  "audio": {
    // 选择使用新的音频系统。
    "experimental.rodio_audio": false,
    // 需要 'rodio_audio: true'
    //
    // 自动增加或降低麦克风音量。这会影响您的声音对他人的响度。
    //
    // 推荐：关闭（默认）
    // 在 Zed 中麦克风太安静，直到所有人都使用实验性音频并且开启了自动扬声器音量，这会使您的声音相对于其他说话者非常响亮。
    "experimental.auto_microphone_volume": false,
    // 需要 'rodio_audio: true'
    //
    // 自动增加或降低其他通话成员的音量。这仅影响您听到的声音。
    "experimental.auto_speaker_volume": true,
    // 需要 'rodio_audio: true'
    //
    // 去除背景噪音。对打字、汽车、狗、空调效果很好。对音乐效果不佳。
    "experimental.denoise": true,
    // 需要 'rodio_audio: true'
    //
    // 使用与先前版本的实验性音频和非实验性音频兼容的音频参数。当此选项为 false 时，对于未使用最新实验性音频的人来说，您的声音会显得奇怪。将来我们将通过将其设置为 false 来迁移。
    //
    // 您需要重新加入通话才能使此设置生效。
    "experimental.legacy_audio_compatible": true,
  },
  // 滚动条相关设置
  "scrollbar": {
    // 何时在编辑器中显示滚动条。
    // 此设置可取四个值：
    //
    // 1. 如果有重要信息则显示滚动条，或遵循系统的配置行为（默认）：
    //   "auto"
    // 2. 匹配系统的配置行为：
    //    "system"
    // 3. 始终显示滚动条：
    //    "always"
    // 4. 从不显示滚动条：
    //    "never"
    "show": "auto",
    // 是否在滚动条中显示光标位置。
    "cursors": true,
    // 是否在滚动条中显示 git diff 指示器。
    "git_diff": true,
    // 是否在滚动条中显示缓冲区搜索结果。
    "search_results": true,
    // 是否在滚动条中显示所选文本的出现位置。
    "selected_text": true,
    // 是否在滚动条中显示所选符号的出现位置。
    "selected_symbol": true,
    // 在滚动条中显示哪些诊断指示器：
    //  - "none" 或 false：不显示诊断
    //  - "error"：仅显示错误
    //  - "warning"：仅显示错误和警告
    //  - "information"：仅显示错误、警告和信息
    //  - "all" 或 true：显示所有诊断
    "diagnostics": "all",
    // 为每个轴强制启用或禁用滚动条
    "axes": {
      // 当为 false 时，强制禁用水平滚动条。否则，遵循其他设置。
      "horizontal": true,
      // 当为 false 时，强制禁用垂直滚动条。否则，遵循其他设置。
      "vertical": true,
    },
  },
  // 小地图相关设置
  "minimap": {
    // 何时在编辑器中显示小地图。
    // 此设置可取三个值：
    // 1. 如果编辑器的滚动条可见，则显示小地图：
    //    "auto"
    // 2. 始终显示小地图：
    //    "always"
    // 3. 从不显示小地图：
    //    "never"（默认）
    "show": "never",
    // 在编辑器中显示小地图的位置。
    // 此设置可取两个值：
    // 1. 仅在聚焦的编辑器上显示小地图：
    //    "active_editor"（默认）
    // 2. 在所有打开的编辑器上显示小地图：
    //    "all_editors"
    "display_in": "active_editor",
    // 何时显示小地图滑块。
    // 此设置可取两个值：
    // 1. 如果鼠标悬停在小地图上，则显示小地图滑块：
    //    "hover"
    // 2. 始终显示小地图滑块：
    //    "always"（默认）
    "thumb": "always",
    // 小地图滑块边框的外观。
    // 此设置可取五个值：
    // 1. 在滑块的四个边显示边框：
    //    "thumb_border": "full"
    // 2. 在滑块的所有边（左侧除外）显示边框：
    //    "thumb_border": "left_open"（默认）
    // 3. 在滑块的所有边（右侧除外）显示边框：
    //    "thumb_border": "right_open"
    // 4. 仅在滑块左侧显示边框：
    //    "thumb_border": "left_only"
    // 5. 显示不带任何边框的滑块：
    //    "thumb_border": "none"
    "thumb_border": "left_open",
    // 如何在小地图中突出显示当前行。
    // 此设置可取以下值：
    //
    // 1. `null` 继承编辑器的 `current_line_highlight` 设置（默认）
    // 2. "line" 或 "all" 在小地图中突出显示当前行。
    // 3. "gutter" 或 "none" 不在小地图中突出显示当前行。
    "current_line_highlight": null,
    // 在小地图中显示的最大列数。
    "max_width_columns": 80,
  },
  // 在 Linux 上启用中键粘贴。
  "middle_click_paste": true,
  // 当多缓冲区在其某些摘录（单例缓冲区的部分）中被双击时要执行的操作。
  // 可取两个值：
  //  1. 表现为常规缓冲区并选择整个单词（默认）。
  //         "double_click_in_multibuffer": "select"
  //  2. 将点击的摘录作为新缓冲区在新标签页中打开。
  //         "double_click_in_multibuffer": "open",
  // 对于 "open" 的情况，可以通过在双击时按住 `alt` 来实现常规选择行为。
  "double_click_in_multibuffer": "select",
  "gutter": {
    // 是否在装订线中显示行号。
    "line_numbers": true,
    // 是否在装订线中显示可运行按钮。
    "runnables": true,
    // 是否在装订线中显示断点。
    "breakpoints": true,
    // 是否在装订线中显示折叠按钮。
    "folds": true,
    // 在装订线中预留空间的最小字符数。
    "min_line_number_digits": 4,
  },
  "indent_guides": {
    // 是否在编辑器中显示缩进参考线。
    "enabled": true,
    // 缩进参考线的宽度（像素），介于 1 和 10 之间。
    "line_width": 1,
    // 活动缩进参考线的宽度（像素），介于 1 和 10 之间。
    "active_line_width": 1,
    // 确定缩进参考线的着色方式。
    // 此设置可取以下三个值：
    //
    // 1. "disabled"
    // 2. "fixed"
    // 3. "indent_aware"
    "coloring": "fixed",
    // 确定缩进参考线背景的着色方式。
    // 此设置可取以下两个值：
    //
    // 1. "disabled"
    // 2. "indent_aware"
    "background_coloring": "disabled",
  },
  // 编辑器是否会滚动到最后一行之后。
  "scroll_beyond_last_line": "one_page",
  // 使用键盘滚动时，要在光标上方/下方保留的行数。
  "vertical_scroll_margin": 3,
  // 是否在单击可见文本区域边缘附近时滚动。
  "autoscroll_on_clicks": false,
  // 使用鼠标滚动时，要在两侧保留的字符数。
  "horizontal_scroll_margin": 5,
  // 滚动灵敏度乘数。此乘数在滚动时同时应用于水平和垂直增量值。
  "scroll_sensitivity": 1.0,
  // 快速滚动的灵敏度乘数。此乘数在滚动时同时应用于水平和垂直增量值。当用户在滚动时按住 alt 或 option 键时，会触发快速滚动。
  "fast_scroll_sensitivity": 4.0,
  "sticky_scroll": {
    // 是否将作用域固定在编辑器顶部。
    "enabled": false,
  },
  "relative_line_numbers": "disabled",
  // 如果禁用 'search_wrap'，搜索结果不会在文件末尾换行。
  "search_wrap": true,
  // 打开新的项目和缓冲区搜索时默认启用的搜索选项。
  "search": {
    // 是否在状态栏中显示项目搜索按钮。
    "button": true,
    // 是否仅匹配整个单词。
    "whole_word": false,
    // 是否区分大小写。
    "case_sensitive": false,
    // 是否在搜索结果中包含被 gitignore 忽略的文件。
    "include_ignored": false,
    // 是否将搜索查询解释为正则表达式。
    "regex": false,
    // 导航时是否将光标居中于每个搜索匹配项。
    "center_on_match": false,
  },
  // 何时根据光标下的文本填充新搜索的查询。
  // 此设置可取以下三个值：
  //
  // 1. 始终用光标下的单词填充搜索查询（默认）。
  //    "always"
  // 2. 仅当有文本被选中时才填充搜索查询。
  //    "selection"
  // 3. 从不填充搜索查询。
  //    "never"
  "seed_search_query_from_cursor": "always",
  // 启用时，根据查询自动调整搜索的大小写敏感性。
  // 如果搜索查询包含任何大写字母，则搜索变为区分大小写；
  // 如果仅包含小写字母，则搜索变为不区分大小写。
  "use_smartcase_search": false,
  // 内嵌提示相关设置
  "inlay_hints": {
    // 全局开关，用于切换提示的开启和关闭，默认关闭。
    "enabled": false,
    // 切换某些类型提示的开启和关闭，默认全部开启。
    "show_type_hints": true,
    "show_parameter_hints": true,
    "show_value_hints": true,
    // 对应于 null/None 的 LSP 提示类型值。
    "show_other_hints": true,
    // 是否为内嵌提示显示背景。
    //
    // 如果设置为 `true`，背景将使用当前主题中的 `hint.background` 颜色。
    "show_background": false,
    // 编辑缓冲区后等待请求提示的时间，设置为 0 以禁用去抖。
    "edit_debounce_ms": 700,
    // 滚动缓冲区后等待请求提示的时间，设置为 0 以禁用去抖。
    "scroll_debounce_ms": 50,
    // 一组修饰键，当按下时会切换内嵌提示的可见性。
    // 如果该集合为空或未按下所有指定的修饰键，则不会切换内嵌提示。
    "toggle_on_modifiers_press": {
      "control": false,
      "shift": false,
      "alt": false,
      "platform": false,
      "function": false,
    },
  },
  // 调整停靠栏大小时，是否调整停靠栏中所有面板的大小。
  // 可以是 "left"、"right" 和 "bottom" 的组合。
  "resize_all_panels_in_dock": ["left"],
  "project_panel": {
    // 是否在状态栏中显示项目面板按钮
    "button": true,
    // 是否在项目面板中隐藏 gitignore 条目。
    "hide_gitignore": false,
    // 项目面板的默认宽度。
    "default_width": 240,
    // 项目面板的停靠位置。可以是 'left' 或 'right'。
    "dock": "left",
    // 项目面板中工作树条目之间的间距。可以是 'comfortable' 或 'standard'。
    "entry_spacing": "comfortable",
    // 是否在项目面板中显示文件图标。
    "file_icons": true,
    // 是否在项目面板中为目录显示文件夹图标或角标。
    "folder_icons": true,
    // 是否在项目面板中显示 git 状态。
    "git_status": true,
    // 嵌套项的缩进量。
    "indent_size": 20,
    // 当相应的项目条目变为活动状态时，是否自动在项目面板中显示它。
    // 被 gitignore 忽略的条目永远不会自动显示。
    "auto_reveal_entries": true,
    // 项目面板是否应在启动时打开。
    "starts_open": true,
    // 是否自动折叠目录，并在目录内只有一个子目录时显示紧凑文件夹（例如 "a/b/c"）。
    "auto_fold_dirs": true,
    // 是否在项目面板中用粗体文本显示文件夹名称。
    "bold_folder_labels": false,
    // 滚动条相关设置
    "scrollbar": {
      // 何时在项目面板中显示滚动条。
      // 此设置可取五个值：
      //
      // 1. null（默认）：继承编辑器设置
      // 2. 如果有重要信息则显示滚动条，或遵循系统的配置行为（默认）：
      //   "auto"
      // 3. 匹配系统的配置行为：
      //    "system"
      // 4. 始终显示滚动条：
      //    "always"
      // 5. 从不显示滚动条：
      //    "never"
      "show": null,
    },
    // 在项目面板中标记哪些包含诊断错误/警告的文件。
    // 此设置可取以下三个值：
    //
    // 1. 不标记任何文件：
    //    "off"
    // 2. 仅标记有错误的文件：
    //    "errors"
    // 3. 标记有错误和警告的文件：
    //    "all"
    "show_diagnostics": "all",
    // 是否将父目录固定在项目面板顶部。
    "sticky_scroll": true,
    // 项目面板中缩进参考线的相关设置。
    "indent_guides": {
      // 何时在项目面板中显示缩进参考线。
      // 此设置可取两个值：
      //
      // 1. 始终显示缩进参考线：
      //    "always"
      // 2. 从不显示缩进参考线：
      //    "never"
      "show": "always",
    },
    // 项目面板中条目的排序顺序。
    // 此设置可取三个值：
    //
    // 1. 先显示目录，然后显示文件：
    //    "directories_first"
    // 2. 混合显示目录和文件：
    //    "mixed"
    // 3. 先显示文件，然后显示目录：
    //    "files_first"
    "sort_mode": "directories_first",
    // 是否在项目面板中启用拖放操作。
    "drag_and_drop": true,
    // 当窗口中只打开一个文件夹时，是否隐藏根条目。
    "hide_root": false,
    // 是否在项目面板中隐藏隐藏条目。
    "hide_hidden": false,
    // 自动打开文件的设置。
    "auto_open": {
      // 是否在编辑器中自动打开新创建的文件。
      "on_create": true,
      // 粘贴或复制文件后是否自动打开它们。
      "on_paste": true,
      // 是否自动打开从外部源拖放的文件。
      "on_drop": true,
    },
  },
  "outline_panel": {
    // 是否在状态栏中显示大纲面板按钮
    "button": true,
    // 大纲面板的默认宽度。
    "default_width": 300,
    // 大纲面板的停靠位置。可以是 'left' 或 'right'。
    "dock": "left",
    // 是否在大纲面板中显示文件图标。
    "file_icons": true,
    // 是否在大纲面板中为目录显示文件夹图标或角标。
    "folder_icons": true,
    // 是否在大纲面板中显示 git 状态。
    "git_status": true,
    // 嵌套项的缩进量。
    "indent_size": 20,
    // 当相应的大纲条目变为活动状态时，是否自动在大纲面板中显示它。
    // 被 gitignore 忽略的条目永远不会自动显示。
    "auto_reveal_entries": true,
    // 当目录内只有一个目录时，是否自动折叠目录。
    "auto_fold_dirs": true,
    // 大纲面板中缩进参考线的相关设置。
    "indent_guides": {
      // 何时在大纲面板中显示缩进参考线。
      // 此设置可取两个值：
      //
      // 1. 始终显示缩进参考线：
      //    "always"
      // 2. 从不显示缩进参考线：
      //    "never"
      "show": "always",
    },
    // 滚动条相关设置
    "scrollbar": {
      // 何时在项目面板中显示滚动条。
      // 此设置可取五个值：
      //
      // 1. null（默认）：继承编辑器设置
      // 2. 如果有重要信息则显示滚动条，或遵循系统的配置行为（默认）：
      //   "auto"
      // 3. 匹配系统的配置行为：
      //    "system"
      // 4. 始终显示滚动条：
      //    "always"
      // 5. 从不显示滚动条：
      //    "never"
      "show": null,
    },
    // 当前文件中展开大纲项的默认深度。
    // 设置为 0 以折叠所有有子项的项目，设置为 1 或更高以折叠该深度或更深层的项目。
    "expand_outlines_with_depth": 100,
  },
  "collaboration_panel": {
    // 是否在状态栏中显示协作面板按钮。
    "button": true,
    // 协作面板的停靠位置。可以是 'left' 或 'right'。
    "dock": "left",
    // 协作面板的默认宽度。
    "default_width": 240,
  },
  "git_panel": {
    // 是否在状态栏中显示 git 面板按钮。
    "button": true,
    // git 面板的停靠位置。可以是 'left' 或 'right'。
    "dock": "left",
    // git 面板的默认宽度。
    "default_width": 360,
    // 面板中 git 状态指示器的样式。
    //
    // 选项：label_color, icon
    // 默认值：icon
    "status_style": "icon",
    // 如果未设置 `init.defaultBranch`，要使用的分支名称。
    //
    // 默认值：main
    "fallback_branch_name": "main",
    // 是否按路径对面板中的条目进行排序，而不是按状态（默认）。
    //
    // 默认值：false
    "sort_by_path": false,
    // 是否在差异面板中折叠未跟踪的文件。
    //
    // 默认值：false
    "collapse_untracked_diff": false,
    /// 是否以树形视图或扁平视图显示面板中的条目。
    ///
    /// 默认值：false
    "tree_view": false,
    "scrollbar": {
      // 何时在 git 面板中显示滚动条。
      //
      // 选项：always, auto, never, system
      // 默认值：继承编辑器滚动条设置
      // "show": null
    },
  },
  "message_editor": {
    // 是否自动用 emoji 字符替换 emoji 短代码。
    // 例如：键入 `:wave:` 会被替换为 `👋`。
    "auto_replace_emoji_shortcode": true,
  },
  "notification_panel": {
    // 是否在状态栏中显示通知面板按钮。
    "button": true,
    // 通知面板的停靠位置。可以是 'left' 或 'right'。
    "dock": "right",
    // 通知面板的默认宽度。
    "default_width": 380,
  },
  "agent": {
    // 内联助手是否应使用流式工具（如果可用）。
    "inline_assistant_use_streaming_tools": true,
    // 是否启用代理。
    "enabled": true,
    // 是否在状态栏中显示代理面板按钮。
    "button": true,
    // 代理面板的停靠位置。可以是 'left'、'right' 或 'bottom'。
    "dock": "right",
    // 代理列表面板的停靠位置。可以是 'left' 或 'right'。
    "agents_panel_dock": "left",
    // 当代理面板停靠在左侧或右侧时的默认宽度。
    "default_width": 640,
    // 当代理面板停靠在底部时的默认高度。
    "default_height": 320,
    // 默认使用的视图（thread 或 text_thread）。
    "default_view": "thread",
    // 创建新线程时使用的默认模型。
    "default_model": {
      // 要使用的提供商。
      "provider": "zed.dev",
      // 要使用的模型。
      "model": "claude-sonnet-4",
      // 是否启用思考。
      "enable_thinking": false,
    },
    // 语言模型请求的附加参数。当向模型发出请求时，将从此列表中与模型的提供商和名称匹配的最后一个条目中获取参数。在每个条目中，提供商和模型都是可选的，以便您可以为其中任何一个指定参数。
    "model_parameters": [
      // 要为所有对 OpenAI 模型的请求设置参数：
      // {
      //   "provider": "openai",
      //   "temperature": 0.5
      // }
      //
      // 要为所有请求设置参数：
      // {
      //   "temperature": 0
      // }
      //
      // 要为特定提供商和模型设置参数：
      // {
      //   "provider": "zed.dev",
      //   "model": "claude-sonnet-4",
      //   "temperature": 1.0
      // }
    ],
    // 工具操作的权限规则。
    //
    // "default" 设置在没有任何工具特定规则匹配时应用。
    // 对于定义了自己权限模式的外部代理，
    // "deny" 和 "confirm" 仍然优先——仅在 Zed 允许该操作时才使用外部代理的权限系统。
    //
    // 每个工具的 regex 模式（下面的 "tools"）与工具输入文本匹配
    // （命令、路径、URL 等）。对于 `copy_path` 和 `move_path`，
    // 模式会针对每个路径（源和目标）独立匹配。
    "tool_permissions": {
      // 当没有工具特定规则匹配时的全局默认权限。
      // "allow" - 自动批准，无需提示
      // "deny" - 自动拒绝
      // "confirm" - 始终提示（默认）
      "default": "confirm",
      // 每个工具的权限规则。regex 模式与工具输入文本匹配。
      // 每个工具的 "default" 也适用于 MCP 工具。
      // 每个工具可以有自己默认和 regex 模式。
      "tools": {
        // "terminal": {
        //   "default": "confirm",
        //   "always_confirm": [
        //     // 破坏性的 git 操作
        //     { "pattern": "git\\s+(reset|clean)\\s+--hard" },
        //     { "pattern": "git\\s+push\\s+(-f|--force)" },
        //   ],
        // },
        // "edit_file": {
        //   "default": "confirm",
        //   "always_deny": [
        //     // 密钥和凭据
        //     { "pattern": "\\.env($|\\.)" },
        //     { "pattern": "secrets?/" },
        //     { "pattern": "\\.pem$" },
        //     { "pattern": "\\.key$" },
        //   ],
        // },
      },
    },
    // 启用时，代理编辑将显示在单文件编辑器中以供审查。
    "single_file_review": true,
    // 启用时，为代理编辑显示投票拇指以提供反馈。
    "enable_feedback": true,
    "default_profile": "write",
    "profiles": {
      "write": {
        "name": "Write",
        "enable_all_context_servers": true,
        "tools": {
          "copy_path": true,
          "create_directory": true,
          "delete_path": true,
          "diagnostics": true,
          "edit_file": true,
          "fetch": true,
          "list_directory": true,
          "project_notifications": false,
          "move_path": true,
          "now": true,
          "find_path": true,
          "read_file": true,
          "restore_file_from_disk": true,
          "save_file": true,
          "open": true,
          "grep": true,
          "subagent": true,
          "terminal": true,
          "thinking": true,
          "web_search": true,
        },
      },
      "ask": {
        "name": "Ask",
        // 我们不知道哪些上下文服务器工具对 "Ask" 配置文件是安全的，因此默认不启用它们。
        // "enable_all_context_servers": true,
        "tools": {
          "diagnostics": true,
          "fetch": true,
          "list_directory": true,
          "project_notifications": false,
          "now": true,
          "find_path": true,
          "read_file": true,
          "open": true,
          "grep": true,
          "subagent": true,
          "thinking": true,
          "web_search": true,
        },
      },
      "minimal": {
        "name": "Minimal",
        "enable_all_context_servers": false,
        "tools": {},
      },
    },
    // 在代理完成响应或需要确认才能运行工具操作时，在哪里显示通知。
    // "primary_screen" - 仅在主屏幕上显示通知（默认）
    // "all_screens" - 在所有屏幕上显示这些通知
    // "never" - 从不显示这些通知
    "notify_when_agent_waiting": "primary_screen",
    // 当代理完成响应或需要用户输入时，是否播放声音。

    // 默认值：false
    "play_sound_when_agent_done": false,
    // 是否在代理面板中展开编辑卡片，显示完整差异的预览。
    //
    // 默认值：true
    "expand_edit_card": true,
    // 是否在代理面板中展开终端卡片，显示完整的命令输出。
    //
    // 默认值：true
    "expand_terminal_card": true,
    // 单击正在运行的终端工具上的停止按钮是否也应取消代理的生成。
    // 请注意，这仅适用于停止按钮，不适用于终端内的 ctrl+c。
    //
    // 默认值：true
    "cancel_generation_on_terminal_stop": true,
    // 是否始终使用 cmd-enter（在 Linux 或 Windows 上为 ctrl-enter）在代理面板中发送消息。
    //
    // 默认值：false
    "use_modifier_to_send": false,
    // 代理消息编辑器中显示的最小行数。
    //
    // 默认值：4
    "message_editor_min_lines": 4,
    // 是否显示回合统计信息（生成期间的已用时间、最终回合持续时间）。
    //
    // 默认值：false
    "show_turn_stats": false,
  },
  // 是否在操作系统状态栏中显示屏幕共享图标。
  "show_call_status_icon": true,
  // 是否使用语言服务器提供代码智能。
  "enable_language_server": true,
  // 是否执行关联范围的链接编辑，如果语言服务器支持的话。
  // 例如，当编辑开头的 <html> 标签时，结尾的 </html> 标签的内容也会被编辑。
  "linked_edits": true,
  // 用于所有语言的语言服务器列表（或要禁用的列表）。
  //
  // 这通常按每种语言进行自定义。
  "language_servers": ["..."],
  // 控制如何将语言服务器的语义标记用于语法高亮显示。
  //
  // 选项：
  // - "off": 不从语言服务器请求语义标记。
  // - "combined": 将 LSP 语义标记与 tree-sitter 高亮显示一起用作基础。
  // - "full": 仅使用 LSP 语义标记来高亮显示文本，tree-sitter 语法高亮显示关闭。
  //
  // 可能需要重启语言服务器才能正确应用。
  "semantic_tokens": "off",

  // 控制是否使用来自语言服务器的折叠范围而不是 tree-sitter 和基于缩进的折叠。
  //
  // 选项：
  // - "off": 使用 tree-sitter 和基于缩进的折叠（默认）。
  // - "on": 尽可能使用 LSP 折叠，当服务器未返回结果时回退到 tree-sitter 和基于缩进的折叠。
  "document_folding_ranges": "off",

  // 何时自动保存编辑的缓冲区。此设置可取四个值。
  //
  // 1. 从不自动保存：
  //     "autosave": "off",
  // 2. 当焦点从 Zed 窗口移开时保存：
  //     "autosave": "on_window_change",
  // 3. 当焦点从特定缓冲区移开时保存：
  //     "autosave": "on_focus_change",
  // 4. 空闲一段时间后保存：
  //     "autosave": { "after_delay": {"milliseconds": 500} },
  "autosave": "off",
  // 每个窗格的最大标签页数。不设置则无限制。
  "max_tabs": null,
  // 编辑器标签栏的相关设置。
  "tab_bar": {
    // 是否在编辑器中显示标签栏
    "show": true,
    // 是否显示导航历史按钮。
    "show_nav_history_buttons": true,
    // 是否显示标签栏按钮。
    "show_tab_bar_buttons": true,
    // 是否将固定的标签页显示在单独的行中。
    // 启用时，固定的标签页显示在顶行，未固定的标签页显示在底行。
    // 禁用时，所有标签页显示在一行中（默认行为）。
    "show_pinned_tabs_in_separate_row": false,
  },
  // 编辑器标签页的相关设置
  "tabs": {
    // 在编辑器标签页中显示 git 状态颜色。
    "git_status": false,
    // 编辑器标签页上关闭按钮的位置。
    // 可以是：["right", "left"]
    "close_position": "right",
    // 是否显示标签页的文件图标。
    "file_icons": false,
    // 控制标签页关闭按钮的显示行为。
    //
    // 1. 仅在鼠标悬停标签页时显示。（默认）
    //     "hover"
    // 2. 始终显示。
    //     "always"
    // 3. 从不显示，即使悬停也不显示。
    //     "hidden"
    "show_close_button": "hover",
    // 关闭当前标签页后的操作。
    //
    // 1. 激活之前打开的标签页（默认）
    //     "history"
    // 2. 如果存在，则激活右侧相邻标签页
    //     "neighbour"
    // 3. 如果存在，则激活左侧相邻标签页
    //     "left_neighbour"
    "activate_on_close": "history",
    // 在标签页中标记哪些包含诊断错误/警告的文件。
    // 仅当文件图标也处于活动状态时，才会显示诊断信息。
    // 此设置仅适用于以下三个值：
    //
    // 1. 不标记任何文件：
    //    "off"
    // 2. 仅标记有错误的文件：
    //    "errors"
    // 3. 标记有错误和警告的文件：
    //    "all"
    "show_diagnostics": "off",
  },
  // 预览标签页的相关设置。
  "preview_tabs": {
    // 是否启用预览标签页。
    // 预览标签页允许您以预览模式打开文件，当您打开另一个预览标签页时，它们会自动关闭。
    // 这对于快速查看文件而不使工作区杂乱很有用。
    "enabled": true,
    // 当从项目面板中通过单击打开标签页时，是否以预览模式打开。
    "enable_preview_from_project_panel": true,
    // 当从文件查找器中选择标签页时，是否以预览模式打开。
    "enable_preview_from_file_finder": false,
    // 当从多缓冲区打开标签页时，是否以预览模式打开。
    "enable_preview_from_multibuffer": true,
    // 当使用代码导航打开多缓冲区时，是否以预览模式打开标签页。
    "enable_preview_multibuffer_from_code_navigation": false,
    // 当使用代码导航打开单个文件时，是否以预览模式打开标签页。
    "enable_preview_file_from_code_navigation": true,
    // 当使用代码导航从预览标签页导航离开时，是否保持预览模式。
    // 如果 `enable_preview_file_from_code_navigation` 或 `enable_preview_multibuffer_from_code_navigation` 也为 true，新标签页可能会替换现有标签页。
    "enable_keep_preview_on_code_navigation": false,
  },
  // 文件查找器的相关设置。
  "file_finder": {
    // 是否在文件查找器中显示文件图标。
    "file_icons": true,
    // 确定文件查找器可以占用的空间相对于可用窗口宽度的比例。
    // 有 5 种可能的宽度值：
    //
    // 1. Small：此值本质上是固定宽度。
    //    "modal_max_width": "small"
    // 2. Medium：
    //    "modal_max_width": "medium"
    // 3. Large：
    //    "modal_max_width": "large"
    // 4. Extra Large：
    //    "modal_max_width": "xlarge"
    // 5. Fullscreen：此值移除所有水平填充，因为它占用整个视口宽度。
    //    "modal_max_width": "full"
    //
    // 默认值：small
    "modal_max_width": "small",
    // 确定文件查找器是否应在搜索结果中跳过对活动文件的焦点。
    // 有 2 种可能的值：
    //
    // 1. true：搜索文件时，如果当前活动文件显示为第一个结果，
    //    自动焦点将跳过它，而将焦点放在第二个结果上。
    //    "skip_focus_for_active_in_search": true
    //
    // 2. false：搜索文件时，第一个结果将始终获得焦点，
    //    即使它是当前活动文件。
    //    "skip_focus_for_active_in_search": false
    //
    // 默认值：true
    "skip_focus_for_active_in_search": true,
    // 是否在文件查找器中显示 git 状态。
    "git_status": true,
    // 搜索时是否使用被 gitignore 忽略的文件。
    // 只会使用 Zed 已索引的文件，不一定是所有被 gitignore 忽略的文件。
    //
    // 可以接受 3 个值：
    //   * "all": 使用所有被 gitignore 忽略的文件
    //   * "indexed": 仅使用 Zed 已索引的文件
    //   * "smart": 智能地，当从被 gitignore 忽略的工作树调用时搜索被忽略的文件
    "include_ignored": "smart",
  },
  // 是否在保存缓冲区之前删除行中的任何尾随空白。
  "remove_trailing_whitespace_on_save": true,
  // 当上一行也是注释时，是否在新行以注释开头。
  "extend_comment_on_newline": true,
  // 按下回车时是否继续 markdown 列表。
  "extend_list_on_newline": true,
  // 按下制表符后，是否缩进列表项（在列表标记之后）。
  "indent_list_on_tab": true,
  // 删除文件末尾仅包含空白的任何行，并确保末尾只有一个换行符。
  "ensure_final_newline_on_save": true,
  // 是否在保存前执行缓冲区格式化：[on, off]
  // 请记住，如果启用了带有延迟的自动保存，format_on_save 将被忽略。
  "format_on_save": "on",
  // 如何执行缓冲区格式化。此设置可取多个值：
  //
  // 1. 默认。使用 Zed 的 Prettier 集成（如果适用）格式化文件，
  //    或回退到通过语言服务器格式化：
  //     "formatter": "auto"
  // 2. 使用当前语言服务器格式化代码：
  //     "formatter": "language_server"
  // 3. 使用特定的语言服务器格式化代码：
  //     "formatter": {"language_server": {"name": "ruff"}}
  // 4. 使用外部命令格式化代码：
  //     "formatter": {
  //       "external": {
  //         "command": "prettier",
  //         "arguments": ["--stdin-filepath", "{buffer_path}"]
  //       }
  //     }
  // 5. 使用 Zed 的 Prettier 集成格式化代码：
  //     "formatter": "prettier"
  // 6. 使用代码操作格式化代码
  //     "formatter": {"code_action": "source.fixAll.eslint"}
  // 7. 上述任何格式化步骤的数组，按顺序应用
  //     "formatter": [{"code_action": "source.fixAll.eslint"}, "prettier"]
  "formatter": "auto",
  // 如何软换行长行文本。
  // 可能的值：
  //
  // 1. 通常首选单行，除非遇到过长的行。
  //      "soft_wrap": "none",
  //      "soft_wrap": "prefer_line", // （已弃用，与 "none" 相同）
  // 2. 对超出编辑器宽度的行进行软换行。
  //      "soft_wrap": "editor_width",
  // 3. 在首选行长度处软换行。
  //      "soft_wrap": "preferred_line_length",
  // 4. 在首选行长度或编辑器宽度（取较小者）处软换行。
  //      "soft_wrap": "bounded",
  "soft_wrap": "none",
  // 对于启用了软换行的缓冲区，进行软换行的列位置。
  "preferred_line_length": 80,
  // 是否使用制表符而不是多个空格缩进行。
  "hard_tabs": false,
  // 制表符应占用的列数。
  "tab_size": 4,
  // 所有语言的默认首选调试器。
  "debuggers": [],
  // 是否在编辑器中启用单词差异高亮显示。
  //
  // 启用时，修改行内的更改单词会被高亮显示，以精确显示更改内容。
  //
  // 默认值：true
  "word_diff_enabled": true,
  // 控制 Zed 收集哪些信息。
  "telemetry": {
    // 发送调试信息，如崩溃报告。
    "diagnostics": true,
    // 发送匿名使用数据，如您使用 Zed 的语言。
    "metrics": true,
  },
  // 是否禁用 Zed 中的所有 AI 功能。
  //
  // 默认值：false
  "disable_ai": false,
  // 自动更新 Zed。如果通过包管理器安装，此设置可能在 Linux 上被忽略。
  "auto_update": true,
  // 如何在编辑器中渲染 LSP `textDocument/documentColor` 颜色。
  //
  // 可能的值：
  //
  // 1. 不查询和渲染文档颜色。
  //      "lsp_document_colors": "none",
  // 2. 在颜色文本附近以内嵌提示形式渲染文档颜色（默认）。
  //      "lsp_document_colors": "inlay",
  // 3. 在颜色文本周围绘制边框。
  //      "lsp_document_colors": "border",
  // 4. 在颜色文本后面绘制背景。
  //      "lsp_document_colors": "background",
  "lsp_document_colors": "inlay",
  // 诊断配置。
  "diagnostics": {
    // 是否在状态栏中显示项目诊断按钮。
    "button": true,
    // 是否默认显示警告。
    //
    // 默认值：true
    "include_warnings": true,
    // 在 Zed 中使用 LSP 拉取诊断机制的设置。
    "lsp_pull_diagnostics": {
      // 是否拉取诊断。
      "enabled": true,
      // 从语言服务器拉取诊断前等待的最小时间。
      // 0 表示关闭去抖。
      "debounce_ms": 50,
    },
    // 内联诊断的设置
    "inline": {
      // 是否内联显示诊断
      "enabled": false,
      // 上次诊断更新后显示内联诊断的延迟（毫秒）。
      "update_debounce_ms": 150,
      // 源行末尾与内联诊断开始之间的填充量，以 em 宽度为单位。
      "padding": 4,
      // 显示内联诊断的最小列数。此设置可用于在某些列水平对齐内联诊断。长于此值的行仍会将诊断推得更右。
      "min_column": 0,
      // 要内联显示的诊断的最小严重性。
      // 当为 `null` 时，继承编辑器的诊断最大严重性设置。
      "max_severity": null,
    },
  },
  // Zed 将完全排除的文件或文件 glob。它们将在文件扫描、文件搜索期间被跳过，并且不会显示在项目文件树中。优先于 `file_scan_inclusions`。
  "file_scan_exclusions": [
    "**/.git",
    "**/.svn",
    "**/.hg",
    "**/.jj",
    "**/.sl",
    "**/.repo",
    "**/CVS",
    "**/.DS_Store",
    "**/Thumbs.db",
    "**/.classpath",
    "**/.settings",
  ],
  // Zed 将包含的文件或文件 glob，即使被 git 忽略。这对于未被 git 跟踪但对项目仍然重要的文件很有用。请注意，过于宽泛的 glob 可能会减慢 Zed 的文件扫描速度。`file_scan_exclusions` 优先于这些包含项。
  "file_scan_inclusions": [".env*"],
  // 与文件匹配的 glob，这些文件将被视为“隐藏”。通过切换 "hide_hidden" 设置，可以在项目面板中隐藏这些文件。
  "hidden_files": ["**/.*"],
  // 与文件匹配的 glob，这些文件将作为只读打开。您仍然可以查看这些文件，但不能编辑它们。这对于生成的文件或外部依赖项很有用。
  "read_only_files": [],
  // Git 装订线行为配置。
  "git": {
    // 全局开关，用于启用或禁用所有 git 集成功能。
    // 如果设置为 true，则禁用所有 git 集成功能。
    // 如果设置为 false，则下面的各个 git 集成功能将独立启用或禁用。
    "disable_git": false,
    // 是否启用 git 状态跟踪。
    "enable_status": true,
    // 是否启用 git 差异显示。
    "enable_diff": true,
    // 控制是否显示 git 装订线。可取 2 个值：
    // 1. 显示装订线
    //      "git_gutter": "tracked_files"
    // 2. 隐藏装订线
    //      "git_gutter": "hide"
    "git_gutter": "tracked_files",
    /// 设置更改反映在 git 装订线中的去抖阈值（毫秒）。
    ///
    /// 默认值：0
    "gutter_debounce": 0,
    // 控制是否在当前聚焦的行内联显示 git blame 信息。
    "inline_blame": {
      "enabled": true,
      // 设置显示内联 blame 信息的延迟。
      // 每次光标移动都会重新启动延迟。
      "delay_ms": 0,
      // 源行末尾与内联 blame 开始之间的填充量，以 em 宽度为单位。
      "padding": 7,
      // 是否在同一行显示 git 提交摘要。
      "show_commit_summary": false,
      // 显示内联 blame 信息的最小列号。
      "min_column": 0,
    },
    "blame": {
      "show_avatar": true,
    },
    // 控制在分支选择器中显示哪些信息。
    "branch_picker": {
      "show_author_name": true,
    },
    // 如何在编辑器中直观显示 git 块。
    // 此设置可取两个值：
    //
    // 1. 显示实心的未暂存块和空心的已暂存块：
    //    "hunk_style": "staged_hollow"
    // 2. 显示空心的未暂存块和实心的已暂存块：
    //    "hunk_style": "unstaged_hollow"
    "hunk_style": "staged_hollow",
    // 在 git 视图中，应该先显示名称还是路径。
    // "path_style": "file_name_first" 或 "file_path_first"
    "path_style": "file_name_first",
  },
  // 自定义 Git 托管服务提供商列表。
  "git_hosting_providers": [
    // {
    //   "provider": "github",
    //   "name": "BigCorp GitHub",
    //   "base_url": "https://code.big-corp.com"
    // }
  ],
  // 如何加载 direnv 配置的配置。可取 2 个值：
  // 1. 使用 `direnv export json` 直接加载 direnv 配置。
  //      "load_direnv": "direct"
  // 2. 通过 shell 钩子加载 direnv 配置，适用于 POSIX shell 和 fish。
  //      "load_direnv": "shell_hook"
  // 3. 根本不加载 direnv 配置。
  //      "load_direnv": "disabled"
  "load_direnv": "direct",
  "edit_predictions": {
    // 要使用的编辑预测提供商。
    "provider": "zed",
    // 表示应禁用编辑预测的文件的 glob 列表。
    // 已经包含了一个合理的默认 glob 列表。
    // 对此列表的任何添加都将与默认列表合并。
    // Glob 相对于工作树根进行匹配，
    // 除非以斜杠（/）或 Windows 中的等效符号开头。
    "disabled_globs": [
      "**/.env*",
      "**/*.pem",
      "**/*.key",
      "**/*.cert",
      "**/*.crt",
      "**/.dev.vars",
      "**/secrets.yml",
      "**/.zed/settings.json", // zed 项目设置
      "/**/zed/settings.json", // zed 用户设置
      "/**/zed/keymap.json",
    ],
    // 何时在缓冲区中显示编辑预测预览。
    // 此设置有两个可能的值：
    // 1. 当没有语言服务器补全可用时，内联显示预测。
    //     "mode": "eager"
    // 2. 仅在按住修饰键（默认 alt）时内联显示预测。
    //     "mode": "subtle"
    "mode": "eager",
    // Copilot 特定设置
    // "copilot": {
    //   "enterprise_uri": "",
    //   "proxy": "",
    //   "proxy_no_verify": false
    //   "enable_next_edit_suggestions": true
    // },
    "copilot": {
      "enterprise_uri": null,
      "proxy": null,
      "proxy_no_verify": null,
      "enable_next_edit_suggestions": true,
    },
    "codestral": {
      "api_url": "https://codestral.mistral.ai",
      "model": "codestral-latest",
      "max_tokens": 150,
    },
    "sweep": {
      // 启用时，Sweep 不会存储编辑预测输入或输出。
      // 禁用时，Sweep 可能会收集数据，包括缓冲区内容、
      // 诊断、文件路径、存储库名称和生成的预测，以改进服务。
      "privacy_mode": false,
    },
    "ollama": {
      "api_url": "http://localhost:11434",
      "model": "qwen2.5-coder:7b-base",
      "max_output_tokens": 64,
    },
    // 在代理面板中编辑文本线程时，是否启用编辑预测。
    // 如果全局禁用，此设置无效。
    "enabled_in_text_threads": true,
  },
  // 特定于日记的设置
  "journal": {
    // 存储日记条目的目录路径
    "path": "~",
    // 显示小时数的格式
    // 可取 2 个值：
    // 1. hour12
    // 2. hour24
    "hour_format": "hour12",
  },
  // 状态栏相关设置。
  "status_bar": {
    // 是否显示状态栏。
    "experimental.show": true,
    // 是否在状态栏中显示活动语言按钮。
    "active_language_button": true,
    // 是否在状态栏中显示光标位置按钮。
    "cursor_position_button": true,
    // 是否在状态栏中显示活动行结尾按钮。
    "line_endings_button": false,
    // 控制在状态栏中何时显示活动编码。
    "active_encoding_button": "non_utf8",
  },
  // 特定于终端的设置
  "terminal": {
    // 打开终端时使用的 shell。可取 3 个值：
    // 1. 使用 /etc/passwd 中的系统默认终端配置
    //      "shell": "system"
    // 2. 一个程序：
    //      "shell": {
    //        "program": "sh"
    //      }
    // 3. 带参数的程序：
    //     "shell": {
    //         "with_arguments": {
    //           "program": "/bin/bash",
    //           "args": ["--login"]
    //         }
    //     }
    "shell": "system",
    // 终端面板的停靠位置。可以是 `left`、`right`、`bottom`。
    "dock": "bottom",
    // 当终端停靠在左侧或右侧时的默认宽度。
    "default_width": 640,
    // 当终端停靠在底部时的默认高度。
    "default_height": 320,
    // 启动终端时使用的工作目录。
    // 可取 5 个值：
    // 1. 使用当前文件所在的目录，回退到项目
    //    目录，然后回退到工作区中的第一个项目。
    //      "working_directory": "current_file_directory"
    // 2. 使用当前文件的项目目录。如果不成功，回退到
    //    第一个项目目录策略。
    //      "working_directory": "current_project_directory"
    // 3. 使用此工作区中第一个项目的目录
    //      "working_directory": "first_project_directory"
    // 4. 始终使用此平台的主目录（如果找到）
    //     "working_directory": "always_home"
    // 5. 始终使用特定目录。此值将进行 shell 扩展。
    //    如果此路径不是有效的目录，终端将默认使用
    //    此平台的主目录（如果找到）
    //      "working_directory": {
    //        "always": {
    //          "directory": "~/zed/projects/"
    //        }
    //      }
    "working_directory": "current_project_directory",
    // 设置终端中的光标闪烁行为。
    // 可取 3 个值：
    //  1. 从不闪烁光标，忽略终端模式
    //         "blinking": "off",
    //  2. 默认光标闪烁关闭，但允许终端
    //     设置闪烁
    //         "blinking": "terminal_controlled",
    //  3. 始终闪烁光标，忽略终端模式
    //         "blinking": "on",
    "blinking": "terminal_controlled",
    // 终端的默认光标形状。
    //  1. 包围后续字符的块
    //     "block"
    //  2. 垂直条
    //     "bar"
    //  3. 沿后续字符的下划线
    //     "underline"
    //  4. 包围后续字符的框
    //     "hollow"
    //
    // 默认值："block"
    "cursor_shape": "block",
    // 设置备用滚动模式（代码：?1007）是否默认激活。
    // 备用滚动模式在备用屏幕中（例如运行 vim 或 less 等应用程序时）将鼠标滚动事件转换为向上/向下键按下。终端仍可设置和取消此模式。
    // 可取 2 个值：
    //  1. 默认开启备用滚动模式
    //         "alternate_scroll": "on",
    //  2. 默认关闭备用滚动模式
    //         "alternate_scroll": "off",
    "alternate_scroll": "on",
    // 设置 option 键是否作为 meta 键。
    // 可取 2 个值：
    //  1. 依赖于 option 键的默认平台处理，在 macOS 上
    //     这意味着生成某些 Unicode 字符
    //         "option_as_meta": false,
    //  2. 使 option 键作为 'meta' 键，例如用于 emacs
    //         "option_as_meta": true,
    "option_as_meta": false,
    // 在终端中选择文本时是否自动复制到系统剪贴板。
    "copy_on_select": false,
    // 复制到剪贴板后是否保留文本选择。
    "keep_selection_on_copy": true,
    // 是否在状态栏中显示终端按钮
    "button": true,
    // 添加到此列表的任何键值对都将添加到终端的环境中。使用 `:` 分隔多个值。
    "env": {
      // "KEY": "value1:value2"
    },
    // 设置终端的行高。
    // 可取 3 个值：
    //  1. 使用适合阅读的行高，1.618
    //         "line_height": "comfortable"
    //  2. 使用标准行高，1.3。此选项对于 TUI 很有用，
    //     特别是如果它们使用框字符
    //         "line_height": "standard",
    //  3. 使用自定义行高。
    //         "line_height": {
    //           "custom": 2
    //         },
    "line_height": "standard",
    // 在终端的工作目录中（由 working_directory 设置解析）激活 Python 虚拟环境（如果找到）。将其设置为 "off" 以禁用此行为。
    "detect_venv": {
      "on": {
        // 搜索虚拟环境的默认目录，相对于当前工作目录。我们建议在项目设置中而不是全局覆盖此设置。
        "directories": [".env", "env", ".venv", "venv"],
        // 也可以是 `csh`、`fish`、`nushell` 和 `power_shell`
        "activate_script": "default",
        // 激活 Conda 环境时要使用的首选 Conda 管理器。
        // 值："auto", "conda", "mamba", "micromamba"
        // 默认值："auto"
        "conda_manager": "auto",
      },
    },
    "toolbar": {
      // 是否在其工具栏的面包屑中显示终端标题。
      // 仅当终端标题不为空时显示。
      //
      // 需要在终端中运行的 shell 配置为发出标题。
      // 示例：`echo -e "\e]2;New Title\007";`
      "breadcrumbs": false,
    },
    // 滚动条相关设置
    "scrollbar": {
      // 何时在终端中显示滚动条。
      // 此设置可取五个值：
      //
      // 1. null（默认）：继承编辑器设置
      // 2. 如果有重要信息则显示滚动条，或遵循系统的配置行为（默认）：
      //   "auto"
      // 3. 匹配系统的配置行为：
      //    "system"
      // 4. 始终显示滚动条：
      //    "always"
      // 5. 从不显示滚动条：
      //    "never"
      "show": null,
    },
    // 设置终端的字体大小。如果未包含此选项，终端将默认匹配缓冲区的字体大小。
    // "font_size": 15,
    // 设置终端的字体系列。如果未包含此选项，终端将默认匹配缓冲区的字体系列。
    // "font_family": ".ZedMono",
    // 设置终端的字体后备。如果未包含此选项，终端将默认匹配缓冲区的字体后备。
    // 这将与平台的默认字体后备合并。
    // "font_fallbacks": ["FiraCode Nerd Fonts"],
    // 编辑器字体的粗细，采用标准 CSS 单位，范围从 100 到 900。
    "font_weight": 400,
    // 设置终端回滚缓冲区中的最大行数。
    // 默认值：10_000，最大值：100_000（所有更大的值都将被视为 100_000），0 表示禁用滚动。
    // 现有终端在重新创建之前不会应用此更改。
    "max_scroll_history_lines": 10000,
    // 终端中滚动速度的乘数。
    "scroll_multiplier": 1.0,
    // 前景色和背景色之间的最小 APCA 感知对比度。
    // APCA（可访问感知对比度算法）比 WCAG 2.x 更准确，尤其是在暗色模式下。值范围从 0 到 106。
    //
    // 基于 APCA 可读性标准（ARC）青铜简单模式：
    // https://readtech.org/ARC/tests/bronze-simple-mode/
    // - 0：无对比度调整
    // - 45：大型流畅文本的最小值（36px+）
    // - 60：其他内容文本的最小值
    // - 75：正文文本的最小值
    // - 90：正文文本的首选值
    //
    // 大多数终端主题的 APCA 值为 40-70。
    // 值为 45 可在确保可读性的同时保留彩色主题。
    "minimum_contrast": 45,
    // 用于标识超链接导航路径的正则表达式。支持可选的命名捕获组 `path`、`line`、`column` 和 `link`。如果这些都不存在，则整个匹配项是超链接目标。如果存在 `path`，则它是超链接目标，如果存在 `line` 和 `column`，则也包括它们。`link` 可用于自定义终端中哪些文本是超链接的一部分。如果不存在 `link`，则使用整个匹配项的文本。如果不存在 `line` 和 `column`，则使用默认的内置行和列后缀处理，该处理解析 `line:column` 和 `(line,column)` 变体。默认值处理 Python 诊断和常见路径、行、列语法。这可以扩展或替换以处理特定场景。例如，要支持对 rust 输出中包含空格的路径进行超链接，
    //
    // [
    //   "\\s+(-->|:::|at) (?<link>(?<path>.+?))(:$|$)",
    //   "\\s+(Compiling|Checking|Documenting) [^(]+\\((?<link>(?<path>.+))\\)"
    // ],
    //
    // 可以使用。处理在第一个匹配的正则表达式处停止，即使没有产生链接（当光标未悬停在超链接文本上时就是这种情况）。为了获得最佳性能，建议按从最常见到最不常见的顺序排列正则表达式。为了可读性和文档记录，每个正则表达式可以是一个字符串数组，这些字符串被收集成一个多行正则表达式字符串，用于终端路径超链接检测。
    "path_hyperlink_regexes": [
      // Python 风格的诊断
      "File \"(?<path>[^\"]+)\", line (?<line>[0-9]+)",
      // 常见路径语法，带有可选的行、列、描述、尾随标点符号或周围符号或引号
      [
        "(?x)",
        "(?<path>",
        "    (",
        "        # 多字符路径：第一个字符（不是开分隔符或空格）",
        "        [^({\\[<\"'`\\ ]",
        "        # 中间字符：非空格，并且冒号/括号仅在不后跟数字/括号时",
        "        ([^\\ :(]|[:(][^0-9()])*",
        "        # 最后一个字符：不是闭分隔符或冒号",
        "        [^()}\\]>\"'`.,;:\\ ]",
        "    |",
        "        # 单字符路径：不是分隔符、标点符号或空格",
        "        [^(){}\\[\\]<>\"'`.,;:\\ ]",
        "    )",
        "    # 可选的行/列后缀（包含在 path 中，用于 PathWithPosition::parse_str）",
        "    (:+[0-9]+(:[0-9]+)?|:?\\([0-9]+([,:]?[0-9]+)?\\))?",
        ")",
      ],
    ],
    // 悬停和 Cmd-click 路径超链接发现的超时时间（毫秒）。指定超时为 `0` 将禁用终端中的路径超链接。
    "path_hyperlink_timeout_ms": 1,
  },
  "code_actions_on_format": {},
  // 与运行任务相关的设置。
  "tasks": {
    "variables": {},
    "enabled": true,
    // 优先使用 LSP 任务而不是 Zed 语言扩展任务。
    // 如果没有 LSP 任务返回（由于错误/超时或正常执行），
    // 将使用 Zed 语言扩展任务。
    //
    // 其他 Zed 任务仍将显示：
    // * 来自任一任务配置文件的 Zed 任务
    // * 来自历史记录的 Zed 任务（例如之前曾生成过一次性任务）
    //
    // 默认值：true
    "prefer_lsp": true,
  },
  // 一个对象，其键是语言名称，其值是要使用这些语言的文件名或扩展名数组。
  //
  // 例如，要将 `foo.notjs` 之类的文件视为 JavaScript，
  // 并将 `Embargo.lock` 视为 TOML：
  //
  // {
  //   "JavaScript": ["notjs"],
  //   "TOML": ["Embargo.lock"]
  // }
  //
  "file_types": {
    "JSONC": [
      "**/.zed/*.json",
      "**/.vscode/**/*.json",
      "**/{zed,Zed}/{settings,keymap,tasks,debug}.json",
      "tsconfig*.json",
    ],
    "Markdown": [".rules", ".cursorrules", ".windsurfrules", ".clinerules"],
    "Shell Script": [".env.*"],
  },
  // 安装语言服务器和 Copilot 时要使用的 Node.js 和 NPM 版本的设置。
  //
  // 注意：更改此设置目前需要重启 Zed。
  "node": {
    // 默认情况下，Zed 会在您的 `$PATH` 中查找 `node` 和 `npm`，如果现有可执行文件的版本足够新，则使用它们。将此设置为 `true` 以防止此行为，并强制 Zed 始终下载并安装自己的 Node 版本。
    "ignore_system_version": false,
    // 您还可以指定 Node 和 NPM 的替代路径。如果指定了 `path` 但未指定 `npm_path`，Zed 将假定 `npm` 位于 `${path}/../npm`。
    "path": null,
    "npm_path": null,
  },
  // Zed 应在启动时自动安装的扩展。
  //
  // 如果您不想要这些扩展中的任何一个，请将此字段添加到您的设置中并将值更改为 `false`。
  "auto_install_extensions": {
    "html": true,
  },
  // 授予扩展的能力。
  //
  // 可以自定义此列表以限制扩展能够执行的操作。
  "granted_extension_capabilities": [
    { "kind": "process:exec", "command": "*", "args": ["**"] },
    { "kind": "download_file", "host": "*", "path": ["**"] },
    { "kind": "npm:install", "package": "*" },
  ],
  // 控制如何为此语言处理补全。
  "completions": {
    // 控制如何补全单词。
    // 对于大型文档，可能不会获取所有单词进行补全。
    //
    // 可取 3 个值：
    // 1. "enabled"
    //   始终获取文档的单词进行补全以及 LSP 补全。
    // 2. "fallback"
    //   仅当 LSP 响应错误或超时时，使用文档的单词显示补全。
    // 3. "disabled"
    //   从不获取或补全文档的单词进行补全。
    //   （仍可通过单独的操作查询基于单词的补全）
    //
    // 默认值：fallback
    "words": "fallback",
    // 自动触发基于单词的补全所需的最小字符数。
    // 在此之前，仍然可以通过相应的编辑器命令手动触发基于单词的补全。
    //
    // 默认值：3
    "words_min_length": 3,
    // 是否获取 LSP 补全。
    //
    // 默认值：true
    "lsp": true,
    // 获取 LSP 补全时，确定等待特定服务器响应的最长时间。
    // 设置为 0 时，无限期等待。
    //
    // 默认值：0
    "lsp_fetch_timeout_ms": 0,
    // 控制接受 LSP 补全时要替换的范围。
    //
    // 当 LSP 服务器提供 `InsertReplaceEdit` 补全时，它们提供两个范围：`insert` 和 `replace`。通常，`insert`
    // 包含光标前的单词前缀，而 `replace` 包含整个单词。
    //
    // 实际上，此设置仅更改 Zed 是否使用接收到的范围用于 `insert` 或 `replace`，因此结果可能因底层 LSP 服务器而异。
    //
    // 可能的值：
    // 1. "insert"
    //   使用 LSP 规范中描述的 `insert` 范围替换光标前的文本。
    // 2. "replace"
    //   使用 LSP 规范中描述的 `replace` 范围替换光标前和光标后的文本。
    // 3. "replace_subsequence"
    //   如果要替换的文本是补全文本的子序列，则行为类似于 `"replace"`，否则类似于 `"insert"`。
    // 4. "replace_suffix"
    //   如果光标后的文本是补全的后缀，则行为类似于 `"replace"`，否则类似于 `"insert"`。
    "lsp_insert_mode": "replace_suffix",
  },
  // 特定语言的设置。
  "languages": {
    "Astro": {
      "language_servers": ["astro-language-server", "..."],
      "prettier": {
        "allowed": true,
        "plugins": ["prettier-plugin-astro"],
      },
    },
    "Blade": {
      "prettier": {
        "allowed": true,
      },
    },
    "C": {
      "format_on_save": "off",
      "use_on_type_format": false,
      "prettier": {
        "allowed": false,
      },
    },
    "C++": {
      "format_on_save": "off",
      "use_on_type_format": false,
      "prettier": {
        "allowed": false,
      },
    },
    "CSharp": {
      "language_servers": ["roslyn", "!omnisharp", "..."],
    },
    "CSS": {
      "prettier": {
        "allowed": true,
      },
    },
    "Dart": {
      "tab_size": 2,
    },
    "Diff": {
      "show_edit_predictions": false,
      "remove_trailing_whitespace_on_save": false,
      "ensure_final_newline_on_save": false,
    },
    "Elixir": {
      "language_servers": [
        "elixir-ls",
        "!expert",
        "!next-ls",
        "!lexical",
        "...",
      ],
    },
    "Elm": {
      "tab_size": 4,
    },
    "Erlang": {
      "language_servers": ["erlang-ls", "!elp", "..."],
    },
    "Git Commit": {
      "allow_rewrap": "anywhere",
      "soft_wrap": "editor_width",
      "preferred_line_length": 72,
    },
    "Go": {
      "hard_tabs": true,
      "code_actions_on_format": {
        "source.organizeImports": true,
      },
      "debuggers": ["Delve"],
    },
    "GraphQL": {
      "prettier": {
        "allowed": true,
      },
    },
    "HEEX": {
      "language_servers": [
        "elixir-ls",
        "!expert",
        "!next-ls",
        "!lexical",
        "...",
      ],
    },
    "HTML": {
      "prettier": {
        "allowed": true,
      },
    },
    "HTML+ERB": {
      "language_servers": ["herb", "!ruby-lsp", "..."],
    },
    "Java": {
      "prettier": {
        "allowed": true,
        "plugins": ["prettier-plugin-java"],
      },
    },
    "JavaScript": {
      "language_servers": ["!typescript-language-server", "vtsls", "..."],
      "prettier": {
        "allowed": true,
      },
    },
    "JSON": {
      "prettier": {
        "allowed": true,
      },
    },
    "JSONC": {
      "prettier": {
        "allowed": true,
      },
    },
    "JS+ERB": {
      "language_servers": ["!ruby-lsp", "..."],
    },
    "Kotlin": {
      "language_servers": ["!kotlin-language-server", "kotlin-lsp", "..."],
    },
    "LaTeX": {
      "formatter": "language_server",
      "language_servers": ["texlab", "..."],
      "prettier": {
        "allowed": true,
        "plugins": ["prettier-plugin-latex"],
      },
    },
    "Markdown": {
      "format_on_save": "off",
      "use_on_type_format": false,
      "remove_trailing_whitespace_on_save": false,
      "allow_rewrap": "anywhere",
      "soft_wrap": "editor_width",
      "completions": {
        "words": "disabled",
      },
      "prettier": {
        "allowed": true,
      },
    },
    "PHP": {
      "language_servers": ["phpactor", "!intelephense", "!phptools", "..."],
      "prettier": {
        "allowed": true,
        "plugins": ["@prettier/plugin-php"],
        "parser": "php",
      },
    },
    "Plain Text": {
      "allow_rewrap": "anywhere",
      "soft_wrap": "editor_width",
      "completions": {
        "words": "disabled",
      },
    },
    "Proto": {
      "language_servers": [
        "buf",
        "!protols",
        "!protobuf-language-server",
        "...",
      ],
    },
    "Python": {
      "code_actions_on_format": {
        "source.organizeImports.ruff": true,
      },
      "formatter": {
        "language_server": {
          "name": "ruff",
        },
      },
      "debuggers": ["Debugpy"],
      "language_servers": [
        "basedpyright",
        "ruff",
        "!ty",
        "!pyrefly",
        "!pyright",
        "!pylsp",
        "...",
      ],
    },
    "Ruby": {
      "language_servers": [
        "solargraph",
        "!ruby-lsp",
        "!rubocop",
        "!sorbet",
        "!steep",
        "!kanayago",
        "...",
      ],
    },
    "Rust": {
      "debuggers": ["CodeLLDB"],
    },
    "SCSS": {
      "prettier": {
        "allowed": true,
      },
    },
    "Starlark": {
      "language_servers": ["starpls", "!buck2-lsp", "..."],
    },
    "Svelte": {
      "language_servers": ["svelte-language-server", "..."],
      "prettier": {
        "allowed": true,
        "plugins": ["prettier-plugin-svelte"],
      },
    },
    "TSX": {
      "language_servers": ["!typescript-language-server", "vtsls", "..."],
      "prettier": {
        "allowed": true,
      },
    },
    "Twig": {
      "prettier": {
        "allowed": true,
      },
    },
    "TypeScript": {
      "language_servers": ["!typescript-language-server", "vtsls", "..."],
      "prettier": {
        "allowed": true,
      },
    },
    "SystemVerilog": {
      "format_on_save": "off",
      "language_servers": ["!slang", "..."],
      "use_on_type_format": false,
    },
    "Vue.js": {
      "language_servers": ["vue-language-server", "vtsls", "..."],
      "prettier": {
        "allowed": true,
      },
    },
    "XML": {
      "prettier": {
        "allowed": true,
        "plugins": ["@prettier/plugin-xml"],
      },
    },
    "YAML": {
      "prettier": {
        "allowed": true,
      },
    },
    "YAML+ERB": {
      "language_servers": ["!ruby-lsp", "..."],
    },
    "Zig": {
      "language_servers": ["zls", "..."],
    },
  },
  // 特定语言模型的设置。
  "language_models": {
    "anthropic": {
      "api_url": "https://api.anthropic.com",
    },
    "bedrock": {},
    "google": {
      "api_url": "https://generativelanguage.googleapis.com",
    },
    "ollama": {
      "api_url": "http://localhost:11434",
    },
    "openai": {
      "api_url": "https://api.openai.com/v1",
    },
    "openai_compatible": {},
    "open_router": {
      "api_url": "https://openrouter.ai/api/v1",
    },
    "lmstudio": {
      "api_url": "http://localhost:1234/api/v0",
    },
    "deepseek": {
      "api_url": "https://api.deepseek.com/v1",
    },
    "mistral": {
      "api_url": "https://api.mistral.ai/v1",
    },
    "vercel": {
      "api_url": "https://api.v0.dev/v1",
    },
    "x_ai": {
      "api_url": "https://api.x.ai/v1",
    },
    "zed.dev": {},
  },
  "session": {
    // 是否在重启时恢复未保存的缓冲区。
    //
    // 如果为 true，则关闭应用程序时不会提示用户保存/丢弃脏文件。
    //
    // 默认值：true
    "restore_unsaved_buffers": true,
    // 是否跳过工作树信任检查。
    // 当受信任时，项目设置会自动同步，
    // 语言和 MCP 服务器会自动下载并启动。
    //
    // 默认值：false
    "trust_all_worktrees": false,
  },
  // Zed 的 Prettier 集成设置。
  // 允许启用/禁用 Prettier 格式化
  // 并配置默认的 Prettier，在找不到项目级 Prettier 安装时使用。
  "prettier": {
    // 启用或禁用使用 Prettier 格式化任何给定语言。
    "allowed": false,
    // 强制 Prettier 集成在格式化具有该语言的文件时使用特定的解析器名称。
    "plugins": [],
    // 默认 Prettier 选项，格式与 package.json 中 Prettier 的格式相同。
    // 如果项目通过其 package.json 安装了 Prettier，则将忽略这些选项。
    // "trailingComma": "es5",
    // "tabWidth": 4,
    // "semi": false,
    // "singleQuote": true
    // 当设置为非空字符串时，强制 Prettier 集成在格式化具有该语言的文件时使用特定的解析器名称。
    "parser": "",
  },
  // 自动关闭 JSX 标签的设置。
  "jsx_tag_auto_close": {
    "enabled": true,
  },
  // LSP 特定设置。
  "lsp": {
    // 在此处将 LSP 名称指定为键。
    // "rust-analyzer": {
    //     // rust-analyzer 集成的特殊标志，使用服务器提供的任务
    //     enable_lsp_tasks": true,
    //     // 这些初始化选项与 Zed 的默认值合并
    //     "initialization_options": {
    //         "check": {
    //             "command": "clippy" // rust-analyzer.check.command (default: "check")
    //         }
    //     }
    // }
  },
  // DAP 特定设置。
  "dap": {
    // 在此处将 DAP 名称指定为键。
    "CodeLLDB": {
      "env": {
        "RUST_LOG": "info",
      },
    },
  },
  // 通用语言服务器设置。
  "global_lsp_settings": {
    // 是否在状态栏中显示 LSP 服务器按钮。
    "button": true,
    // 等待语言服务器响应的最长时间（秒）。
    // 值为 0 表示不应用超时。
    //
    // 默认值：120
    "request_timeout": 120,
    "notifications": {
      // 自动关闭语言服务器通知的超时时间（毫秒）。
      // 设置为 0 以禁用自动关闭。
      "dismiss_timeout_ms": 5000,
    },
    // 高亮显示语义标记的规则。用户定义的规则将添加到默认规则（可通过“显示默认语义标记规则”查看）之前，因此优先。
    //
    // 每个 `rule` 具有以下属性：
    // - `token_type`：要自定义的 LSP 语义标记类型。如果省略，则该规则匹配所有标记类型。
    // - `token_modifiers`：要匹配的 LSP 语义标记修饰符列表。必须存在所有修饰符才能匹配。
    // - `style`：要使用的当前语法主题中的样式列表。找到的第一个样式将被使用。下面的任何设置都将覆盖该样式。
    // - `foreground_color`：用于标记类型的前景色，十六进制格式（例如 "#ff0000"）。
    // - `background_color`：用于标记类型的背景色，十六进制格式。
    // - `underline`：布尔值或用于添加下划线的颜色，十六进制格式。如果为 `true`，则标记将使用文本颜色添加下划线。
    // - `strikethrough`：布尔值或用于添加删除线的颜色，十六进制格式。如果为 `true`，则标记将使用文本颜色添加删除线。
    // - `font_weight`：可以是 "normal"、"bold"。
    // - `font_style`：可以是 "normal"、"italic"。
    //
    // 第一个匹配的规则将应用于标记。由于用户定义的规则添加在默认规则之前，因此可以通过添加一个匹配该标记的空规则来完全禁用它。
    //
    // 示例：将未解析的引用高亮显示为红色和粗体：
    // "semantic_token_rules": [
    //   {
    //     "token_type": "unresolvedReference",
    //     "foreground_color": "#c93f3f",
    //     "font_weight": "bold"
    //   }
    // ]
    //
    // 默认规则可通过 "zed: show default semantic token rules" 操作查看。
    "semantic_token_rules": [],
  },
  // Jupyter 设置
  "jupyter": {
    "enabled": true,
    "kernel_selections": {},
    // 将语言名称指定为键，将内核名称指定为值。
    // "kernel_selections": {
    //    "python": "conda-base"
    //    "typescript": "deno"
    // }
  },
  // REPL 设置。
  "repl": {
    // 在 REPL 的回滚缓冲区中保留的最大列数。
    // 限制在 [20, 512] 范围内。
    "max_columns": 128,
    // 在 REPL 的回滚缓冲区中保留的最大行数。
    // 限制在 [4, 256] 范围内。
    "max_lines": 32,
    // 滚动之前显示的最大输出行数。
    // 设置为 0 以禁用输出高度限制。
    "output_max_height_lines": 0,
    // 缩放图像之前显示的最大输出列数。
    // 设置为 0 以禁用输出宽度限制。
    "output_max_width_columns": 0,
  },
  // Vim 设置
  "vim": {
    "default_mode": "normal",
    "toggle_relative_line_numbers": false,
    "use_system_clipboard": "always",
    "use_smartcase_find": false,
    "gdefault": false,
    "highlight_on_yank_duration": 200,
    "custom_digraphs": {},
    // 每种模式的光标形状。
    // 形状可以是以下之一："block", "bar", "underline", "hollow".
    "cursor_shape": {
      "normal": "block",
      "replace": "underline",
      "visual": "block",
      // 设置为 "inherit" 以使用编辑器的 cursor_shape。
      "insert": "inherit",
    },
  },
  // 按键提示弹出窗口设置
  "which_key": {
    // 按住组合键时是否显示按键提示弹出窗口。
    "enabled": false,
    // 显示按键提示弹出窗口前的延迟（毫秒）。
    "delay_ms": 1000,
  },
  // 要连接的服务器。如果设置了环境变量 ZED_SERVER_URL，它将覆盖此设置。
  "server_url": "https://zed.dev",
  // 使用 Zed Preview 时要使用的设置覆盖。
  // 主要用于管理多个 Zed 实例的开发人员。
  "preview": {
    // "theme": "Andromeda"
  },
  // 使用 Zed Nightly 时要使用的设置覆盖。
  // 主要用于管理多个 Zed 实例的开发人员。
  "nightly": {
    // "theme": "Andromeda"
  },
  // 使用 Zed Stable 时要使用的设置覆盖。
  // 主要用于管理多个 Zed 实例的开发人员。
  "stable": {
    // "theme": "Andromeda"
  },
  // 使用 Zed Dev 时要使用的设置覆盖。
  // 主要用于管理多个 Zed 实例的开发人员。
  "dev": {
    // "theme": "Andromeda"
  },
  // 在 Linux 上使用的设置覆盖。
  "linux": {},
  // 在 macOS 上使用的设置覆盖。
  "macos": {},
  // 在 Windows 上使用的设置覆盖。
  "windows": {
    "languages": {
      "PHP": {
        "language_servers": ["intelephense", "!phpactor", "!phptools", "..."],
      },
    },
  },
  // 是否在行指示器中显示完整标签或简短标签。
  //
  // 值：
  //   - `short`: "2 s, 15 l, 32 c"
  //   - `long`: "2 selections, 15 lines, 32 characters"
  // 默认值：long
  "line_indicator_format": "long",
  // 设置要使用的代理。代理协议由 URI 方案指定。
  //
  // 支持的 URI 方案：`http`、`https`、`socks4`、`socks4a`、`socks5`、
  // `socks5h`。如果未指定方案，将使用 `http`。
  //
  // 默认情况下不使用代理，或者 Zed 会尝试从环境变量中获取代理设置。如果某些主机不应使用代理，请设置 `no_proxy` 环境变量并提供逗号分隔的列表。
  //
  // 示例：
  //   - "proxy": "socks5h://localhost:10808"
  //   - "proxy": "http://127.0.0.1:10809"
  "proxy": "",
  // 设置命令面板的别名。
  // 当键入的查询是该对象的键时，将使用对应的值。
  //
  // 示例：
  // {
  //   "W": "workspace::Save"
  // }
  "command_aliases": {},
  // ssh_connections 是一个 SSH 连接数组。
  // 您可以通过命令面板中的 `project: Open Remote` 进行配置。
  // Zed 的 SSH 支持也会从您的 ~/.ssh 中拉取配置。
  // 示例：
  // [
  //   {
  //     "host": "example-box",
  //     // "port": 22, "username": "test", "args": ["-i", "/home/user/.ssh/id_rsa"]
  //     "projects": [
  //       {
  //         "paths": ["/home/user/code/zed"]
  //       }
  //     ]
  //   }
  // ]
  "ssh_connections": [],
  // 是否为 SSH 连接源读取 ~/.ssh/config。
  "read_ssh_config": true,
  // 所有上下文服务器工具调用的默认超时时间（秒）。
  // 各个服务器可以在其配置中覆盖此设置。
  // 示例：
  // "context_servers": {
  //   "my-stdio-server": {
  //     "command": "/path/to/server",
  //     "timeout": 120  // 覆盖：此服务器为 2 分钟
  //   },
  // }
  // 默认值：60
  "context_server_timeout": 60,
  // 配置代理可用的上下文服务器。
  "context_servers": {},
  // 配置代理面板中可用的代理服务器。
  "agent_servers": {},
  "debugger": {
    "stepping_granularity": "line",
    "save_breakpoints": true,
    "timeout": 2000,
    "dock": "bottom",
    "log_dap_communications": true,
    "format_dap_log_messages": true,
    "button": true,
  },
  // 配置任意数量的设置配置文件，当从
  // `settings profile selector: toggle` 中选择时，这些配置文件将临时应用于您现有的用户设置之上。
  // 示例：
  // "profiles": {
  //   "Presenting": {
  //     "agent_ui_font_size": 20.0,
  //     "buffer_font_size": 20.0,
  //     "theme": "One Light",
  //     "ui_font_size": 20.0
  //   },
  //   "Python (ty)": {
  //     "languages": {
  //       "Python": {
  //         "language_servers": ["ty"]
  //       }
  //     }
  //   }
  // }
  "profiles": {},

  // 日志范围到所需日志级别的映射。
  // 用于过滤掉嘈杂的日志或启用更详细的日志记录。
  //
  // 示例：{"log": {"client": "warn"}}
  "log": {},
}
```

# 总结
