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
lastmod: 2026-08-31T03:23:00
categories: 生活指南
---

# 前言

写代码的年头一长，多数人都经历过几轮编辑器迁徙。插件要重装，快捷键要重记，工作流要重搭，折腾一圈下来，真正写代码的时间没剩多少。

Zed 想回应的就是这件事。它的团队来自 Atom，当年 Atom 栽在 Electron 的性能上，这次他们用 Rust 从头写了一套 GPUI 框架，把响应速度放在第一位。实际用下来的体感：几千行的 Rust 文件滚动依然跟手，Java 项目加载时 jdt.ls 在后台静默启动，界面不会卡住。语言能力上，LSP 集成、多光标、语法树感知都在，和 JetBrains 的差距没想象中大。

AI 这块，Zed 走的是 BYOK（Bring Your Own Key）：编辑器不自建订阅服务，模型自己带 key。OpenAI、Anthropic、任何兼容 OpenAI 协议的 API 都可以接，Ollama 本地模型也行。想用 Claude 写代码还是 DeepSeek 做解释，自己决定，只付实际调用的 tokens。我的主用模型是 DeepSeek，配置在后文的 AI 一节。

默认状态的 Zed 已经足够好用，但足够好用和真正顺手之间，隔着一堆细节：Tab 宽度、保存时格式化、语言服务器实现、字体、AI 权限。这些没有标准答案，只有你的习惯说了算。

这篇文章不教你入门。开箱即用的部分，打开编辑器就能体验到。我要拆的是配置逻辑，然后给出一套可以直接拿走的配置，覆盖 Java 和 Rust 两个主力语言。篇幅不短，结构如下：开头是 Zed 默认配置的中文注释版全文，接着是我改过的每一项（修改说明）和按场景的分组解读（逐节拆解），JDTLS 偏好和格式化器两个附件全文，最后是改完的 settings.json。

环境以我自己的机器为基准：Fedora Linux 44（COSMIC Atomic），Zed 1.17.2，Flatpak 安装。macOS 和 Windows 上逻辑一样，路径不同，文中遇到会单独标注。

折腾配置是为了把时间省下来写代码。这篇指南想帮你少走弯路。

# 安装与配置文件

Zed 在 Linux 上有三条安装路线：

- 官方脚本：`curl -f https://zed.dev/install.sh | sh`，装到 `~/.local/zed.app`，二进制软链到 `~/.local/bin/zed`。不需要 root，卸载是 `zed --uninstall`。
- 发行版仓库：Fedora 可以走 Terra 源（社区打包，包名 `zed`）。
- Flatpak：`flatpak install flathub dev.zed.Zed`。Flathub 上的包由社区维护，版本可能比官方慢一点。

我选了 Flatpak，原因和编辑器无关：这台机器是 Fedora Atomic（不可变系统），我不想为一个编辑器往系统层叠包。Flatpak 版有两个代价：配置目录换了地方；应用跑在沙箱里，语言服务器和终端看到的是沙箱文件系统。这两点后面单独讲。

配置文件的位置：

- Linux（原生安装）和 macOS：`~/.config/zed/`
- Linux（Flatpak）：`~/.var/app/dev.zed.Zed/config/zed/`
- Windows：`%APPDATA%\Zed\`

目录里常打交道的只有几个文件：

- `settings.json`：编辑器行为，本文的主体。JSONC 格式，可以写注释和尾逗号；改完保存立即生效，不用重启。
- `keymap.json`：键位覆盖，在默认键位之上做增减。
- `tasks.json`：自定义任务（编译、运行、测试）。
- `AGENTS.md`：全局规则文件，Zed 的 Agent 会话会把它注入上下文；项目级规则放在仓库根的 `AGENTS.md`、`CLAUDE.md` 或 `.rules`。
- `themes/`：自定义主题。

`settings.json` 只写和默认值不同的项，Zed 会把你的配置与默认配置合并。想查完整默认值，在命令面板（Ctrl+Shift+P）里执行 “zed: open default settings”，会打开一份只读的 default.json。本文附了中文注释版全文（见「Zed 默认配置全文」），我改完后的 settings.json 全文在文末「完整配置：settings.json」。

# Zed 默认配置全文

下面是 Zed 默认设置的完整中文注释版，对应命令面板里 “zed: open default settings” 打开的 default.json。你的 settings.json 只需要写和默认值不同的项：Zed 启动时把两份配置合并，settings.json 覆盖同名字段。本文改动的每一项与默认值的对照见「修改说明」。

```jsonc
{
  "$schema": "zed://schemas/settings",
  // 用于 UI 的 Zed 主题名称。
  //
  // `mode` 可以是以下之一：
  // - "system"：使用与系统外观相对应的主题
  // - "light"：使用 "light" 字段指定的主题
  // - "dark"：使用 "dark" 字段指定的主题
  "theme": {
    "mode": "system",
    "light": "One Light",
    "dark": "One Dark",
  },
  "icon_theme": "Zed (Default)",
  // 要使用的基础键位映射集名称。
  // 此设置可以取以下值：
  //
  // 1. "Zed"
  // 2. "VSCode"
  // 3. "Atom"
  // 4. "JetBrains"
  // 5. "SublimeText"
  // 6. "TextMate"
  // 7. "Emacs"
  // 8. "Cursor"
  // 9. "None"
  "base_keymap": "Zed",
  // 用于在编辑器中渲染文本的字体名称
  // ".ZedMono" 目前是 Lilex 的别名
  // 但将来可能会改变。
  "buffer_font_family": ".ZedMono",
  // 设置缓冲区文本的字体后备，将与平台默认后备合并。
  "buffer_font_fallbacks": null,
  // 为编辑器中的文本启用的 OpenType 特性。
  "buffer_font_features": {
    // 禁用连字：
    // "calt": false
  },
  // 编辑器中文本的默认字体大小
  "buffer_font_size": 15,
  // 编辑器字体的粗细，标准 CSS 单位，从 100 到 900。
  "buffer_font_weight": 400,
  // 设置缓冲区的行高。
  // 可以取 3 个值：
  //  1. 使用适合阅读的舒适行高 (1.618)
  //         "buffer_line_height": "comfortable"
  //  2. 使用标准行高 (1.3)
  //         "buffer_line_height": "standard",
  //  3. 使用自定义行高
  //         "buffer_line_height": {
  //           "custom": 2
  //         },
  "buffer_line_height": "comfortable",
  // 用于在 UI 中渲染文本的字体名称
  // 你可以将其设置为 ".SystemUIFont" 以使用系统字体
  // ".ZedSans" 目前是 "IBM Plex Sans" 的别名，但这
  // 将来可能会改变
  "ui_font_family": ".ZedSans",
  // 设置 UI 的字体后备，将与平台的默认字体后备合并。
  "ui_font_fallbacks": null,
  // 为 UI 中的文本启用的 OpenType 特性
  "ui_font_features": {
    // 禁用连字：
    "calt": false,
  },
  // UI 字体的粗细，标准 CSS 单位，从 100 到 900。
  "ui_font_weight": 400,
  // UI 中文本的默认字体大小
  "ui_font_size": 16,
  // 代理面板中代理回复的字体族。如果未设置，则回退到 UI 字体族。
  "agent_ui_font_family": null,
  // 代理面板中代理回复的默认字体大小。如果未设置，则回退到 UI 字体大小。
  "agent_ui_font_size": null,
  // 代理面板中用户消息的字体族。如果未设置，则回退到缓冲区字体族。
  "agent_buffer_font_family": null,
  // 代理面板中用户消息的默认字体大小。
  "agent_buffer_font_size": 12,
  // git 面板和提交模态框中提交编辑器的默认字体大小。
  "git_commit_buffer_font_size": 12,
  // Markdown 预览的默认字体大小。如果未设置，则回退到 UI 字体大小。
  "markdown_preview_font_size": null,
  // Markdown 预览的字体族。如果未设置，则回退到 UI 字体族。
  "markdown_preview_font_family": null,
  // Markdown 预览中代码块的字体族。如果未设置，则回退到编辑器字体族。
  "markdown_preview_code_font_family": null,
  // 未使用代码的淡化程度。
  "unnecessary_code_fade": 0.3,
  // 活动窗格样式设置。
  "active_pane_modifiers": {
    // 活动窗格的内嵌边框大小，以像素为单位。
    "border_size": 0.0,
    // 非活动窗格的不透明度。0 表示透明，1 表示不透明。
    // 值被限制在 [0.0, 1.0] 范围内。
    "inactive_opacity": 1.0,
  },
  // 底部停靠区的布局模式。默认为 "contained"
  //   可选：contained, full, left_aligned, right_aligned
  "bottom_dock_layout": "contained",
  // 水平拆分窗格的方向。默认为 "down"
  "pane_split_direction_horizontal": "down",
  // 垂直拆分窗格的方向。默认为 "right"
  "pane_split_direction_vertical": "right",
  // 居中布局相关设置。
  "centered_layout": {
    // 使用居中布局时，中央窗格左侧内边距相对于工作区的宽度比例。
    "left_padding": 0.2,
    // 使用居中布局时，中央窗格右侧内边距相对于工作区的宽度比例。
    "right_padding": 0.2,
  },
  // 图像查看器设置
  "image_viewer": {
    // 图像文件大小的单位："binary"（KiB、MiB）或十进制（KB、MB）
    "unit": "binary",
  },
  // Markdown 预览设置
  "markdown_preview": {
    // 是否限制渲染的 Markdown 内容的宽度。启用后，
    // 内容被限制在 `max_width` 内，并在预览窗格中水平居中。
    "limit_content_width": true,
    // 启用 limit_content_width 时，渲染的 Markdown 内容的最大宽度（像素）。
    "max_width": 800,
  },
  // 确定用鼠标添加多个光标时使用的修饰键。打开悬停链接的鼠标手势
  // 将进行调整，以避免与多光标修饰键冲突。
  //
  // 1. 在 Linux 和 Windows 上映射到 `Alt`，在 macOS 上映射到 `Option`：
  //    "alt"
  // 2. 在 Linux 和 Windows 上映射到 `Control`，在 macOS 上映射到 `Command`：
  //    "cmd_or_ctrl"（别名："cmd", "ctrl"）
  "multi_cursor_modifier": "alt",
  // 是否启用 Vim 模式和键位绑定。
  "vim_mode": false,
  // 是否启用 Helix 模式和键位绑定。
  // 启用此模式将自动启用 Vim 模式。
  "helix_mode": false,
  // 当鼠标悬停在编辑器中的符号上时，是否显示信息悬停框。
  "hover_popover_enabled": true,
  // 显示信息悬停框之前等待的毫秒数。
  // 此延迟也适用于启用 `auto_signature_help` 时的自动签名帮助。
  "hover_popover_delay": 300,
  // 当鼠标朝悬停框移动时，悬停框是否保持显示，
  // 允许在其消失之前与内容交互。
  "hover_popover_sticky": true,
  // 鼠标移开悬停目标后，隐藏悬停框之前等待的毫秒数。
  // 仅在启用 `hover_popover_sticky` 时适用。
  "hover_popover_hiding_delay": 300,
  // 退出 Zed 前是否确认。
  "confirm_quit": false,
  // 是否在打开新的 Zed 实例时恢复上次关闭的项目
  // 可以取 3 个值：
  //  1. 上次会话期间打开的所有工作区
  //         "restore_on_startup": "last_session"
  //  2. 打开的工作区
  //         "restore_on_startup": "last_workspace",
  //  3. 不恢复以前的工作区
  //         "restore_on_startup": "none",
  "restore_on_startup": "last_session",
  // 从 CLI 打开路径且没有显式 `-e`（现有窗口）或 `-n`（新窗口）
  // 标志时的默认行为。
  //
  // 可以取 2 个值：
  //  1. 在当前 Zed 窗口的侧边栏中将目录作为新工作区打开
  //         "cli_default_open_behavior": "existing_window"
  //  2. 在新窗口中打开路径，除非它们是现有项目的子路径
  //         "cli_default_open_behavior": "new_window"
  "cli_default_open_behavior": "existing_window",
  // 从 UI 打开项目时的默认行为。
  //
  // 可以取 2 个值：
  //  1. 在当前 Zed 窗口的侧边栏中将项目作为新工作区打开
  //         "default_open_behavior": "existing_window"
  //  2. 在新窗口中打开项目
  //         "default_open_behavior": "new_window"
  "default_open_behavior": "existing_window",
  // 再次打开文件时是否尝试恢复之前的状态。
  // 状态按窗格存储。
  // 禁用时，将应用默认值而不是状态恢复。
  //
  // 例如，对于编辑器，如果同一文件被关闭，之后又在同一窗格中打开，
  // 则会恢复选区、折叠和滚动位置。
  // 禁用时，默认使用文件开头处的单个选区、零滚动位置和无折叠状态。
  //
  // 默认值：true
  "restore_on_file_reopen": true,
  // 是否自动关闭磁盘上已删除的文件。
  "close_on_file_delete": false,
  // 切换面板（例如使用其键盘快捷键）时，如果面板已聚焦，
  // 是否也关闭面板，而不是仅将焦点移回编辑器。
  "close_panel_on_toggle": false,
  // 编辑器中放置目标的相对大小，放置文件将作为拆分窗格打开（0-0.5）
  // 例如 0.25 == 如果将文件放置到窗格顶部/底部四分之一处，将使用新的垂直拆分
  //              如果将文件放置到窗格左/右四分之一处，将使用新的水平拆分
  "drop_target_size": 0.2,
  // 在没有标签的窗口上使用“关闭活动项”时是否应关闭窗口。
  // 可以取 3 个值：
  //  1. 使用当前平台的约定
  //         "when_closing_with_no_tabs": "platform_default"
  //  2. 总是关闭窗口：
  //         "when_closing_with_no_tabs": "close_window",
  //  3. 从不关闭窗口
  //         "when_closing_with_no_tabs": "keep_window_open",
  "when_closing_with_no_tabs": "platform_default",
  // 关闭最后一个窗口时的操作。
  // 可以取 2 个值：
  //  1. 使用当前平台的约定
  //         "on_last_window_closed": "platform_default"
  //  2. 总是退出应用程序
  //         "on_last_window_closed": "quit_app",
  "on_last_window_closed": "platform_default",
  // 要使用的文本渲染模式。
  // 可以取 3 个值：
  //  1. 使用平台默认行为：
  //         "text_rendering_mode": "platform_default"
  //  2. 使用亚像素（ClearType 风格）文本渲染：
  //         "text_rendering_mode": "subpixel"
  //  3. 使用灰度文本渲染：
  //         "text_rendering_mode": "grayscale"
  "text_rendering_mode": "platform_default",
  // 是否为放大的面板显示内边距。
  // 启用后，放大的中央面板（例如代码编辑器）四周都会有内边距，
  // 而放大的底部/左侧/右侧面板将分别有顶部/右侧/左侧内边距。
  //
  // 默认值：true
  "zoomed_padding": true,
  // 在 Linux 上，什么来绘制 Zed 的窗口装饰（标题栏）。
  // 此设置在其他平台上没有效果：
  // 1. Zed 绘制自己的窗口装饰。
  //    "client"
  // 2. 窗口管理器或合成器绘制窗口装饰。
  //    GNOME Wayland 不支持。
  //    "server"
  //
  // 更改此设置需要重新启动 Zed 才能生效。
  //
  // 默认值："client"
  "window_decorations": "client",
  // 是否针对辅助技术（如屏幕阅读器）优化 Zed 的界面。
  "accessible_mode": false,
  // 对于“打开”和“另存为”是否使用系统提供的对话框。
  // 设置为 false 时，Zed 将使用内置的键盘优先选择器。
  "use_system_path_prompts": true,
  // 对于提示（如确认提示）是否使用系统提供的对话框。
  // 设置为 false 时，Zed 将使用其内置提示。请注意，在 Linux 上，
  // 此选项被忽略，Zed 将始终使用内置提示。
  "use_system_prompts": true,
  // 编辑器中的光标是否闪烁。
  "cursor_blink": true,
  // 默认编辑器的光标形状。
  //  1. 竖线
  //     "bar"
  //  2. 包围下一个字符的方块
  //     "block"
  //  3. 沿下一个字符的下划线
  //     "underline"
  //  4. 在下一个字符周围绘制的方框
  //     "hollow"
  //
  // 默认值："bar"
  "cursor_shape": "bar",
  // 确定响应键盘输入时何时隐藏鼠标光标。
  //
  // 1. 从不隐藏鼠标光标：
  //    "never"
  // 2. 仅在输入时隐藏：
  //    "on_typing"
  // 3. 在输入和解析为动作的键绑定时隐藏：
  //    "on_typing_and_action"
  "hide_mouse": "on_typing_and_action",
  // 是否减少 UI 中的非必要动效，例如加载旋转器和脉动标签，
  // 通过以静态状态渲染它们。
  //
  // 可以取 2 个值：
  // 1. 总是减少动效：
  //    "on"
  // 2. 从不减少动效：
  //    "off"
  "reduce_motion": "off",
  // 确定聚焦的面板是否跟随鼠标位置。
  "focus_follows_mouse": {
    "enabled": false,
    "debounce_ms": 250,
  },
  // 确定代码片段相对于其他补全项的排序方式。
  //
  // 1. 将代码片段放在补全列表顶部：
  //    "top"
  // 2. 正常放置代码片段，无任何偏好：
  //    "inline"
  // 3. 将代码片段放在补全列表底部：
  //    "bottom"
  // 4. 不在补全列表中显示代码片段：
  //    "none"
  "snippet_sort_order": "inline",
  // 如何高亮编辑器中的当前行。
  //
  // 1. 不高亮当前行：
  //    "none"
  // 2. 高亮行号区域：
  //    "gutter"
  // 3. 高亮编辑器区域：
  //    "line"
  // 4. 高亮整行（默认）：
  //    "all"
  "current_line_highlight": "all",
  // 是否高亮编辑器中选中文本的所有出现位置。
  "selection_highlight": true,
  // 文本选区是否应具有圆角。
  "rounded_selection": true,
  // 根据当前光标位置从语言服务器查询高亮之前的防抖延迟。
  "lsp_highlight_debounce": 75,
  // 前景色和背景色之间的最小 APCA 感知对比度。
  // APCA（可访问感知对比度算法）比 WCAG 2.x 更准确，
  // 尤其在深色模式下。值范围从 0 到 106。
  //
  // 基于 APCA 可读性标准（ARC）青铜简单模式：
  // https://readtech.org/ARC/tests/bronze-simple-mode/
  // - 0：无对比度调整
  // - 45：大型流畅文本（36px+）的最低要求
  // - 60：其他内容文本的最低要求
  // - 75：正文文本的最低要求
  // - 90：正文文本的推荐值
  //
  // 这仅影响编辑器中绘制在高亮背景上的文本。
  "minimum_contrast_for_highlights": 45,
  // 在编辑器中输入时是否自动弹出补全菜单，而无需显式请求。
  "show_completions_on_input": true,
  // 是否在补全菜单中显示项目的内联和旁边文档。
  "show_completion_documentation": true,
  // 是否为编辑器中的括号着色。
  // （也称为“彩虹括号”）
  //
  // 不同缩进级别使用的颜色在主题中定义（主题键：`accents`）。
  // 可以通过主题覆盖进行自定义。
  "colorize_brackets": false,
  // 何时在补全菜单中显示滚动条。
  // 此设置可以取四个值：
  //
  // 1. 如果有重要信息或遵循系统配置的行为，则显示滚动条
  //   "auto"
  // 2. 匹配系统配置的行为：
  //    "system"
  // 3. 总是显示滚动条：
  //    "always"
  // 4. 从不显示滚动条：
  //    "never"（默认）
  "completion_menu_scrollbar": "never",
  // 代码补全上下文菜单中的详细信息文本是左对齐还是右对齐。
  "completion_detail_alignment": "left",
  // 如何显示补全菜单中每个条目的 LSP 项目类型（函数、方法、变量等）。
  //
  // 1. 不显示项目类型：
  //   "off"（默认）
  // 2. 显示单字母徽章，根据活动语法主题着色：
  //   "symbol"
  "completion_menu_item_kind": "off",
  // 如何在编辑器中显示差异。
  //
  // 默认：split
  "diff_view_style": "split",
  // 使用拆分差异视图的最小宽度（以 em 宽度为单位）。
  // 当编辑器比这更窄时，差异视图会自动切换到统一模式，
  // 当编辑器足够宽时再切换回来。设置为 0 可禁用自动切换。
  //
  // 默认：100
  "minimum_split_diff_width": 100,
  // 在括号内时，在编辑器中显示方法签名。
  "auto_signature_help": false,
  // 是否在补全或插入括号对后显示签名帮助。
  // 如果启用了 `auto_signature_help`，则此设置也被视为启用。
  "show_signature_help_after_edits": false,
  // 是否在缓冲区行首显示代码操作按钮。
  "inline_code_actions": true,
  // 是否允许在缓冲区中拖放文本选区。
  "drag_and_drop_selection": {
    // 为 true 时，启用缓冲区中的文本选区拖放。
    "enabled": true,
    // 允许拖放之前必须经过的延迟（毫秒）。否则，将创建新的文本选区。
    "delay": 300,
  },
  // 是否以及如何显示来自语言服务器的代码镜头。
  //
  // 可能的值：
  //
  // 1. 不显示代码镜头。
  //      "code_lens": "off",
  // 2. 在代码元素上方显示来自语言服务器的代码镜头。
  //      "code_lens": "on",
  // 3. 在代码操作菜单中显示代码镜头。
  //      "code_lens": "menu",
  "code_lens": "off",
  // 当转到定义没有结果时执行的操作。
  //
  // 1. 不执行任何操作：`none`
  // 2. 查找相同符号的引用：`find_all_references`（默认）
  "go_to_definition_fallback": "find_all_references",
  // 导航到定义或引用时如何将目标滚动到视图中
  // （例如：转到定义、转到类型定义、查找所有引用）。
  //
  // 1. 在视口中垂直居中目标：`center`（默认）
  // 2. 滚动最小量以使目标可见：`minimum`
  // 3. 滚动使目标出现在视口顶部附近：`top`
  // 4. 保持光标在视口中的垂直位置，当光标在屏幕外时回退到 `center`：`preserve`
  "go_to_definition_scroll_strategy": "center",
  // 在哪里显示可能包含多个位置的 LSP 结果
  // （转到定义、转到实现、查找所有引用）。单个结果
  // 总是直接打开。各个操作可以通过其 `open_results_in` 参数覆盖此设置。
  //
  // 1. 在多缓冲区中打开结果：`multi_buffer`（默认）
  // 2. 在可过滤的选择器中打开结果：`picker`
  "lsp_results_location": "multi_buffer",
  // 用于过滤编辑器中显示的诊断信息的级别。
  //
  // 仅影响编辑器渲染，不会中断诊断获取和项目诊断编辑器的功能。
  // 哪些包含诊断错误/警告的文件在标签中标记。
  // 仅当文件图标也激活时才显示诊断。
  // 此设置仅在可以取以下三个值时有效：
  //
  // 滚动条中显示哪些诊断指示器，其级别应大于或等于指定的严重性级别。
  // 可能的值：
  //  - "off" — 不允许任何诊断
  //  - "error"
  //  - "warning"
  //  - "info"
  //  - "hint"
  //  - "all" — 允许所有诊断（默认）
  "diagnostics_max_severity": "all",
  // 是否在编辑器中显示换行参考线（垂直标尺）。
  // 设置为 true 时，如果 'soft_wrap' 设置为 'preferred_line_length'，
  // 将在 'preferred_line_length' 值处显示参考线，并将显示
  // 由 'wrap_guides' 设置指定的任何额外参考线。
  "show_wrap_guides": true,
  // 在编辑器中显示换行参考线的字符数。
  "wrap_guides": [],
  // 在私有文件中隐藏变量值的视觉显示
  "redact_private_values": false,
  // 在多缓冲区中默认展开摘录的行数。
  "expand_excerpt_lines": 5,
  // 多缓冲区摘录中默认显示的上下文行数。
  "excerpt_context_lines": 2,
  // 用于匹配文件路径以确定文件是否为私有的 glob 模式。
  "private_files": ["**/.env*", "**/*.pem", "**/*.key", "**/*.cert", "**/*.crt", "**/secrets.yml"],
  // 是否使用额外的 LSP 查询在每次输入由 LSP 服务器能力定义的“触发”符号后进行格式化（并修改）代码。
  "use_on_type_format": true,
  // 输入开括号、括号、花括号、单引号或双引号字符时，是否自动添加匹配的闭合字符。
  // 例如，当你输入 '(' 时，Zed 会在正确位置添加一个闭合的 ')'。
  "use_autoclose": true,
  // 选中文本后输入开括号、括号、花括号、单引号或双引号字符时，
  // 是否自动用这些字符包围选中的文本。
  // 例如，当你选中文本并输入 '(' 时，Zed 会用 () 包围文本。
  "use_auto_surround": true,
  // 控制输入时的自动缩进行为。
  // - "syntax_aware"：根据语法上下文调整缩进（默认）
  // - "preserve_indent"：在新行上保留当前行的缩进
  // - "none"：无自动缩进
  "auto_indent": "syntax_aware",
  // 是否根据上下文调整粘贴内容的缩进。
  "auto_indent_on_paste": true,
  // 控制编辑器如何处理自动闭合的字符。
  // 设置为 `false`（默认）时，跳过和自动移除闭合字符
  // 仅对自动插入的字符发生。
  // 否则（当 `true` 时），无论闭合字符如何插入，
  // 总是跳过并自动移除它们。
  "always_treat_brackets_as_autoclosed": false,
  // 控制 `editor::Rewrap` 操作在当前语言作用域中允许的位置。
  //
  // 此设置可以取三个值：
  //
  // 1. 仅在注释中允许重排：
  //    "in_comments"
  // 2. 仅在当前选区内允许重排：
  //    "in_selections"
  // 3. 允许在任何地方重排：
  //    "anywhere"
  //
  // 使用除 `in_comments` 以外的值时，重排可能会产生语法无效的代码。
  // 选择所需行为时请记住这一点。
  //
  // 注意：此设置在 Vim 模式下无效，因为重排已在所有地方允许。
  "allow_rewrap": "in_comments",
  // 控制编辑预测是立即显示（true），
  // 还是手动通过触发 `editor::ShowEditPrediction`（false）。
  "show_edit_predictions": true,
  // 控制在给定语言作用域中是否显示编辑预测。
  // 示例：["string", "comment"]
  "edit_predictions_disabled_in": [],
  // 是否在编辑器中显示制表符和空格。
  // 此设置可以取四个值：
  //
  // 1. 仅为选中的文本绘制制表符和空格（默认）：
  //    "selection"
  // 2. 不绘制任何制表符或空格：
  //    "none"
  // 3. 绘制所有不可见符号：
  //    "all"
  // 4. 仅在边界处绘制空白：
  //    "boundary"
  // 5. 仅在非空白字符后绘制空白：
  //    "trailing"
  // 对于空白字符位于边界，需要满足以下任一条件：
  // - 它是制表符
  // - 它与边缘（开始或结束）相邻
  // - 它与空白字符（左侧或右侧）相邻
  "show_whitespaces": "selection",
  // 启用 show_whitespaces 时用于渲染空白字符的可见字符。
  "whitespace_map": {
    "space": "•",
    "tab": "→",
  },
  // Zed 中与通话相关的设置
  "calls": {
    // 加入通话时默认开启麦克风
    "mute_on_join": false,
    // 当你第一个加入频道时共享你的项目
    "share_on_join": false,
  },
  // 工具栏相关设置
  "toolbar": {
    // 是否显示面包屑。
    "breadcrumbs": true,
    // 是否显示快捷操作按钮。
    "quick_actions": true,
    // 是否在编辑器工具栏中显示“选择”菜单。
    "selections_menu": true,
    // 是否在编辑器工具栏中显示代理审查按钮。
    "agent_review": true,
    // 是否在编辑器工具栏中显示代码操作按钮。
    "code_actions": false,
  },
  // 是否允许窗口根据用户的标签偏好组合在一起（仅限 macOS）。
  "use_system_window_tabs": false,
  // `zed::ToggleFullScreen` 操作进入的全屏模式（仅限 macOS）。
  //
  // 1. 使用 macOS 的原生全屏，将窗口移动到其自己的 Mission Control 空间。
  //     "native"
  // 2. 调整窗口大小以覆盖整个屏幕，包括菜单栏，
  //    在刘海屏上，包括刘海周围的区域。
  //     "simple"
  "fullscreen_mode": "native",
  // 标题栏相关设置
  "title_bar": {
    // 是否在标题栏的分支图标上显示 git 状态指示器。
    "show_branch_status_icon": false,
    // 是否在标题栏中显示分支名称按钮。
    "show_branch_name": true,
    // 是否在标题栏中显示工作树名称按钮。
    "show_worktree_name": true,
    // 是否在标题栏中显示项目主机和名称。
    "show_project_items": true,
    // 是否在标题栏中显示引导横幅。
    "show_onboarding_banner": true,
    // 是否在标题栏中显示用户头像。
    "show_user_picture": true,
    // 是否在标题栏中显示用户菜单。
    "show_user_menu": true,
    // 是否在标题栏中显示登录按钮。
    "show_sign_in": true,
    // 是否在标题栏中显示菜单。
    "show_menus": false,
    // 标题栏中窗口控制按钮的布局（仅限 Linux）。
    "button_layout": "platform_default",
  },
  "audio": {
    // 选择特定的输出音频设备。
    // `null` 表示使用系统默认值。
    // 任何无法识别的输出设备将回退到系统默认值。
    "experimental.output_audio_device": null,
    // 选择特定的输入音频设备。
    // `null` 表示使用系统默认值。
    // 任何无法识别的输入设备将回退到系统默认值。
    "experimental.input_audio_device": null,
  },
  // 滚动条相关设置
  "scrollbar": {
    // 何时在编辑器中显示滚动条。
    // 此设置可以取四个值：
    //
    // 1. 如果有重要信息或遵循系统配置的行为，则显示滚动条（默认）：
    //   "auto"
    // 2. 匹配系统配置的行为：
    //    "system"
    // 3. 总是显示滚动条：
    //    "always"
    // 4. 从不显示滚动条：
    //    "never"
    "show": "auto",
    // 是否在滚动条中显示光标位置。
    "cursors": true,
    // 是否在滚动条中显示 git 差异指示器。
    "git_diff": true,
    // 是否在滚动条中显示缓冲区搜索结果。
    "search_results": true,
    // 是否在滚动条中显示选中文本的出现位置。
    "selected_text": true,
    // 是否在滚动条中显示选中符号的出现位置。
    "selected_symbol": true,
    // 在滚动条中显示哪些诊断指示器：
    //  - "none" 或 false：不显示诊断
    //  - "error"：仅显示错误
    //  - "warning"：仅显示错误和警告
    //  - "information"：仅显示错误、警告和信息
    //  - "all" 或 true：显示所有诊断
    "diagnostics": "all",
    // 强制启用或禁用每个轴的滚动条
    "axes": {
      // 为 false 时，强制禁用水平滚动条。否则，遵守其他设置。
      "horizontal": true,
      // 为 false 时，强制禁用垂直滚动条。否则，遵守其他设置。
      "vertical": true,
    },
  },
  // 缩略图相关设置
  "minimap": {
    // 何时在编辑器中显示缩略图。
    // 此设置可以取三个值：
    // 1. 如果编辑器的滚动条可见则显示缩略图：
    //    "auto"
    // 2. 总是显示缩略图：
    //    "always"
    // 3. 从不显示缩略图：
    //    "never"（默认）
    "show": "never",
    // 在编辑器的何处显示缩略图。
    // 此设置可以取两个值：
    // 1. 仅在聚焦的编辑器上显示缩略图：
    //    "active_editor"（默认）
    // 2. 在所有打开的编辑器上显示缩略图：
    //    "all_editors"
    "display_in": "active_editor",
    // 何时显示缩略图滑块。
    // 此设置可以取两个值：
    // 1. 当鼠标悬停在缩略图上时显示滑块：
    //    "hover"
    // 2. 总是显示滑块：
    //    "always"（默认）
    "thumb": "always",
    // 缩略图滑块边框的外观。
    // 此设置可以取五个值：
    // 1. 在滑块的所有边上显示边框：
    //    "thumb_border": "full"
    // 2. 除左边外在滑块的所有边上显示边框：
    //    "thumb_border": "left_open"（默认）
    // 3. 除右边外在滑块的所有边上显示边框：
    //    "thumb_border": "right_open"
    // 4. 仅在滑块的左边显示边框：
    //    "thumb_border": "left_only"
    // 5. 不带任何边框显示滑块：
    //    "thumb_border": "none"
    "thumb_border": "left_open",
    // 如何在缩略图中高亮当前行。
    // 此设置可以取以下值：
    //
    // 1. `null` 继承编辑器的 `current_line_highlight` 设置（默认）
    // 2. "line" 或 "all" 在缩略图中高亮当前行。
    // 3. "gutter" 或 "none" 不在缩略图中高亮当前行。
    "current_line_highlight": null,
    // 缩略图中显示的最大列数。
    "max_width_columns": 80,
  },
  // 在 Linux 上启用中键粘贴。
  "middle_click_paste": true,
  // 在多缓冲区中的某些摘录（单例缓冲区的一部分）上双击时的操作。
  // 可以取 2 个值：
  //  1. 行为像普通缓冲区并选择整个单词（默认）。
  //         "double_click_in_multibuffer": "select"
  //  2. 在新标签页中将点击的摘录作为新缓冲区打开。
  //         "double_click_in_multibuffer": "open",
  // 对于 "open" 的情况，可以通过双击时按住 `alt` 来实现常规选择行为。
  "double_click_in_multibuffer": "select",
  "gutter": {
    // 是否在行号区域显示行号。
    "line_numbers": true,
    // 是否在行号区域显示可运行项按钮。
    "runnables": true,
    // 是否在行号区域显示书签。
    "bookmarks": true,
    // 是否在行号区域显示断点。
    "breakpoints": true,
    // 是否在行号区域显示折叠按钮。
    "folds": true,
    // 在行号区域中为行号保留空间的最小字符数。
    "min_line_number_digits": 4,
    // 行号区域中 git 差异块指示器的宽度。
    // 使用 "default" 随字体大小缩放，或使用 {"custom": <像素>} 设置固定宽度。
    "git_gutter_width": "default",
  },
  "indent_guides": {
    // 是否在编辑器中显示缩进参考线。
    "enabled": true,
    // 缩进参考线的宽度（像素），介于 1 到 10 之间。
    "line_width": 1,
    // 活动缩进参考线的宽度（像素），介于 1 到 10 之间。
    "active_line_width": 1,
    // 确定缩进参考线如何着色。
    // 此设置可以取以下三个值：
    //
    // 1. "disabled"
    // 2. "fixed"
    // 3. "indent_aware"
    "coloring": "fixed",
    // 确定缩进参考线背景如何着色。
    // 此设置可以取以下两个值：
    //
    // 1. "disabled"
    // 2. "indent_aware"
    "background_coloring": "disabled",
  },
  // 编辑器是否会滚动到最后一行的后面。
  "scroll_beyond_last_line": "one_page",
  // 使用键盘滚动时光标上方/下方保留的行数
  "vertical_scroll_margin": 3,
  // 点击可见文本区域边缘附近时是否滚动。
  "autoscroll_on_clicks": false,
  // 使用鼠标滚动时两侧保留的字符数
  "horizontal_scroll_margin": 5,
  // 滚动灵敏度乘数。该乘数同时应用于
  // 滚动时的水平和垂直增量值。
  "scroll_sensitivity": 1.0,
  // 是否在按住主要修饰键（macOS 上为 Cmd，其他平台为 Ctrl）时
  // 使用鼠标滚轮缩放编辑器字体大小。
  "mouse_wheel_zoom": false,
  // 快速滚动的滚动灵敏度乘数。该乘数同时应用于
  // 滚动时的水平和垂直增量值。快速滚动发生在用户
  // 滚动时按住 alt 或 option 键。
  "fast_scroll_sensitivity": 4.0,
  "sticky_scroll": {
    // 是否将作用域粘在编辑器顶部。
    "enabled": false,
  },
  "relative_line_numbers": "disabled",
  // 如果 'search_wrap' 被禁用，搜索结果不会在文件末尾回绕。
  "search_wrap": true,
  // 打开新项目和缓冲区搜索时默认启用的搜索选项。
  "search": {
    // 是否在状态栏中显示项目搜索按钮。
    "button": true,
    // 是否仅匹配整个单词。
    "whole_word": false,
    // 是否区分大小写匹配。
    "case_sensitive": false,
    // 是否在搜索结果中包含被 git 忽略的文件。
    "include_ignored": false,
    // 是否将搜索查询解释为正则表达式。
    "regex": false,
    // 导航时是否将光标居中于每个搜索匹配项。
    "center_on_match": false,
  },
  // 何时根据光标下的文本填充新搜索的查询。
  // 此设置可以取以下三个值：
  //
  // 1. 总是用光标下的单词填充搜索查询（默认）。
  //    "always"
  // 2. 仅当有文本选中时填充搜索查询
  //    "selection"
  // 3. 从不填充搜索查询
  //    "never"
  "seed_search_query_from_cursor": "always",
  // 启用后，根据查询自动调整搜索的大小写敏感性。
  // 如果搜索查询包含任何大写字母，则搜索变为区分大小写；
  // 如果仅包含小写字母，则搜索变为不区分大小写。
  "use_smartcase_search": false,
  // 内嵌提示相关设置
  "inlay_hints": {
    // 全局开关，用于打开和关闭提示，默认关闭。
    "enabled": false,
    // 切换某些类型的提示开关，默认全部打开。
    "show_type_hints": true,
    "show_parameter_hints": true,
    "show_value_hints": true,
    // 对应 LSP 提示类型中的 null/None 值。
    "show_other_hints": true,
    // 是否为内嵌提示显示背景。
    //
    // 如果设置为 `true`，背景将使用当前主题中的 `hint.background` 颜色。
    "show_background": false,
    // 编辑缓冲区后等待请求提示的时间，
    // 设置为 0 可禁用防抖。
    "edit_debounce_ms": 700,
    // 滚动缓冲区后等待请求提示的时间，
    // 设置为 0 可禁用防抖。
    "scroll_debounce_ms": 50,
    // 一组修饰键，按下时将切换内嵌提示的可见性。
    // 如果集合为空或未按下所有指定的修饰键，则不会切换内嵌提示。
    "toggle_on_modifiers_press": {
      "control": false,
      "shift": false,
      "alt": false,
      "platform": false,
      "function": false,
    },
  },
  // 调整停靠区大小时是否调整其中所有面板的大小。
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
    "dock": "right",
    // 项目面板中工作树条目之间的间距。可以是 'comfortable' 或 'standard'。
    "entry_spacing": "comfortable",
    // 是否在项目面板中显示文件图标。
    "file_icons": true,
    // 是否在项目面板中为目录显示文件夹图标或箭头。
    "folder_icons": true,
    // 是否在项目面板中显示 git 状态。
    "git_status": true,
    // 嵌套项的缩进量。
    "indent_size": 20,
    // 当相应的项目条目变为活动状态时，是否自动在项目面板中显示它。
    // Gitignore 条目永远不会自动显示。
    "auto_reveal_entries": true,
    // 项目面板是否在启动时打开。
    "starts_open": true,
    // 当目录内只有一个子目录时，是否自动折叠目录并显示紧凑文件夹
    // （例如 "a/b/c"）。
    "auto_fold_dirs": true,
    // 是否在项目面板中以粗体显示文件夹名称。
    "bold_folder_labels": false,
    // 滚动条相关设置
    "scrollbar": {
      // 何时在项目面板中显示滚动条。
      // 此设置可以取五个值：
      //
      // 1. null（默认）：继承编辑器设置
      // 2. 如果有重要信息或遵循系统配置的行为，则显示滚动条（默认）：
      //   "auto"
      // 3. 匹配系统配置的行为：
      //    "system"
      // 4. 总是显示滚动条：
      //    "always"
      // 5. 从不显示滚动条：
      //    "never"
      "show": null,
      // 是否允许在项目面板中水平滚动。
      // 为 false 时，视图锁定在最左侧位置，长文件名被裁剪。
      "horizontal_scroll": true,
    },
    // 在项目面板中标记哪些包含诊断错误/警告的文件。
    // 此设置可以取以下三个值：
    //
    // 1. 不标记任何文件：
    //    "off"
    // 2. 仅标记有错误的文件：
    //    "errors"
    // 3. 标记有错误和警告的文件：
    //    "all"
    "show_diagnostics": "all",
    // 是否将父目录粘在项目面板顶部。
    "sticky_scroll": true,
    // 项目面板中缩进参考线的相关设置。
    "indent_guides": {
      // 何时在项目面板中显示缩进参考线。
      // 此设置可以取两个值：
      //
      // 1. 总是显示缩进参考线：
      //    "always"
      // 2. 从不显示缩进参考线：
      //    "never"
      "show": "always",
    },
    // 项目面板中条目的排序顺序。
    // 此设置可以取三个值：
    //
    // 1. 先显示目录，再显示文件：
    //    "directories_first"
    // 2. 目录和文件混合：
    //    "mixed"
    // 3. 先显示文件，再显示目录：
    //    "files_first"
    "sort_mode": "directories_first",
    // 是否在项目面板中区分大小写地对文件和文件夹名称排序。
    // 此设置可以取四个值：
    //
    // 1. 不区分大小写的自然排序，平局时小写优先（默认）：
    //    "default"
    // 2. 大写名称在小写名称之前分组，
    //    每组内使用不区分大小写的自然排序：
    //    "upper"
    // 3. 小写名称在大写名称之前分组，
    //    每组内使用不区分大小写的自然排序：
    //    "lower"
    // 4. 纯 Unicode 码点比较。
    //    不进行大小写折叠，不进行自然数字排序：
    //    "unicode"
    "sort_order": "default",
    // 是否在项目面板中的文件名旁边显示错误和警告计数徽章。
    "diagnostic_badges": false,
    // 是否在项目面板中的文件名旁边显示 git 状态指示器。
    "git_status_indicator": false,
    // 是否在项目面板中启用拖放操作。
    "drag_and_drop": true,
    // 当窗口中只打开一个文件夹时是否隐藏根条目；
    // 这也影响文件查找器历史中文件路径的显示方式。
    "hide_root": false,
    // 是否在项目面板中隐藏隐藏条目。
    "hide_hidden": false,
    // 自动打开文件的设置。
    "auto_open": {
      // 是否在编辑器中自动打开新建的文件。
      "on_create": true,
      // 是否在粘贴或复制文件后自动打开。
      "on_paste": true,
      // 是否自动打开从外部来源拖放的文件。
      "on_drop": true,
    },
  },
  "outline_panel": {
    // 是否在状态栏中显示大纲面板按钮
    "button": true,
    // 大纲面板的默认宽度。
    "default_width": 300,
    // 大纲面板的停靠位置。可以是 'left' 或 'right'。
    "dock": "right",
    // 是否在大纲面板中显示文件图标。
    "file_icons": true,
    // 是否在大纲面板中为目录显示文件夹图标或箭头。
    "folder_icons": true,
    // 是否在大纲面板中显示 git 状态。
    "git_status": true,
    // 嵌套项的缩进量。
    "indent_size": 20,
    // 当相应的大纲条目变为活动状态时，是否自动在大纲面板中显示它。
    // Gitignore 条目永远不会自动显示。
    "auto_reveal_entries": true,
    // 当目录内只有一个目录时，是否自动折叠目录。
    "auto_fold_dirs": true,
    // 大纲面板中缩进参考线的相关设置。
    "indent_guides": {
      // 何时在大纲面板中显示缩进参考线。
      // 此设置可以取两个值：
      //
      // 1. 总是显示缩进参考线：
      //    "always"
      // 2. 从不显示缩进参考线：
      //    "never"
      "show": "always",
    },
    // 滚动条相关设置
    "scrollbar": {
      // 何时在大纲面板中显示滚动条。
      // 此设置可以取五个值：
      //
      // 1. null（默认）：继承编辑器设置
      // 2. 如果有重要信息或遵循系统配置的行为，则显示滚动条（默认）：
      //   "auto"
      // 3. 匹配系统配置的行为：
      //    "system"
      // 4. 总是显示滚动条：
      //    "always"
      // 5. 从不显示滚动条：
      //    "never"
      "show": null,
    },
    // 当前文件中大纲项的默认展开深度。
    // 设置为 0 折叠所有有子项的项，1 或更高则折叠该深度或更深的项。
    "expand_outlines_with_depth": 100,
  },
  "collaboration_panel": {
    // 是否在状态栏中显示协作面板按钮。
    "button": true,
    // 协作面板的停靠位置。可以是 'left' 或 'right'。
    "dock": "right",
    // 协作面板的默认宽度。
    "default_width": 240,
  },
  "git_panel": {
    // 是否在状态栏中显示 git 面板按钮。
    "button": true,
    // git 面板的停靠位置。可以是 'left' 或 'right'。
    "dock": "right",
    // git 面板的默认宽度。
    "default_width": 360,
    // 面板中 git 状态指示器的样式。
    //
    // 可选：label_color, icon
    // 默认：icon
    "status_style": "icon",
    // 是否在 git 面板中显示文件图标。
    //
    // 默认：false
    "file_icons": false,
    // 是否在 git 面板中为目录显示文件夹图标或箭头。
    //
    // 默认：true
    "folder_icons": true,
    // 如果未设置 `init.defaultBranch` 时使用的分支名称
    //
    // 默认：main
    "fallback_branch_name": "main",
    // 如何排序 git 面板中的条目。
    //
    // 默认：path
    "sort_by": "path",
    // 如何分组 git 面板中的条目。
    //
    // 默认：status
    "group_by": "status",
    // 是否在差异面板中折叠未跟踪文件。
    //
    // 默认：false
    "collapse_untracked_diff": false,
    /// 是否在面板中以树形或平面视图显示条目
    ///
    /// 默认：false
    "tree_view": false,
    // git 面板是否在启动时打开。
    //
    // 默认：false
    "starts_open": false,
    // 是否在 git 面板图标上显示未提交更改计数的徽章。
    //
    // 默认：false
    "show_count_badge": false,
    "scrollbar": {
      // 何时在 git 面板中显示滚动条。
      //
      // 可选：always, auto, never, system
      // 默认：继承编辑器滚动条设置
      // "show": null
    },
    // 是否在 Git 面板中每个文件旁边显示添加/删除更改计数。
    //
    // 默认：true
    "diff_stats": true,
    // 提交消息标题在显示警告之前的最大长度。
    // 设置为 0 禁用。
    //
    // 默认：0
    "commit_title_max_length": 0,
    // 在 Git 面板中点击已更改文件时的默认操作。
    //
    // 可选：project_diff, file_diff, view_file
    // 默认：project_diff
    "entry_primary_click_action": "project_diff",
  },
  "agent": {
    // 行内助手是否在可用时使用流式工具
    "inline_assistant_use_streaming_tools": true,
    // 生成 git 提交消息时，是否在提示中包含项目规则文件
    // （AGENTS.md、CLAUDE.md、.rules 等）。
    "commit_message_include_project_rules": true,
    // 是否启用代理。
    "enabled": true,
    // 是否在状态栏中显示代理面板按钮。
    "button": true,
    // 代理面板的停靠位置。可以是 'left'、'right' 或 'bottom'。
    "dock": "left",
    // 代理面板是否使用灵活（按比例）尺寸。
    //
    // 默认：true
    "flexible": true,
    // 线程侧边栏的位置。可以是 'left' 或 'right'。
    "sidebar_side": "left",
    // 代理面板停靠在左侧或右侧时的默认宽度。
    "default_width": 640,
    // 代理面板停靠在底部时的默认高度。
    "default_height": 320,
    // 是否限制代理面板中的内容宽度。启用后，
    // 当面板更宽时，内容将被限制在 `max_content_width` 内并居中，
    // 以获得最佳可读性。
    "limit_content_width": true,
    // 启用 limit_content_width 时的最大内容宽度（像素）。
    // 内容将在面板内居中。
    "max_content_width": 850,
    // 创建新线程时使用的默认模型。
    "default_model": {
      // 要使用的提供商。
      "provider": "zed.dev",
      // 要使用的模型。
      "model": "claude-sonnet-4",
      // 是否启用思考。
      "enable_thinking": false,
    },
    // 语言模型请求的附加参数。向模型发出请求时，参数将取自
    // 此列表中最后一个与模型提供商和名称匹配的条目。在每个条目中，
    // 提供商和模型都是可选的，因此你可以为其中之一指定参数。
    "model_parameters": [
      // 为所有 OpenAI 模型的请求设置参数：
      // {
      //   "provider": "openai",
      //   "temperature": 0.5
      // }
      //
      // 为所有请求设置参数：
      // {
      //   "temperature": 0
      // }
      //
      // 为特定的提供商和模型设置参数：
      // {
      //   "provider": "zed.dev",
      //   "model": "claude-sonnet-4",
      //   "temperature": 1.0
      // }
    ],
    // 工具操作的权限规则。
    //
    // “default”设置在没有工具特定规则匹配时适用。
    // 对于定义自己权限模式的外部代理，
    // “deny”和“confirm”仍然优先——只有当 Zed 允许该操作时，
    // 才使用外部代理的权限系统。
    //
    // 下面的按工具正则表达式模式（"tools"）匹配工具输入文本
    // （命令、路径、URL 等）。对于 `copy_path` 和 `move_path`，
    // 模式会分别对每个路径（源和目标）进行匹配。
    "tool_permissions": {
      // 当没有工具特定规则匹配时的全局默认权限。
      // "allow" - 自动批准，无需提示
      // "deny" - 自动拒绝
      // "confirm" - 总是提示（默认）
      "default": "confirm",
      // 按工具的权限规则。正则表达式模式匹配工具输入文本。
      // 按工具的 "default" 也适用于 MCP 工具。
      // 每个工具可以有自己的默认值和正则表达式模式。
      "tools": {
        // "terminal": {
        //   "default": "confirm",
        //   "always_confirm": [
        //     // 破坏性 git 操作
        //     { "pattern": "git\\s+(reset|clean)\\s+--hard" },
        //     { "pattern": "git\\s+push\\s+(-f|--force)" },
        //   ],
        // },
        // "edit_file": {
        //   "default": "confirm",
        //   "always_deny": [
        //     // 机密和凭据
        //     { "pattern": "\\.env($|\\.)" },
        //     { "pattern": "secrets?/" },
        //     { "pattern": "\\.pem$" },
        //     { "pattern": "\\.key$" },
        //   ],
        // },
      },
    },
    // 启用后，代理编辑将显示在单文件编辑器中进行审查
    "single_file_review": false,
    // 自动代理上下文压缩的设置，当上下文窗口变得太大时，
    // 总结较早的消息以释放空间。
    "auto_compact": {
      // 是否在接近限制时自动压缩代理的上下文。
      "enabled": true,
      // 自动压缩运行的阈值。以下之一：
      //   - 以 "%" 结尾的百分比字符串（例如 "90%"），相对于模型的上下文窗口测量。
      //     允许小数（例如 "95.5%"）。
      //   - 正整数：使用这么多令牌后压缩
      //     （例如 100000 在使用 100,000 个令牌后压缩）。
      //   - 负整数：一旦上下文窗口中剩余这么多令牌就压缩
      //     （例如 -20000 在剩余少于 20,000 个令牌时压缩）。
      // 0 不是有效的阈值。
      "threshold": "90%",
    },
    // 启用后，显示投票大拇指以获取代理编辑的反馈。
    "enable_feedback": true,
    "default_profile": "write",
    "profiles": {
      "write": {
        "name": "Write",
        "enable_all_context_servers": true,
        "tools": {
          "copy_path": true,
          "create_directory": true,
          "create_thread": true,
          "delete_path": true,
          "diagnostics": true,
          "apply_code_action": true,
          "ask_user": false,
          "edit_file": true,
          "write_file": true,
          "fetch": true,
          "find_path": true,
          "find_references": true,
          "get_code_actions": true,
          "go_to_definition": true,
          "list_agents_and_models": true,
          "list_directory": true,
          "move_path": true,
          "rename_symbol": true,
          "read_file": true,
          "grep": true,
          "skill": true,
          "spawn_agent": true,
          "terminal": true,
          "search_web": true,
        },
      },
      "ask": {
        "name": "Ask",
        // 我们不知道哪些上下文服务器工具对于 "Ask" 配置文件是安全的，因此默认不启用它们。
        // "enable_all_context_servers": true,
        "tools": {
          "create_thread": true,
          "diagnostics": true,
          "ask_user": false,
          "fetch": true,
          "list_agents_and_models": true,
          "list_directory": true,
          "find_path": true,
          "find_references": true,
          "get_code_actions": true,
          "go_to_definition": true,
          "read_file": true,
          "grep": true,
          "skill": true,
          "spawn_agent": true,
          "search_web": true,
        },
      },
      "minimal": {
        "name": "Minimal",
        "enable_all_context_servers": false,
        "tools": {},
      },
    },
    // 当代理完成响应或需要确认才能运行工具操作时，
    // 在哪里显示通知。
    // "primary_screen" - 仅在主屏幕上显示通知（默认）
    // "all_screens" - 在所有屏幕上显示这些通知
    // "never" - 从不显示这些通知
    "notify_when_agent_waiting": "primary_screen",
    // 当代理完成响应或需要用户输入时，何时播放声音。
    // "never" - 从不播放声音
    // "when_hidden" - 仅在代理面板不可见时播放声音
    // "always" - 总是播放声音
    //
    // 默认：never
    "play_sound_when_agent_done": "never",
    // 代理面板中的编辑卡片是否展开，显示完整差异的预览。
    //
    // 默认：true
    "expand_edit_card": true,
    // 代理面板中的终端卡片是否展开，显示整个命令输出。
    //
    // 默认：true
    "expand_terminal_card": true,
    // 当 Zed 在代理面板中创建终端线程外壳时自动运行的命令。
    // 该命令如同键入一样发送到外壳，因此由你配置的外壳解释
    // （包括在 Windows 和远程/WSL 项目中）。
    // 设置为 "" 禁用。
    //
    // 示例："terminal_init_command": "claude"
    "terminal_init_command": "",
    // 默认情况下如何在代理面板中显示思考块。
    //
    // 默认：auto
    "thinking_display": "auto",
    // 点击正在运行的终端工具上的停止按钮时，是否也应取消代理的生成。
    // 请注意，这仅适用于停止按钮，不适用于终端内的 ctrl+c。
    //
    // 默认：true
    "cancel_generation_on_terminal_stop": true,
    // 是否始终使用 cmd-enter（在 Linux 或 Windows 上为 ctrl-enter）在代理面板中发送消息。
    //
    // 默认：false
    "use_modifier_to_send": false,
    // 代理消息编辑器中显示的最小行数。
    //
    // 默认：4
    "message_editor_min_lines": 4,
    // 是否显示轮次统计信息（生成期间的经过时间、最终轮次持续时间）。
    //
    // 默认：false
    "show_turn_stats": false,
    // 是否在状态栏中显示合并冲突指示器，
    // 该指示器提供使用代理解决冲突的功能。
    //
    // 默认：true
    "show_merge_conflict_indicator": true,
  },
  // 是否在操作系统状态栏中显示屏幕共享图标。
  "show_call_status_icon": true,
  // 是否使用语言服务器提供代码智能。
  "enable_language_server": true,
  // 如果语言服务器支持，是否执行关联范围的链接编辑。
  // 例如，编辑开头的 <html> 标签时，闭合的 </html> 标签的内容也会被编辑。
  "linked_edits": true,
  // 用于所有语言的语言服务器列表（或禁用）。
  //
  // 这通常按语言进行自定义。
  "language_servers": ["..."],
  // 控制如何使用来自语言服务器的语义令牌进行语法高亮。
  //
  // 选项：
  // - "off"：不从语言服务器请求语义令牌。
  // - "combined"：将 LSP 语义令牌与 tree-sitter 高亮结合作为基础。
  // - "full"：仅使用 LSP 语义令牌来高亮文本，tree-sitter 语法高亮关闭。
  //
  // 可能需要重新启动语言服务器才能正确应用。
  "semantic_tokens": "off",

  // 控制是否使用来自语言服务器的折叠范围，而不是
  // tree-sitter 和基于缩进的折叠。
  //
  // 选项：
  // - "off"：使用 tree-sitter 和基于缩进的折叠（默认）。
  // - "on"：尽可能使用 LSP 折叠，当服务器没有返回结果时回退到 tree-sitter 和基于缩进的折叠。
  "document_folding_ranges": "off",

  // 控制用于大纲和面包屑的文档符号来源。
  //
  // 选项：
  // - "off"：使用 tree-sitter 查询来计算文档符号（默认）。
  // - "on"：使用语言服务器的 `textDocument/documentSymbol` LSP 响应。
  //   启用后，不使用 tree-sitter 计算文档符号。
  "document_symbols": "off",

  // 何时自动保存已编辑的缓冲区。此设置可以取四个值。
  //
  // 1. 从不自动保存：
  //     "autosave": "off",
  // 2. 当焦点从 Zed 窗口移开时保存：
  //     "autosave": "on_window_change",
  // 3. 当焦点从特定缓冲区移开时保存：
  //     "autosave": "on_focus_change",
  // 4. 空闲一定时间后保存：
  //     "autosave": { "after_delay": {"milliseconds": 500} },
  "autosave": "off",
  // 每个窗格的最大标签数。未设置表示无限制。
  "max_tabs": null,
  // 与编辑器标签栏相关的设置。
  "tab_bar": {
    // 是否在编辑器中显示标签栏
    "show": true,
    // 是否显示导航历史按钮。
    "show_nav_history_buttons": true,
    // 是否显示标签栏按钮。
    "show_tab_bar_buttons": true,
    // 是否将固定标签显示在单独的一行中。
    // 启用时，固定标签出现在顶部行，未固定标签出现在底部行。
    // 禁用时，所有标签显示在单行中（默认行为）。
    "show_pinned_tabs_in_separate_row": false,
  },
  // 与编辑器标签相关的设置
  "tabs": {
    // 在编辑器标签中显示 git 状态颜色。
    "git_status": false,
    // 编辑器标签上关闭按钮的位置。
    // 可选：["right", "left"]
    "close_position": "right",
    // 是否为标签显示文件图标。
    "file_icons": false,
    // 控制标签关闭按钮的外观行为。
    //
    // 1. 仅在悬停标签时显示。（默认）
    //     "hover"
    // 2. 持久显示。
    //     "always"
    // 3. 从不显示，即使悬停也不显示。
    //     "hidden"
    "show_close_button": "hover",
    // 关闭当前标签后的操作。
    //
    // 1. 激活之前打开的标签（默认）
    //     "history"
    // 2. 激活右侧邻居标签（如果存在）
    //     "neighbour"
    // 3. 激活左侧邻居标签（如果存在）
    //     "left_neighbour"
    "activate_on_close": "history",
    // 在标签中标记哪些包含诊断错误/警告的文件。
    // 仅当文件图标也激活时才显示诊断。
    // 此设置仅在可以取以下三个值时有效：
    //
    // 1. 不标记任何文件：
    //    "off"
    // 2. 仅标记有错误的文件：
    //    "errors"
    // 3. 标记有错误和警告的文件：
    //    "all"
    "show_diagnostics": "off",
  },
  // 与预览标签相关的设置。
  "preview_tabs": {
    // 是否启用预览标签。
    // 预览标签允许你以预览模式打开文件，当你打开另一个预览标签时，
    // 它们会自动关闭。这对于快速查看文件而不使工作区混乱很有用。
    "enabled": true,
    // 从项目面板单击打开时是否以预览模式打开标签。
    "enable_preview_from_project_panel": true,
    // 从文件查找器选择时是否以预览模式打开标签。
    "enable_preview_from_file_finder": false,
    // 从多缓冲区打开时是否以预览模式打开标签。
    "enable_preview_from_multibuffer": true,
    // 使用代码导航打开多缓冲区时是否以预览模式打开标签。
    "enable_preview_multibuffer_from_code_navigation": false,
    // 使用代码导航打开单个文件时是否以预览模式打开标签。
    "enable_preview_file_from_code_navigation": true,
    // 当使用代码导航离开它们时，是否保持标签处于预览模式。
    // 如果 `enable_preview_file_from_code_navigation` 或
    // `enable_preview_multibuffer_from_code_navigation` 也为真，
    // 新标签可能会替换现有标签。
    "enable_keep_preview_on_code_navigation": false,
  },
  // 与文件查找器相关的设置。
  "file_finder": {
    // 是否在文件查找器中显示文件图标。
    "file_icons": true,
    // 确定文件查找器相对于可用窗口宽度可以占用的空间大小。
    // 有 5 种可能的宽度值：
    //
    // 1. 小：此值基本上是固定宽度。
    //    "modal_max_width": "small"
    // 2. 中：
    //    "modal_max_width": "medium"
    // 3. 大：
    //    "modal_max_width": "large"
    // 4. 特大：
    //    "modal_max_width": "xlarge"
    // 5. 全屏：此值移除任何水平内边距，因为它消耗整个视口宽度。
    //    "modal_max_width": "full"
    //
    // 默认：small
    "modal_max_width": "small",
    // 确定文件查找器在搜索结果中是否应跳过活动文件的焦点。
    // 有 2 个可能的值：
    //
    // 1. true：搜索文件时，如果当前活动文件作为第一个结果出现，
    //    自动焦点将跳过它并聚焦第二个结果。
    //    "skip_focus_for_active_in_search": true
    //
    // 2. false：搜索文件时，第一个结果将始终获得焦点，
    //    即使它是当前活动文件。
    //    "skip_focus_for_active_in_search": false
    //
    // 默认：true
    "skip_focus_for_active_in_search": true,
    // 搜索时是否使用被 gitignore 的文件。
    // 仅使用 Zed 已索引的文件，不一定是所有被 gitignore 的文件。
    //
    // 可以接受 3 个值：
    //   * "all"：使用所有被 gitignore 的文件
    //   * "indexed"：仅使用 Zed 已索引的文件
    //   * "smart"：聪明地搜索，当从被 gitignore 的工作树调用时搜索被忽略的文件
    "include_ignored": "smart",
    // 是否在文件查找器结果中包含文本频道。
    "include_channels": false,
  },
  // 保存缓冲区之前是否移除行尾的空白字符。
  "remove_trailing_whitespace_on_save": true,
  // 当上一行也是注释时，是否在新行继续注释。
  "extend_comment_on_newline": true,
  // 按回车时是否继续 Markdown 列表。
  "extend_list_on_newline": true,
  // 在列表标记后按 Tab 时是否缩进列表项。
  "indent_list_on_tab": true,
  // 移除文件末尾仅包含空白字符的行，并确保末尾只有一个换行符。
  "ensure_final_newline_on_save": true,
  // 新文件以及在格式化和保存期间如何处理行尾序列。
  // 此设置可以取五个值：
  //
  // 1. 检测现有行尾序列，否则使用平台默认值
  //    （Unix 上为 `lf`，Windows 上为 `crlf`）：
  //    "line_ending": "detect"
  // 2. 对于新文件和没有现有行尾序列的文件偏好 LF（`\n`）：
  //    "line_ending": "prefer_lf"
  // 3. 对于新文件和没有现有行尾序列的文件偏好 CRLF（`\r\n`）：
  //    "line_ending": "prefer_crlf"
  // 4. 在格式化和保存期间强制 LF（`\n`）：
  //    "line_ending": "enforce_lf"
  // 5. 在格式化和保存期间强制 CRLF（`\r\n`）：
  //    "line_ending": "enforce_crlf"
  //
  // EditorConfig 的 `end_of_line` 属性会覆盖此设置，其行为类似于
  // `enforce_lf` 或 `enforce_crlf`。
  "line_ending": "detect",
  // 保存前是否执行缓冲区格式化：
  //   "on" — 格式化整个缓冲区
  //   "off" — 不格式化
  //   "modifications" — 仅格式化有未暂存更改的行；当没有 git 差异可用
  //     或语言服务器缺少范围格式化时跳过格式化
  //   "modifications_if_available" — 相同，但当无法使用范围格式化时，
  //     回退到格式化整个缓冲区
  // 请记住，如果启用了延迟自动保存，format_on_save 将被忽略
  "format_on_save": "off",
  // 如何执行缓冲区格式化。此设置可以取多个值：
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
  // 7. 按顺序应用的上述任何格式化步骤的数组
  //     "formatter": [{"code_action": "source.fixAll.eslint"}, "prettier"]
  "formatter": "auto",
  // 如何软换行长行文本。
  // 可能的值：
  //
  // 1. 通常偏好单行，除非遇到超长行。
  //      "soft_wrap": "none",
  //      "soft_wrap": "prefer_line", // （已弃用，与 "none" 相同）
  // 2. 软换行溢出编辑器的行。
  //      "soft_wrap": "editor_width",
  // 3. 在首选行长或编辑器宽度（以较小者为准）处软换行。
  //      "soft_wrap": "bounded",
  "soft_wrap": "none",
  // 对于启用软换行的缓冲区，软换行的列位置。
  "preferred_line_length": 80,
  // 是否使用制表符缩进行，而不是多个空格。
  "hard_tabs": false,
  // 一个制表符应占用的列数。
  "tab_size": 4,
  // 在文件开头和结尾搜索模式行的行数。
  // 模式行包含编辑器指令（例如 vim/emacs 设置），用于为特定文件配置编辑器行为。
  //
  // 值为 0 禁用模式行支持。
  "modeline_lines": 5,
  // 默认情况下所有语言首选的调试器。
  "debuggers": [],
  // 是否在编辑器中启用单词差异高亮。
  //
  // 启用后，修改行中更改的单词会被高亮显示，
  // 以准确显示更改了什么。
  //
  // 默认：true
  "word_diff_enabled": true,
  // 控制 Zed 收集哪些信息。
  "telemetry": {
    // 发送调试信息，如崩溃报告。
    "diagnostics": true,
    // 发送匿名使用数据，如你在 Zed 中使用的语言。
    "metrics": true,
    // 允许发送无法以零数据保留方式提供的 Anthropic 模型请求
    "anthropic_retention": false,
  },
  // 是否禁用 Zed 中的所有 AI 功能。
  //
  // 默认：false
  "disable_ai": false,
  // 自动更新 Zed。如果通过包管理器安装，此设置在 Linux 上可能被忽略。
  "auto_update": true,
  // 如何在编辑器中渲染 LSP `textDocument/documentColor` 颜色。
  //
  // 可能的值：
  //
  // 1. 不查询和渲染文档颜色。
  //      "lsp_document_colors": "none",
  // 2. 将文档颜色渲染为颜色文本附近的内嵌提示（默认）。
  //      "lsp_document_colors": "inlay",
  // 3. 在颜色文本周围绘制边框。
  //      "lsp_document_colors": "border",
  // 4. 在颜色文本后面绘制背景。
  //      "lsp_document_colors": "background",
  "lsp_document_colors": "inlay",
  // 是否查询并显示编辑器中的 LSP `textDocument/documentLink` 链接。
  //
  // 默认：true
  "lsp_document_links": true,
  // 诊断配置。
  "diagnostics": {
    // 是否在状态栏中显示项目诊断按钮。
    "button": true,
    // 是否默认显示警告。
    //
    // 默认：true
    "include_warnings": true,
    // 在 Zed 中使用 LSP 拉取诊断机制的设置。
    "lsp_pull_diagnostics": {
      // 是否拉取诊断。
      "enabled": true,
      // 从语言服务器拉取诊断前等待的最小时间。
      // 0 关闭防抖。
      "debounce_ms": 50,
    },
    // 行内诊断设置
    "inline": {
      // 是否显示行内诊断
      "enabled": false,
      // 最后一次诊断更新后显示行内诊断的延迟（毫秒）。
      "update_debounce_ms": 150,
      // 源代码行结束与行内诊断开始之间的填充量，以 em 宽度为单位。
      "padding": 4,
      // 显示行内诊断的最小列。此设置可用于在某列水平对齐行内诊断。
      // 比此值长的行仍会将诊断进一步向右推。
      "min_column": 0,
      // 显示行内诊断的最小严重性。
      // 为 `null` 时继承编辑器诊断的最大严重性设置。
      "max_severity": null,
    },
  },
  // Zed 完全排除的文件或 glob 模式。它们在文件扫描、
  // 文件搜索期间被跳过，并且不显示在项目文件树中。优先级高于 `file_scan_inclusions`。
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
  // Zed 包含的文件或 glob 模式，即使被 git 忽略。这对于未被 git 跟踪
  // 但对项目仍然重要的文件很有用。请注意，过宽的 glob 模式会减慢
  // Zed 的文件扫描速度。`file_scan_exclusions` 优先于这些包含项。
  "file_scan_inclusions": [".env*"],
  // 在 git 仓库之外急切索引的最大目录深度；
  // 此深度或更深处的目录内容按需索引。
  // 根目录比此深度更浅的仓库总是完全索引。
  // 在不是以 git 仓库为根的项目中，根文件夹内直接包含的仓库会立即
  // 激活其 git 功能；更深的仓库在首次使用时激活。
  // `0` 表示无限制并立即激活所有 git 仓库。
  "file_scan_depth": 5,
  // 何时扫描链接目录的内容。
  // 可以取 2 个值：
  //  1. 仅在符号链接目录在工作区中展开时扫描：
  //         "scan_symlinks": "expanded"
  //  2. 总是扫描符号链接目录：
  //         "scan_symlinks": "always"
  "scan_symlinks": "expanded",
  // 用于匹配被视为“隐藏”文件的 glob 模式。可以通过切换
  // “hide_hidden”设置从项目面板隐藏这些文件。
  "hidden_files": ["**/.*"],
  // 用于匹配将以只读方式打开的文件的 glob 模式。你仍然可以查看这些文件，
  // 但无法编辑它们。这对于生成的文件或外部依赖很有用。
  "read_only_files": [],
  // Git 行号区域行为配置。
  "git": {
    // 全局开关，用于启用或禁用所有 git 集成功能。
    // 如果设置为 true，则禁用所有 git 集成功能。
    // 如果设置为 false，则下面的各个 git 集成功能将独立启用或禁用。
    "disable_git": false,
    // 是否启用 git 状态跟踪。
    "enable_status": true,
    // 是否启用 git 差异显示。
    "enable_diff": true,
    // 控制是否显示 git 行号区域。可以取 2 个值：
    // 1. 显示行号区域
    //      "git_gutter": "tracked_files"
    // 2. 隐藏行号区域
    //      "git_gutter": "hide"
    "git_gutter": "tracked_files",
    /// 设置更改反映在 git 行号区域中的防抖阈值（毫秒）。
    ///
    /// 默认：0
    "gutter_debounce": 0,
    // 控制是否为当前聚焦的行显示 git blame 信息，以及在哪里渲染。
    "inline_blame": {
      "enabled": true,
      // 设置显示行内 blame 信息之前的延迟。
      // 每次光标移动都会重新开始延迟。
      "delay_ms": 0,
      // 启用时 blame 信息的渲染位置。
      "location": "inline",
      // 源代码行结束与行内 blame 开始之间的填充量，以 em 宽度为单位。
      "padding": 7,
      // 是否在同一行显示 git 提交摘要。
      "show_commit_summary": false,
      // 显示行内 blame 信息的最小列号
      "min_column": 0,
    },
    "blame": {
      "show_avatar": true,
    },
    // 控制在分支选择器中显示哪些信息。
    "branch_picker": {
      "show_author_name": true,
    },
    "file_diff": {
      // 新打开的文件差异是否显示完整文件，而不是仅显示更改。
      "show_full_file": true,
    },
    // 编辑器中 git 差异块如何显示。
    // 此设置可以取两个值：
    //
    // 1. 未暂存的差异块填充，已暂存的差异块空心：
    //    "hunk_style": "staged_hollow"
    // 2. 未暂存的差异块空心，已暂存的差异块填充：
    //    "hunk_style": "unstaged_hollow"
    "hunk_style": "staged_hollow",
    // git 功能是与 HEAD（"head"）还是与默认分支（"default_branch"）比较差异。
    "diff_base": "head",
    // 在 git 视图中，名称或路径哪个先显示。
    // "path_style": "file_name_first" 或 "file_path_first"
    "path_style": "file_name_first",
    // 是否在差异块上显示暂存和恢复按钮。
    "show_stage_restore_buttons": true,
    // 创建 git 工作树的目录，相对于仓库工作目录。
    //
    // 当解析的目录在项目根目录之外时，会自动追加项目的目录名称，
    // 以便同级仓库不会冲突。例如，使用默认的
    // "../worktrees" 和位于 ~/code/zed 的项目，工作树将创建在
    // ~/code/worktrees/zed/ 下。
    //
    // 当解析的目录在项目根目录内时，不添加额外组件（它已经是项目范围的）。
    //
    // 示例：
    //   "../worktrees" — ~/code/worktrees/<project>/（默认）
    //   ".git/zed-worktrees" — <project>/.git/zed-worktrees/
    //   "my-worktrees" — <project>/my-worktrees/
    //
    // 尾部斜杠被忽略。
    "worktree_directory": "../worktrees",
  },
  // 自定义 Git 托管提供商列表。
  "git_hosting_providers": [
    // {
    //   "provider": "github",
    //   "name": "BigCorp GitHub",
    //   "base_url": "https://code.big-corp.com"
    // }
  ],
  // 配置如何加载 direnv 配置。可以取 2 个值：
  // 1. 使用 `direnv export json` 直接加载 direnv 配置。
  //      "load_direnv": "direct"
  // 2. 通过 shell 钩子加载 direnv 配置，适用于 POSIX shell 和 fish。
  //      "load_direnv": "shell_hook"
  // 3. 完全不加载 direnv 配置。
  //      "load_direnv": "disabled"
  "load_direnv": "direct",
  "edit_predictions": {
    // 要使用的编辑预测提供商。
    "provider": "zed",
    // 表示应禁用编辑预测的文件的 glob 模式列表。
    // 已经包含了一个合理的默认 glob 模式列表。
    // 对此列表的任何添加都将与默认列表合并。
    // Glob 模式相对于工作树根目录匹配，
    // 除非以斜杠（/）开头，或在 Windows 中等效。
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
    // 2. 仅当按住修饰键（默认为 alt）时内联显示预测。
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
    "ollama": {
      "api_url": "http://localhost:11434",
      "model": "qwen2.5-coder:7b-base",
      "prompt_format": "infer",
      "max_output_tokens": 64,
    },
    "open_ai_compatible_api": {
      "api_url": "",
      "model": "",
      "prompt_format": "infer",
      "max_output_tokens": 64,
    },
    // 控制 Zed 在使用 Zed 的编辑预测时是否可以收集训练数据。
    // 仅当项目被检测为开源时才捕获数据。
    // 可能的值：
    //   - "default"：使用之前通过状态栏切换设置的偏好，
    //     或者如果没有存储偏好则为 false。
    //   - "yes"：允许为开源项目中的文件收集数据。
    //   - "no"：从不允许数据收集。
    "allow_data_collection": "default",
  },
  // 日志记录特定设置
  "journal": {
    // 日志条目存储的目录路径
    "path": "~",
    // 以什么格式显示小时
    // 可以取 2 个值：
    // 1. hour12
    // 2. hour24
    "hour_format": "hour12",
  },
  // 状态栏相关设置。
  "status_bar": {
    // 是否显示状态栏。
    "experimental.show": true,
    // 是否在状态栏中显示活动文件的名称。
    "show_active_file": false,
    // 是否在状态栏中显示活动语言按钮。
    "active_language_button": true,
    // 是否在状态栏中显示光标位置按钮。
    "cursor_position_button": true,
    // 是否在状态栏中显示活动行尾按钮。
    "line_endings_button": false,
    // 控制何时在状态栏中显示活动编码。
    "active_encoding_button": "non_utf8",
  },
  // 终端特定设置
  "terminal": {
    // 打开终端时使用什么 shell。可以取 3 个值：
    // 1. 使用系统在 /etc/passwd 中的默认终端配置
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
    // 终端面板是否在启动时打开。
    //
    // 默认：false
    "starts_open": false,
    // 终端面板是否使用灵活（按比例）尺寸。
    //
    // 默认：true
    "flexible": true,
    // 终端停靠在左侧或右侧时的默认宽度。
    "default_width": 640,
    // 终端停靠在底部时的默认高度。
    "default_height": 320,
    // 启动终端时使用什么工作目录。
    // 可以取 5 个值：
    // 1. 使用当前文件的目录，回退到项目目录，然后是工作区中的第一个项目。
    //      "working_directory": "current_file_directory"
    // 2. 使用当前文件的项目目录。如果不成功，回退到
    //    第一个项目目录策略
    //      "working_directory": "current_project_directory"
    // 3. 使用工作区中第一个项目的目录
    //      "working_directory": "first_project_directory"
    // 4. 总是使用此平台的主目录（如果能找到）
    //     "working_directory": "always_home"
    // 5. 总是使用特定目录。此值将进行 shell 扩展。
    //    如果此路径不是有效目录，终端将默认使用
    //    此平台的主目录（如果能找到）
    //      "working_directory": {
    //        "always": {
    //          "directory": "~/zed/projects/"
    //        }
    //      }
    "working_directory": "current_project_directory",
    // 设置终端中光标的闪烁行为。
    // 可以取 3 个值：
    //  1. 从不闪烁光标，忽略终端模式
    //         "blinking": "off",
    //  2. 默认光标闪烁关闭，但允许终端设置闪烁
    //         "blinking": "terminal_controlled",
    //  3. 总是闪烁光标，忽略终端模式
    //         "blinking": "on",
    "blinking": "terminal_controlled",
    // 终端的默认光标形状。
    //  1. 包围下一个字符的方块
    //     "block"
    //  2. 竖线
    //     "bar"
    //  3. 沿下一个字符的下划线
    //     "underline"
    //  4. 在下一个字符周围绘制的方框
    //     "hollow"
    //
    // 默认："block"
    "cursor_shape": "block",
    // 设置交替滚动模式（代码：?1007）是否默认激活。
    // 交替滚动模式在备用屏幕（例如运行 vim 或 less 等应用程序时）
    // 将鼠标滚动事件转换为上/下键按下。终端仍然可以设置和取消此模式。
    // 可以取 2 个值：
    //  1. 默认交替滚动模式开启
    //         "alternate_scroll": "on",
    //  2. 默认交替滚动模式关闭
    //         "alternate_scroll": "off",
    "alternate_scroll": "on",
    // 设置 Option 键是否作为 Meta 键。
    // 可以取 2 个值：
    //  1. 依赖平台对 Option 键的默认处理，在 macOS 上
    //     这意味着生成某些 Unicode 字符
    //         "option_as_meta": false,
    //  2. 使 Option 键表现为 'meta' 键，例如用于 emacs
    //         "option_as_meta": true,
    "option_as_meta": false,
    // 在终端中选择文本是否自动复制到系统剪贴板。
    "copy_on_select": false,
    // 复制到剪贴板后是否保持文本选区。
    "keep_selection_on_copy": true,
    // 即使终端应用程序启用了鼠标报告（例如使用 mouse=a 的 vim、
    // htop），cmd-单击（Linux 和 Windows 上为 ctrl-单击）是否打开超链接。
    // 为 false 时，这些单击将转发给应用程序，
    // 超链接仍然可以使用 shift-cmd-单击（shift-ctrl-单击）打开。
    "open_links_in_mouse_mode": true,
    // 是否在状态栏中显示终端按钮
    "button": true,
    // 添加到此列表的任何键值对都将添加到终端的环境中。
    // 使用 `:` 分隔多个值。
    "env": {
      // "KEY": "value1:value2"
    },
    // 设置终端的行高。
    // 可以取 3 个值：
    //  1. 使用适合阅读的舒适行高，1.618
    //         "line_height": "comfortable"
    //  2. 使用标准行高，1.3。此选项对 TUI 很有用，
    //     特别是如果它们使用方框字符
    //         "line_height": "standard",
    //  3. 使用自定义行高。
    //         "line_height": {
    //           "custom": 2
    //         },
    "line_height": "standard",
    // 如果找到 Python 虚拟环境，则在终端的工作目录中激活它
    // （由 working_directory 设置解析）。将此设置为 "off" 禁用此行为。
    "detect_venv": {
      "on": {
        // 相对于当前工作目录搜索虚拟环境的默认目录。
        // 我们建议在项目设置中覆盖此设置，而不是全局设置。
        "directories": [".env", "env", ".venv", "venv"],
        // 也可以是 `csh`、`fish`、`nushell` 和 `power_shell`
        "activate_script": "default",
        // 激活 Conda 环境时首选的 Conda 管理器。
        // 值："auto", "conda", "mamba", "micromamba"
        // 默认："auto"
        "conda_manager": "auto",
      },
    },
    "toolbar": {
      // 是否在其工具栏的面包屑中显示终端标题。
      // 仅当终端标题非空时显示。
      //
      // 终端中运行的外壳需要配置为发出标题。
      // 示例：`echo -e "\e]2;New Title\007";`
      "breadcrumbs": false,
    },
    // 滚动条相关设置
    "scrollbar": {
      // 何时在终端中显示滚动条。
      // 此设置可以取五个值：
      //
      // 1. null（默认）：继承编辑器设置
      // 2. 如果有重要信息或遵循系统配置的行为，则显示滚动条（默认）：
      //   "auto"
      // 3. 匹配系统配置的行为：
      //    "system"
      // 4. 总是显示滚动条：
      //    "always"
      // 5. 从不显示滚动条：
      //    "never"
      "show": null,
    },
    // 设置终端的字体大小。如果不包含此选项，
    // 终端将默认为匹配缓冲区的字体大小。
    // "font_size": 15,
    // 设置终端的字体系列。如果不包含此选项，
    // 终端将默认为匹配缓冲区的字体系列。
    // "font_family": ".ZedMono",
    // 设置终端的字体后备。如果不包含此选项，
    // 终端将默认为匹配缓冲区的字体后备。
    // 这将与平台的默认字体后备合并
    // "font_fallbacks": ["FiraCode Nerd Fonts"],
    // 编辑器字体的粗细，标准 CSS 单位，从 100 到 900。
    "font_weight": 400,
    // 设置终端滚动缓冲区中的最大行数。
    // 默认：10_000，最大：100_000（所有大于此值的设置将被视为 100_000），0 禁用滚动。
    // 现有终端在重新创建之前不会应用此更改。
    "max_scroll_history_lines": 10000,
    // 终端中滚动速度的乘数。
    "scroll_multiplier": 1.0,
    // 前景色和背景色之间的最小 APCA 感知对比度。
    // APCA（可访问感知对比度算法）比 WCAG 2.x 更准确，
    // 尤其在深色模式下。值范围从 0 到 106。
    //
    // 基于 APCA 可读性标准（ARC）青铜简单模式：
    // https://readtech.org/ARC/tests/bronze-simple-mode/
    // - 0：无对比度调整
    // - 45：大型流畅文本（36px+）的最低要求
    // - 60：其他内容文本的最低要求
    // - 75：正文文本的最低要求
    // - 90：正文文本的推荐值
    //
    // 大多数终端主题的 APCA 值在 40-70 之间。
    // 值 45 在确保可读性的同时保留了色彩丰富的主题。
    "minimum_contrast": 45,
    // 用于识别超链接导航路径的正则表达式。支持可选命名捕获组
    // `path`、`line`、`column` 和 `link`。如果这些都不存在，则整个匹配
    // 是超链接目标。如果存在 `path`，则它是超链接目标，连同 `line`
    // 和 `column`（如果存在）。`link` 可用于自定义终端中哪些文本是超链接
    // 的一部分。如果 `link` 不存在，则使用整个匹配的文本。如果 `line` 和
    // `column` 不存在，则使用默认的内置行和列后缀处理，该处理解析
    // `line:column` 和 `(line,column)` 变体。默认值处理 Python 诊断和常见的
    // 路径、行、列语法。这可以扩展或替换以处理特定场景。例如，要支持
    // rust 输出中包含空格的路径的超链接，
    //
    // [
    //   "\\s+(-->|:::|at) (?<link>(?<path>.+?))(:$|$)",
    //   "\\s+(Compiling|Checking|Documenting) [^(]+\\((?<link>(?<path>.+))\\)"
    // ],
    //
    // 可以使用。处理在第一个匹配的正则表达式处停止，即使没有产生链接，
    // 当光标不在超链接文本上时就是这种情况。为了获得最佳性能，
    // 建议将正则表达式从最常见到最不常见排序。为了可读性和文档，
    // 每个正则表达式可以是字符串数组，这些字符串被收集到一个多行正则表达式
    // 字符串中，用于终端路径超链接检测。
    "path_hyperlink_regexes": [
      // Python 风格诊断
      "File \"(?<path>[^\"]+)\", line (?<line>[0-9]+)",
      // 常见路径语法，带有可选的行、列、描述、尾随标点或周围的符号或引号
      [
        "(?x)",
        "(?<path>",
        "    (",
        "        # 多字符路径：第一个字符（不是开分隔符、空格或制表符绘图字符）",
        "        [^({\\[<\"'`\\ \\u2500-\\u257F]",
        "        # 中间字符：非空格，冒号/括号仅在后跟不是数字/括号/空格时",
        "        ([^\\ :(]|[:(][^0-9()\\ ])*",
        "        # 最后一个字符：不是闭分隔符或冒号",
        "        [^()}\\]>\"'`.,;:\\ ]",
        "    |",
        "        # 单字符路径：不是分隔符、标点、空格或制表符绘图字符",
        "        [^(){}\\[\\]<>\"'`.,;:\\ \\u2500-\\u257F]",
        "    )",
        "    # 可选的行/列后缀（包含在 PathWithPosition::parse_str 的路径中）",
        "    (:+[0-9]+(:[0-9]+)?|:?\\([0-9]+([,:]?[0-9]+)?\\))?",
        ")",
      ],
    ],
    // 悬停和 Cmd-单击路径超链接发现的超时时间（毫秒）。指定超时为
    // `0` 将禁用终端中的路径超链接。
    "path_hyperlink_timeout_ms": 1,
    // 是否在终端面板图标上显示打开终端计数的徽章。
    "show_count_badge": false,
    // 当打印终端响铃（BEL 字符）时是否调用操作系统特定的警报声。
    "bell": "off",
  },
  "code_actions_on_format": {},
  // 与运行任务相关的设置。
  "tasks": {
    "variables": {},
    "enabled": true,
    // 优先使用 LSP 任务而不是 Zed 语言扩展任务。
    // 如果由于错误/超时或常规执行而没有返回 LSP 任务，
    // 则将使用 Zed 语言扩展任务。
    //
    // 其他 Zed 任务仍会显示：
    // * 来自任一任务配置文件的 Zed 任务
    // * 来自历史记录的 Zed 任务（例如之前生成的一次性任务）
    //
    // 默认：true
    "prefer_lsp": true,
  },
  // 一个对象，其键是语言名称，其值是应使用这些语言的
  // 文件名或扩展名数组。
  //
  // 例如，将像 `foo.notjs` 这样的文件视为 JavaScript，
  // 将 `Embargo.lock` 视为 TOML：
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
  // 安装语言服务器和 Copilot 时使用哪个版本的 Node.js 和 NPM 的设置。
  //
  // 注意：更改此设置当前需要重新启动 Zed。
  "node": {
    // 默认情况下，Zed 会在你的 `$PATH` 中查找 `node` 和 `npm`，如果它们的版本
    // 足够新，则使用现有的可执行文件。将此设置为 `true` 可阻止此行为，
    // 并强制 Zed 始终下载并安装自己的 Node 版本。
    "ignore_system_version": false,
    // 你也可以指定 Node 和 NPM 的替代路径。如果你指定了
    // `path`，但没有指定 `npm_path`，Zed 将假定 `npm` 位于
    // `${path}/../npm`。
    "path": null,
    "npm_path": null,
  },
  // Zed 应在启动时自动安装的扩展。
  //
  // 如果你不想要这些扩展中的任何一个，请在你的设置中添加此字段
  // 并将值更改为 `false`。
  "auto_install_extensions": {
    "html": true,
  },
  // 授予扩展的能力。
  //
  // 此列表可以自定义以限制扩展可以执行的操作。
  "granted_extension_capabilities": [
    { "kind": "process:exec", "command": "*", "args": ["**"] },
    { "kind": "download_file", "host": "*", "path": ["**"] },
    { "kind": "npm:install", "package": "*" },
  ],
  // 控制如何处理此语言的补全。
  "completions": {
    // 控制如何完成单词。
    // 对于大型文档，可能不会获取所有单词用于补全。
    //
    // 可以取 3 个值：
    // 1. "enabled"
    //   总是获取文档的单词用于补全，以及 LSP 补全。
    // 2. "fallback"
    //   仅当 LSP 响应错误或超时时，使用文档的单词显示补全。
    // 3. "disabled"
    //   从不获取或完成文档的单词用于补全。
    //   （基于单词的补全仍可通过单独的操作查询）
    //
    // 默认：fallback
    "words": "fallback",
    // 自动触发基于单词的补全所需的最小字符数。
    // 在该值之前，仍然可以通过相应的编辑器命令手动触发基于单词的补全。
    //
    // 默认：3
    "words_min_length": 3,
    // 是否获取 LSP 补全。
    //
    // 默认：true
    "lsp": true,
    // 获取 LSP 补全时，确定等待特定服务器响应的时长。
    // 设置为 0 时，无限等待。
    //
    // 默认：0
    "lsp_fetch_timeout_ms": 0,
    // 接受 LSP 补全时控制替换的范围。
    //
    // 当 LSP 服务器给出 `InsertReplaceEdit` 补全时，它们提供两个范围：`insert` 和 `replace`。通常，`insert`
    // 包含光标前的单词前缀，`replace` 包含整个单词。
    //
    // 实际上，此设置只是改变 Zed 是使用 `insert` 还是 `replace` 的接收范围，因此结果可能
    // 因底层 LSP 服务器而异。
    //
    // 可能的值：
    // 1. "insert"
    //   使用 LSP 规范中描述的 `insert` 范围替换光标前的文本。
    // 2. "replace"
    //   使用 LSP 规范中描述的 `replace` 范围替换光标前后的文本。
    // 3. "replace_subsequence"
    //   如果要替换的文本是补全文本的子序列，则行为类似于 `"replace"`，
    //   否则类似于 `"insert"`。
    // 4. "replace_suffix"
    //   如果光标后的文本是补全的后缀，则行为类似于 `"replace"`，否则类似于
    //   `"insert"`。
    "lsp_insert_mode": "replace_suffix",
  },
  // 特定语言的不同设置。
  "languages": {
    "Astro": {
      "format_on_save": "on",
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
      "use_on_type_format": false,
      "prettier": {
        "allowed": false,
      },
    },
    "C++": {
      "use_on_type_format": false,
      "prettier": {
        "allowed": false,
      },
    },
    "CSharp": {
      "language_servers": ["roslyn", "!csharp-ls", "!omnisharp", "..."],
    },
    "CSS": {
      "prettier": {
        "allowed": true,
      },
    },
    "Dart": {
      "format_on_save": "on",
      "tab_size": 2,
    },
    "Diff": {
      "show_edit_predictions": false,
      "remove_trailing_whitespace_on_save": false,
      "ensure_final_newline_on_save": false,
    },
    "EEx": {
      "format_on_save": "on",
      "language_servers": ["elixir-ls", "!expert", "!dexter", "!next-ls", "!lexical", "..."],
    },
    "Elixir": {
      "format_on_save": "on",
      "language_servers": ["elixir-ls", "!expert", "!dexter", "!next-ls", "!lexical", "!emmet-language-server", "..."],
    },
    "Elm": {
      "format_on_save": "on",
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
      "format_on_save": "on",
      "hard_tabs": true,
      "code_actions_on_format": {
        "source.organizeImports": true,
      },
      "debuggers": ["Delve"],
    },
    "GraphQL": {
      "format_on_save": "on",
      "prettier": {
        "allowed": true,
      },
    },
    "HEEx": {
      "format_on_save": "on",
      "language_servers": ["elixir-ls", "!expert", "!dexter", "!next-ls", "!lexical", "..."],
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
      "format_on_save": "on",
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
      "language_servers": ["phpactor", "!intelephense", "!phptools", "!phpantom", "..."],
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
      "language_servers": ["buf", "!protols", "!protobuf-language-server", "..."],
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
      "language_servers": ["basedpyright", "ruff", "!ty", "!pyrefly", "!pyright", "!pylsp", "..."],
    },
    "Ruby": {
      "language_servers": [
        "solargraph",
        "!ruby-lsp",
        "!rubocop",
        "!sorbet",
        "!steep",
        "!kanayago",
        "!fuzzy-ruby-server",
        "...",
      ],
    },
    "Rust": {
      "format_on_save": "on",
      "debuggers": ["CodeLLDB"],
    },
    "SCSS": {
      "prettier": {
        "allowed": true,
      },
    },
    "Starlark": {
      "format_on_save": "on",
      "language_servers": ["starpls", "!buck2-lsp", "!tilt", "..."],
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
      "format_on_save": "on",
      "language_servers": ["zls", "..."],
    },
  },
  // 特定语言模型的不同设置。
  "language_models": {
    "anthropic": {
      "api_url": "https://api.anthropic.com",
    },
    "anthropic_compatible": {},
    "bedrock": {},
    "google": {
      "api_url": "https://generativelanguage.googleapis.com",
    },
    "ollama": {
      "api_url": "http://localhost:11434",
    },
    "llama.cpp": {
      "api_url": "http://localhost:8080",
    },
    "openai": {
      "api_url": "https://api.openai.com/v1",
    },
    "openai_compatible": {},
    "opencode": {
      "api_url": "https://opencode.ai/zen",
    },
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
    "vercel_ai_gateway": {
      "api_url": "https://ai-gateway.vercel.sh/v1",
    },
    "x_ai": {
      "api_url": "https://api.x.ai/v1",
    },
    "zed.dev": {},
  },
  "session": {
    // 是否在重新启动时恢复未保存的缓冲区。
    //
    // 如果为 true，关闭应用程序时不会提示用户是否保存/丢弃
    // 脏文件。
    //
    // 默认：true
    "restore_unsaved_buffers": true,
    // 是否跳过工作树信任检查。
    // 受信任时，项目设置会自动同步，
    // 语言和 MCP 服务器会自动下载和启动。
    //
    // 默认：false
    "trust_all_worktrees": false,
  },
  // Zed 的 Prettier 集成设置。
  // 允许启用/禁用使用 Prettier 进行格式化
  // 并配置默认的 Prettier，在未找到项目级 Prettier 安装时使用。
  "prettier": {
    // 为任何给定语言启用或禁用使用 Prettier 进行格式化。
    "allowed": false,
    // 强制 Prettier 集成在格式化该语言的文件时使用特定的解析器名称。
    "plugins": [],
    // 默认的 Prettier 选项，格式与 package.json 中 Prettier 部分相同。
    // 如果项目通过其 package.json 安装了 Prettier，这些选项将被忽略。
    // "trailingComma": "es5",
    // "tabWidth": 4,
    // "semi": false,
    // "singleQuote": true
    // 当设置为非空字符串时，强制 Prettier 集成在格式化该语言的文件时
    // 使用特定的解析器名称。
    "parser": "",
  },
  // JSX 标签自动关闭的设置。
  "jsx_tag_auto_close": {
    "enabled": true,
  },
  // LSP 特定设置。
  "lsp": {
    // 在此处指定 LSP 名称作为键。
    // "rust-analyzer": {
    //     // rust-analyzer 集成的特殊标志，用于使用服务器提供的任务
    //     enable_lsp_tasks": true,
    //     // 这些初始化选项会合并到 Zed 的默认值中
    //     "initialization_options": {
    //         "check": {
    //             "command": "clippy" // rust-analyzer.check.command（默认："check"）
    //         }
    //     }
    // }
  },
  // DAP 特定设置。
  "dap": {
    // 在此处指定 DAP 名称作为键。
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
    // 等待语言服务器响应的最长时间，以秒为单位。
    // 值为 0 表示不应用超时。
    //
    // 默认：120
    "request_timeout": 120,
    // 缓冲区中允许的最大行长度，超过此长度将禁用整个缓冲区的语言服务器功能。
    //
    // 默认：20000
    "max_buffer_line_length": 20000,
    "notifications": {
      // 自动关闭语言服务器通知的超时时间（毫秒）。
      // 设置为 0 禁用自动关闭。
      "dismiss_timeout_ms": 5000,
    },
    // 高亮语义令牌的规则。用户定义的规则会被添加到默认规则之前
    // （可通过“显示默认语义令牌规则”查看），因此它们具有优先级。
    //
    // 每个 `rule` 具有以下属性：
    // - `token_type`：要自定义的 LSP 语义令牌类型。如果省略，则规则匹配所有令牌类型。
    // - `token_modifiers`：要匹配的 LSP 语义令牌修饰符列表。所有修饰符都必须存在才能匹配。
    // - `style`：要使用的当前语法主题中的样式列表。使用找到的第一个样式。
    //    下面的任何设置都会覆盖该样式。
    // - `foreground_color`：用于该令牌类型的前景色，十六进制格式（例如 "#ff0000"）。
    // - `background_color`：用于该令牌类型的背景色，十六进制格式。
    // - `underline`：布尔值或十六进制格式的颜色，用于下划线。如果为 `true`，则令牌将使用文本颜色加下划线。
    // - `strikethrough`：布尔值或十六进制格式的颜色，用于删除线。如果为 `true`，则令牌将使用文本颜色加删除线。
    // - `font_weight`："normal" 或 "bold" 之一。
    // - `font_style`："normal" 或 "italic" 之一。
    //
    // 对令牌应用第一个匹配的规则。因为用户定义的规则会被添加到默认规则之前，
    // 所以可以通过添加一个匹配令牌的空规则来完全禁用该令牌。
    //
    // 示例：以红色和粗体高亮未解析的引用：
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
    // 指定语言名称作为键，内核名称作为值。
    // "kernel_selections": {
    //    "python": "conda-base"
    //    "typescript": "deno"
    // }
  },
  // REPL 设置。
  "repl": {
    // REPL 回滚缓冲区中保留的最大列数。
    // 被限制在 [20, 512] 范围内。
    "max_columns": 128,
    // REPL 回滚缓冲区中保留的最大行数。
    // 被限制在 [4, 256] 范围内。
    "max_lines": 32,
    // 滚动前显示的最大输出行数。
    // 设置为 0 可禁用输出高度限制。
    "output_max_height_lines": 0,
    // 缩放图像前显示的最大输出列数。
    // 设置为 0 可禁用输出宽度限制。
    "output_max_width_columns": 0,
  },
  // Vim 设置
  "vim": {
    "default_mode": "normal",
    "toggle_relative_line_numbers": false,
    "use_system_clipboard": "always",
    "use_smartcase_find": false,
    "use_regex_search": true,
    "gdefault": false,
    "highlight_on_yank_duration": 200,
    "custom_digraphs": {},
    // 启用后，在 Vim 普通模式下也会显示编辑预测。
    // 默认情况下，编辑预测仅在插入和替换模式下显示。
    "show_edit_predictions_in_normal_mode": false,
    // 各模式的光标形状。
    // 形状可以是以下之一："block"、"bar"、"underline"、"hollow"。
    "cursor_shape": {
      "normal": "block",
      "replace": "underline",
      "visual": "block",
      // 设置为 "inherit" 以使用编辑器的 cursor_shape。
      "insert": "inherit",
    },
  },
  // Which-key 弹出窗口设置
  "which_key": {
    // 按住组合键时是否显示 which-key 弹出窗口。
    "enabled": false,
    // 显示 which-key 弹出窗口之前的延迟（毫秒）。
    "delay_ms": 1000,
  },
  // 要连接的服务器。如果设置了环境变量
  // ZED_SERVER_URL，它将覆盖此设置。
  "server_url": "https://zed.dev",
  // 使用 Zed Preview 时要使用的设置覆盖。
  // 主要用于管理多个 Zed 实例的开发者。
  "preview": {
    // "theme": "Andromeda"
  },
  // 使用 Zed Nightly 时要使用的设置覆盖。
  // 主要用于管理多个 Zed 实例的开发者。
  "nightly": {
    // "theme": "Andromeda"
    "instrumentation": {
      "performance_profiler": {
        "enabled": true,
      },
    },
  },
  // 使用 Zed Stable 时要使用的设置覆盖。
  // 主要用于管理多个 Zed 实例的开发者。
  "stable": {
    // "theme": "Andromeda"
  },
  // 使用 Zed Dev 时要使用的设置覆盖。
  // 主要用于管理多个 Zed 实例的开发者。
  "dev": {
    // "theme": "Andromeda"
    "instrumentation": {
      "performance_profiler": {
        "enabled": true,
      },
    },
  },
  // 在 Linux 上使用时要应用的设置覆盖。
  "linux": {},
  // 在 macOS 上使用时要应用的设置覆盖。
  "macos": {},
  // 在 Windows 上使用时要应用的设置覆盖。
  "windows": {
    "languages": {
      "PHP": {
        "language_servers": ["intelephense", "!phpactor", "!phptools", "!phpantom", "..."],
      },
    },
  },
  // 行指示器中显示完整标签还是简短标签
  //
  // 值：
  //   - `short`："2 s, 15 l, 32 c"
  //   - `long`："2 selections, 15 lines, 32 characters"
  // 默认：long
  "line_indicator_format": "long",
  // 设置要使用的代理。代理协议由 URI 方案指定。
  //
  // 支持的 URI 方案：`http`、`https`、`socks4`、`socks4a`、`socks5`、
  // `socks5h`。未指定方案时将使用 `http`。
  //
  // 默认情况下不使用代理，或者 Zed 会尝试从环境变量获取代理设置。
  // 如果某些主机不应被代理，请设置 `no_proxy` 环境变量并提供逗号分隔的列表。
  //
  // 示例：
  //   - "proxy": "socks5h://localhost:10808"
  //   - "proxy": "http://127.0.0.1:10809"
  "proxy": "",
  // 设置命令面板的别名。
  // 当输入的查询是此对象的键时，将使用其值代替。
  //
  // 示例：
  // {
  //   "W": "workspace::Save"
  // }
  "command_aliases": {},
  // ssh_connections 是一个 SSH 连接数组。
  // 你可以通过命令面板中的 `project: Open Remote` 配置这些连接。
  // Zed 的 SSH 支持也会从你的 ~/.ssh 中拉取配置。
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
  // 是否读取 ~/.ssh/config 以获取 SSH 连接源。
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
  // 默认：60
  "context_server_timeout": 60,
  // 配置供代理使用的上下文服务器。
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
  // `settings profile selector: toggle` 中选择时临时应用。
  //
  // 每个配置文件都有一个可选的 `base`（"user" 或 "default"）和一个
  // `settings` 对象。当 `base` 为 "user"（默认值）时，配置文件应用在
  // 你的用户设置之上。当 `base` 为 "default" 时，忽略用户设置，
  // 配置文件应用在 Zed 的默认值之上。
  //
  // 示例：
  // "profiles": {
  //   "Presenting": {
  //     "base": "default",
  //     "settings": {
  //       "agent_ui_font_size": 20.0,
  //       "buffer_font_size": 20.0,
  //       "theme": "One Light",
  //       "ui_font_size": 20.0
  //     }
  //   },
  //   "Python (ty)": {
  //     "settings": {
  //       "languages": {
  //         "Python": {
  //           "language_servers": ["ty"]
  //         }
  //       }
  //     }
  //   }
  // }
  "profiles": {},

  // 日志作用域到所需日志级别的映射。
  // 用于过滤嘈杂的日志或启用更详细的日志记录。
  //
  // 示例：{"log": {"client": "warn"}}
  "log": {},

  // 面向开发者的仪表化工具配置，可在运行时切换。
  "instrumentation": {
    // 性能分析器，通过 `zed: open performance profiler` 操作访问。
    // 收集前台和后台执行器任务的时间数据。
    // 启用此功能可能会导致内存使用增加，因此默认情况下
    // 在常规构建中禁用。
    "performance_profiler": {
      "enabled": false,
    },
  },
}
```

# 修改说明

下面把 settings.json 相对默认值的改动逐条列出来，格式「键：默认值 → 修改值，原因」。默认值以「Zed 默认配置全文」为准，JDTLS 部分以「JDTLS 偏好配置全文」的注释为准（默认配置里 lsp 块整个是空的）。和默认值相同但显式写出来的项，标「未改」。settings.json 全文在文末「完整配置：settings.json」。

## 外观与字体

- `theme`：One Light / One Dark → Catppuccin Latte (Blur) / Catppuccin Espresso (Blur)。换 Catppuccin 主题，Blur 变体带毛玻璃质感；`mode` 保持 system 跟随系统。
- `icon_theme`：Zed (Default) → Catppuccin Latte。图标配色跟随主题。
- `buffer_font_family`：.ZedMono → Maple Mono NF CN。中文覆盖完整、自带 Nerd Font 图标、连字齐全（安装见「字体安装」）。
- `buffer_font_features.calt`：未设置 → true。显式开启连字。官方标注该设置目前只在 macOS / Windows 生效，Linux 上保留只为跨平台配置统一。
- `ui_font_family`：.ZedSans → .SystemUIFont。界面文字用系统字体，与桌面环境观感一致。
- `ui_font_features.calt`：false → true。界面字体连字开启（同上，Linux 暂不生效）。

## 编辑器行为

- `base_keymap`：Zed，未改（显式声明）。
- `colorize_brackets`：false → true。彩虹括号，深嵌套代码里救眼睛。
- `completion_menu_item_kind`：off → symbol。补全菜单显示 LSP 项类型徽章。
- `diff_view_style`：split → unified。单栏 diff 省水平空间。
- `auto_signature_help`：false → true。光标在括号内自动弹方法签名。
- `show_signature_help_after_edits`：false → true。补全或插入括号对后也显示签名。
- `code_lens`：off → on。方法上方显示引用数、实现数。
- `indent_guides.coloring`：fixed → indent_aware。缩进辅助线按层级着色。
- `use_smartcase_search`：false → true。智能大小写搜索。
- `inlay_hints.enabled`：false → true。内联提示总开关。
- `semantic_tokens`：off → combined。高亮结合 LSP 语义标记与 tree-sitter。
- `document_folding_ranges`：off → on。折叠范围交给语言服务器。
- `document_symbols`：off → on。大纲与面包屑用 documentSymbol。
- `autosave`：off → on_focus_change。焦点离开缓冲区就保存。
- `format_on_save`：off → modifications。只格式化有未暂存改动的行。
- `soft_wrap`：none → bounded。120 列与编辑器宽度较小者处软换行。
- `preferred_line_length`：80 → 120。行宽上限。

## 面板与隐私

- `collaboration_panel.button`：true → false。不用实时协作，收掉状态栏按钮。
- `git_panel.group_by`：status → none。不按状态分组。
- `git_panel.file_icons`：false → true。面板显示文件图标。
- `git_panel.tree_view`：false → true。树形视图，变更文件按目录层级展示。
- `telemetry.diagnostics` / `telemetry.metrics`：true → false。不发送崩溃诊断与匿名使用数据。
- `diagnostics.inline.enabled`：false → true。行内诊断。
- `edit_predictions.allow_data_collection`：default → no。编辑预测不参与任何数据收集。
- `session.trust_all_worktrees`：false → true。跳过工作树信任弹窗（机器与项目来源可控时才开）。

## 终端

- `terminal.shell`：system → fish。显式指定 fish 4.x。
- `terminal.detect_venv.activate_script`：default → fish。venv 激活脚本类型与 shell 匹配；搜索目录列表保持默认。

## AI Agent

- `agent.default_model`：zed.dev claude-sonnet-4（不思考）→ deepseek deepseek-v4-pro + enable_thinking + effort max。主用 DeepSeek，思考开启，推理强度拉满。
- `agent.default_profile`：write，未改（显式声明）。
- `agent.commit_message_model`：默认无 → deepseek-v4-flash。提交信息任务用快且便宜的模型。
- `agent.commit_message_instructions`：默认空 → 约定式提交中文提示词。约束提交信息格式（类型表、长度上限、脚注规则）。
- `agent.commit_message_include_project_rules`：true → false。生成提交信息时不注入项目规则。
- `agent.sandbox_permissions.allow_unsandboxed`：默认配置未列出（无 sandbox_permissions 块）→ 显式 false，禁止无沙箱执行。
- `agent.sandbox_permissions.write_paths`：默认无 → 放行 fnm 状态目录与 ~/.m2/repository（沙箱内默认只读，需要写的目录显式列出）。
- `agent.tool_permissions.default`：confirm，未改（显式声明）。
- `agent.tool_permissions.tools.terminal.always_allow`：默认空 → 只读/构建类命令前缀白名单。查看类命令直接放行，破坏性操作保持确认。
- `agent.favorite_models` / `agent.model_parameters`：[]，未改（显式声明）。

## LSP：JDTLS（Java）

- `java_home`：默认自动探测（JAVA_HOME 或 PATH）→ /home/zhangtianci/.jdks/25。显式指定 JDTLS 运行用的 JDK。
- `lombok_support`：true，未改（显式声明）。
- `jdk_auto_download`：false，未改（显式声明）。
- `check_updates`：always，未改（显式声明）。
- `java.home`：null → /home/zhangtianci/.jdks/25。项目编译解析的默认 JDK。
- `java.edit.smartSemicolonDetection.enabled`：false → true。智能分号。
- `java.format.onType.enabled`：false → true。键入分号/右大括号时格式化当前行。
- `java.format.settings.url`：null → eclipse-java-formatter.xml 绝对路径。挂载 Eclipse 格式化器。
- `java.configuration.updateBuildConfiguration`：interactive → automatic。构建文件变化自动刷新项目模型。
- `java.configuration.runtimes`：[] → JavaSE-25/21/17/1.8 四档。多版本项目按 release 挑 JDK。
- `java.maven.downloadSources`：false → true。自动下载依赖源码 jar。
- `java.maven.updateSnapshots`：false → true。同步时强制更新 SNAPSHOT。
- `java.implementationCodeLens`：none → all。接口/抽象方法显示所有实现。
- `java.saveActions.organizeImports`：false → true。保存时整理 import。
- `java.signatureHelp.enabled`：false → true。签名帮助开启。
- `java.signatureHelp.description.enabled`：false → true。签名帮助显示 Javadoc 概要。
- `java.maxConcurrentBuilds`：1 → 5。构建任务并发上限。
- `java.completion.lazyResolveTextEdit.enabled`：false → true。补全列表快速返回，详情延迟加载。
- `java.completion.guessMethodArguments`：insertParameterNames → insertBestGuessedArguments。选中方法后插入最佳猜测实参。
- `java.completion.favoriteStaticMembers`：默认 JUnit 六项 → 追加 java.util.Objects.*、java.util.Collections.*。
- `java.completion.importOrder`：[java, javax, org, com] → 追加 io。
- `java.completion.chain.enabled`：false → true。链式补全。
- `java.sources.organizeImports.starThreshold`：99 → 50。同包普通导入超过 50 个压成星号导入。
- `java.sources.organizeImports.staticStarThreshold`：99 → 10。静态导入阈值。
- `java.codeGeneration.hashCodeEquals.useJava7Objects`：false → true。生成 Objects.hash / equals。
- `java.codeGeneration.hashCodeEquals.useInstanceof`：false → true。equals 用 instanceof。
- `java.codeGeneration.useBlocks`：false → true。生成代码的 if/for 带大括号。
- `java.codeGeneration.generateComments`：false → true。生成方法带 Javadoc。
- `java.codeGeneration.insertionLocation`：null → lastMember。新成员插到类最后。
- `java.codeGeneration.addFinalForNewDeclaration`：none → all。新字段一律 final。
- `java.codeGeneration.toString.template`：null → STRING_BUILDER_CHAINED。toString 用链式 StringBuilder。
- `java.codeGeneration.toString.codeStyle`：null → Eclipse。
- `java.codeGeneration.toString.skipNullValues`：false → true。跳过 null 字段。
- `java.codeGeneration.toString.limitElements`：0 → 10。集合最多列 10 个元素。
- `java.inlayHints.parameterNames.enabled`：literals → all。所有实参都显示参数名。
- `java.inlayHints.parameterNames.suppressWhenSameNameNumbered`：false → true。同名仅编号不同的抑制。
- `java.inlayHints.variableTypes.enabled`：false → true。var 显示推断类型。
- `java.inlayHints.parameterTypes.enabled`：false → true。实参旁显示参数类型。
- `java.inlayHints.formatParameters.enabled`：false → true。格式化方法参数旁显示占位符。
- `java.jdt.ls.protobufSupport.enabled`：false → true。实验性 .proto 支持。
- `java.jdt.ls.javac.enabled`：false → true。实验性 javac 编译器。
- `java.compile.nullAnalysis.nonnull / nullable / nonnullbydefault`：[] → 注册 javax 与 org.eclipse 两组注解。
- `java.compile.nullAnalysis.mode`：disabled → automatic。检测到注解自动启用空值分析。

JDTLS 部分未改（显式写出，等于默认值）：`java.format.enabled`（true）、`java.format.insertSpaces`（true）、`java.format.tabSize`（4）、`java.format.comments.enabled`（true）、`java.project.encoding`（ignore）、`java.completion.filteredTypes`（默认六项）、`java.codeGeneration.toString.listArrayContents`（true）、`java.telemetry.enabled`（false）。

## LSP：rust-analyzer（Rust）

- `enable_lsp_tasks`：true，未改（显式声明）。
- `inlayHints.maxLength`：25 → null。提示不截断。
- `inlayHints.lifetimeElisionHints.enable`：never → skip_trivial。返回类型涉及生命周期时才显示省略提示。
- `inlayHints.lifetimeElisionHints.useParameterNames`：false → true。提示优先用参数名。
- `inlayHints.closureReturnTypeHints.enable`：never → always。闭包返回类型直接标出。
- `diagnostics.experimental.enable`：false → true。实验性诊断。
- `check.command`：check → clippy。保存时检查用 clippy，多出一堆 lint。
- `cargo.features`：[] → all。分析时启用全部 feature。
- `cargo.targetDir`：null → true。rust-analyzer 用独立的 target/rust-analyzer 子目录。

rust-analyzer 部分未改（显式写出）：`diagnostics.disabled` / `warningsAsInfo` / `warningsAsHint`（[]）、`check.features`（null）、`check.ignore`（[]）。

## 其他

- `agent_servers`：{} → 注册 pi-acp（registry 类型）。接入 ACP Agent 服务器。

# 逐节拆解

这一节按场景把配置串一遍：外观与编辑行为、字体、终端、AI，Java 和 Rust 的环境搭建，最后是常用快捷键。完整配置在文末。

## 主题、字体与外观

外观这块折腾得最久，但配置本身只有三个字段：主题、图标、字体。

主题用的 Catppuccin，白天 Latte、晚上 Espresso，后缀都带 (Blur)，面板有毛玻璃质感。`theme.mode` 用 system，跟着系统明暗走，不用手动管。这个选择偏保守：Catppuccin 低饱和，四个变体之间配色统一，盯一天眼睛不累。One Light / One Dark 我也用过，浅色发灰、暗色偏冷，都不太对味；更早还试过一个 Vercel 主题，太花，弃了。想换口味按 Ctrl+K Ctrl+T 翻主题选择器，Ctrl+K Ctrl+Shift+T 是强制明暗切换。

键位 `base_keymap` 我留的 Zed 默认。刚从 IDEA 迁过来时选过 JetBrains 键位，肌肉记忆确实舒服，但文档、教程、别人分享的 keymap 全按 Zed 键位写，两套记着累，最后又换回来了。个别键想改就去 `keymap.json`（见「常用快捷键」），不用动 base_keymap。

字体 `buffer_font_family` 填家族名，不是文件名，装完用 `fc-list | rg Maple` 确认（「字体安装」有完整步骤）。连字由 `buffer_font_features` 的 `calt` 管，官方标注这设置只在 macOS / Windows 生效，Linux 上先留着，等它支持。界面字体用 `.SystemUIFont`，跟着系统走。

剩下的四个小开关：`colorize_brackets` 彩虹括号，深嵌套代码里能救命；`completion_menu_item_kind` 让补全项前面多一个类型徽章（f 函数、m 方法、v 变量），扫一眼就知道是什么；`diff_view_style` 用 unified，单栏比左右分栏省地方，行多的文件尤其明显；`code_lens` 在方法上方标引用数和实现数，Java 里翻实现列表全靠它。

## 编辑器行为

`autosave` 用 on_focus_change，焦点一切走就存。没选定时保存，是因为打字间隙不想被落盘打断语言服务器；坏处也有，一直待在一个缓冲区里改东西它就不存，得切出去一趟。

`format_on_save` 我用了 modifications，只格式化有未暂存改动的行。这个值不是拍脑袋选的：老项目开全文件格式化（"on"），一保存整个文件重排，diff 全是噪音，Code Review 没法看；团队里格式不统一的话，还会把别人写的行顺手改了。modifications 只动你自己碰过的行，出了事也好查。注意一点，autosave 用 after_delay 定时保存的话，format_on_save 会被忽略。

`semantic_tokens`、`document_folding_ranges`、`document_symbols` 三个开关是同一个决定：高亮、折叠、大纲都交给语言服务器。tree-sitter 只认语法，语言服务器还分得清变量、参数、类型，semantic_tokens 用 combined 把两者叠起来。改完要重启语言服务器才生效，服务器没起来的时候这些能力会退回 tree-sitter 兜底。

搜索和提示的几个开关：`use_smartcase_search` 智能大小写，查询里有大写就区分，全小写就不区分；`inlay_hints.enabled` 是内联提示的总闸，具体哪些提示由语言服务器定，Java 和 Rust 各有一节；`indent_guides.coloring` 用 indent_aware，缩进线按层级变色。

`soft_wrap` 用 bounded，超过 120 列折行。120 不是随便写的，和 Java 格式化器的 lineSplit 一致，编辑器里的一行和格式化器眼里的一行是同一个概念。`auto_signature_help` 进括号就弹签名，`show_signature_help_after_edits` 补全之后也弹。

面板：`collaboration_panel` 的按钮关了，实时协作用不上，留着占状态栏；`git_panel` 开了文件图标和树形视图，group_by 留的 none，树形视图下再按状态分组会把一个目录的改动拆成好几段，找文件更慢。

隐私相关一口气说掉：`telemetry` 两个都关，不传崩溃诊断也不传使用数据；`edit_predictions.allow_data_collection` 写成 no；`diagnostics.inline.enabled` 打开行内诊断，报错直接跟在行尾，不用悬停到波浪线上再看。

`session.trust_all_worktrees` 关掉了工作树信任弹窗。Zed 会直接同步项目设置、自动下载启动语言服务器和 MCP 服务器，等于放弃了那层确认，只在机器和代码来源都可控的时候这么开。

## 字体安装

编辑区字体我用的 Maple Mono NF CN。选它三个原因：中文覆盖完整（CN 变体）、自带 Nerd Font 图标、连字齐全。NF 是 Nerd Font 变体的标记，文件名带 NF 的版本才内嵌图标字形，不带的话状态栏、文件树里的图标字符全变方块。同类还有 Sarasa Gothic（更纱黑体）和 LXGW WenKai，看口味。

Fedora 上安装：

```bash
mkdir -p ~/.local/share/fonts/MapleMono
# 从 https://github.com/subframe7536/maple-font/releases 下载 MapleMono-NF-CN.zip
unzip MapleMono-NF-CN.zip -d ~/.local/share/fonts/MapleMono
fc-cache -fv
```

确认字体家族名（不是文件名）：

```bash
fc-list | rg Maple
# /home/zhangtianci/.local/share/fonts/MapleMono/MapleMono-NF-CN-Regular.ttf: Maple Mono NF CN:style=Regular
```

冒号前面的 "Maple Mono NF CN" 才是家族名，填进 `buffer_font_family`。填错了 Zed 不报错，静默回退默认字体，所以改完扫一眼渲染效果。字号和行高我留在默认（15px / comfortable），要改的话是 `buffer_font_size` 和 `buffer_line_height`。macOS 的家族名在系统“字体册”里查，Windows 在设置的字体列表里查。

## 终端

Zed 自带终端面板（Ctrl+ 反引号键）。shell 在 `terminal.shell` 里选，`"system"` 表示用 /etc/passwd 里的登录 shell，我的配置显式写了 fish：

```jsonc
"terminal": {
  "shell": {
    "program": "fish",
  },
  // 在项目工作目录下发现 Python 虚拟环境时自动激活
  "detect_venv": {
    "on": {
      "directories": [".env", "env", ".venv", "venv"],
      "activate_script": "fish",
    },
  },
},
```

这套配置里，shell 用 fish 4.x；detect_venv 打开目录时自动激活 `.venv`，`activate_script` 必须和 shell 对上（fish / bash / csh / nushell / power_shell），写错的话激活脚本语法不匹配，venv 激活会静默失败。搜索目录列表是默认的四个，特殊项目在项目级设置里覆盖。

Linux 这边有个 Flatpak 的坑：沙箱里只有运行时自带的 sh 和 bash，宿主机的 fish 看不见。所以 Flatpak 版 Zed 建议把 `terminal` 块删掉或者 shell 改成 system，fish 配置只对原生安装有效。非要 fish 的话，要么指到沙箱里可用的解释器，要么换官方脚本安装。

## AI Agent

模型走 BYOK，key 自己带，我的默认模型是 deepseek-v4-pro，思考开着，effort 拉满。每个请求会更慢更贵，但复杂改动的方案明显更好；提交信息这种小任务单独用 deepseek-v4-flash，快且便宜。`default_profile` 用 write，能改文件、跑终端；只想问问题就切 ask，纯聊天用 minimal，Agent 面板里切换。

Agent 的终端默认在沙箱里跑。`sandbox_permissions` 里 allow_unsandboxed 显式 false，不让它跑出沙箱；write_paths 列出允许写的目录：fnm（Node 版本管理器）的状态目录和 `~/.m2/repository`（Maven 本地仓库），构建要写这两个地方，沙箱外其他目录默认只读。

`tool_permissions` 管命令确认：没匹配规则的命令每次确认（default 是 confirm），`terminal.always_allow` 里的正则前缀直接放行。规则按正则匹配命令文本，`^rg\b` 就是放行任何 rg 开头的命令。原则一句话：只读的放开，会改状态的手动过一眼。列表里 rg / fd / ls / head / tail / env / git 是查看类；mvn / java / which mvn 和带 JAVA_HOME 的 mvn test 是构建测试；ls ~/.jdks、ls target/surefire-reports 是排查用；中间还混着几条 Agent 内部工具命令（head_lines / tail_lines）和项目特定的 rg 搜索词，照抄之前把那些词换成你自己的。拿不准的命令就删掉对应模式，回到每次确认。

`commit_message_include_project_rules` 写成 false，生成提交信息不塞项目规则，这个任务上下文越少越好。`commit_message_instructions` 是提交信息的完整指令：约定式提交、中文、标题 50 字符以内、正文 72 字符行宽、BREAKING CHANGE 脚注，Git 面板里点一下就能按这套格式生成，指令全文在完整配置里。

规则注入：Zed 把仓库根的 `AGENTS.md`、`CLAUDE.md`、`.rules` 注入 Agent 上下文，全局规则放配置目录的 `AGENTS.md`（Flatpak 版在 `~/.var/app/dev.zed.Zed/config/zed/AGENTS.md`）。我在里面放的是全机器通用的约定：默认语言、shell 工具约束、Git 规范，不用每个仓库重复一遍。

`agent_servers` 接外部 Agent 服务器（ACP 协议），pi-acp 是已注册的服务器，registry 类型直接引用。

Agent 面板里 Enter 发送、Shift+Enter 换行，写一半想直接发就 Ctrl+Shift+Enter。

## Java（JDTLS）

Java 的能力来自 JDTLS，Eclipse 那套语言服务器，Zed 的扩展会自动下载并启动它。启动条件是机器上有 JDK 21 或更高（`JAVA_HOME` 或者 PATH 里的 `java`）。这里有两件事别混：JDTLS 自己运行用哪个 JDK，由 `java_home` 指定，必须是 21+；项目编译解析用哪个 JDK，由 `java.home` 和 `runtimes` 决定，项目要 8、17、21 都行。

这台机器是 Fedora Atomic，系统层不可变，开发环境我全部丢进 toolbox。rpm-ostree 叠包也行，但每叠一次都要重启，包装进镜像层之后想摘还麻烦，开发工具留在系统镜像里只会添乱：

```bash
toolbox create     # 用当前 Fedora 版本的镜像建一个开发容器
toolbox enter      # 进入容器，之后构建、测试都在里面跑
# 容器内：
sudo dnf install maven java-25-openjdk-devel java-21-openjdk-devel java-17-openjdk-devel java-1.8.0-openjdk-devel
```

dnf 的 java-XX-openjdk 包能并行安装，8/17/21/25 共存没问题（包名以 `dnf search openjdk` 为准）。toolbox 和宿主共享 home，`~/.m2` 是同一份，容器里跑构建，宿主的 Zed 打开同一套源码，两边不打架。

toolbox 管的是工具链。JDTLS 还有个边界问题：它由 Zed 启动，Zed 是宿主上的 Flatpak，看不见容器里的 `/usr/lib/jvm`。所以宿主上还得再放一份用户级 JDK 21+，专门给 JDTLS 用。SDKMAN 装，不动系统层：

```bash
curl -s "https://get.sdkman.io" | bash
sdk install java    # 最新稳定版；需要 Java 8 / 17 / 21 时用 sdk list java | rg tem 挑版本再装
```

SDKMAN 的版本目录带完整版本号，一升级路径就变。我给每个大版本建了固定软链，配置里写软链，以后升级不用改配置：

```bash
mkdir -p ~/.jdks
ln -sfn ~/.sdkman/candidates/java/<25 的版本目录> ~/.jdks/25
ln -sfn ~/.sdkman/candidates/java/<21 的版本目录> ~/.jdks/21
ln -sfn ~/.sdkman/candidates/java/<17 的版本目录> ~/.jdks/17
ln -sfn ~/.sdkman/candidates/java/<8 的版本目录>  ~/.jdks/8
```

`java_home`、`java.home`、`runtimes` 填这些软链。分工：宿主的 JDK 服务 JDTLS 的运行和语法解析（诊断用的是内存编译），出产物的构建在 toolbox 里跑。还有一条路是把 Zed 也装进 toolbox，编辑器、语言服务器、构建工具同一个环境，路径直接写容器里的 `/usr/lib/jvm/…`，前提是容器里 Vulkan 能用，我没走这条。

`lsp.jdtls.settings` 是 Zed 扩展这一层的配置。java_home 管 JDTLS 进程自己的 JDK；lombok_support 保持默认开启，扩展自动下载 lombok jar 注册成 Java agent；jdk_auto_download 保持默认 false，不让它自己下 JDK；check_updates 保持默认 always。

`initialization_options` 会原样传给 JDTLS，java 块我按功能过一遍：

- `java.home` 是项目默认 JDK，和 `java_home` 分工不同，一个管项目，一个管 JDTLS 自己。
- `edit.smartSemicolonDetection`：智能分号，已经有分号的地方不会再补一个。
- `format`：Eclipse 风格格式化。onType 在敲分号/右大括号时格式化当前行；insertSpaces 和 tabSize 4 定义缩进；settings.url 挂 Eclipse Formatter XML（全文在后面），必须绝对路径。
- `configuration.updateBuildConfiguration` 改成 automatic：pom.xml、build.gradle 一变就自动刷新项目模型，不弹窗。
- `configuration.runtimes` 是多版本调度表。name 用执行环境名，Java 8 叫 "JavaSE-1.8"，不是 "JavaSE-8"；JDTLS 按构建文件里的 release（pom 的 maven.compiler.release 或 source）挑 JDK。
- `maven.downloadSources` 自动拉依赖的源码 jar，跳定义直接进源码，不用看反编译；updateSnapshots 每次同步强制刷 SNAPSHOT。
- `implementationCodeLens` 开成 all，接口和抽象方法上方列出所有实现。
- `saveActions.organizeImports` 保存时整理 import；同包普通导入超过 50 个、静态导入超过 10 个压成星号（sources.organizeImports 的两个阈值）。
- `signatureHelp` 开签名帮助，顺带显示 Javadoc 概要。
- `maxConcurrentBuilds` 设 5。
- `project.encoding` 保持默认 ignore，项目没声明编码就用系统默认，不弹窗。
- `completion` 一组：lazyResolveTextEdit 让补全列表先出来，详情选中再加载；guessMethodArguments 选完方法直接插最佳猜测的实参；favoriteStaticMembers 把 Objects、Collections、JUnit 断言顶到最前面；importOrder 定 import 顺序；filteredTypes 过滤 jdk.*、sun.* 这类噪音；chain 开链式补全，`a.b().` 后面直接列 `b()` 返回类型的方法，不用跳过去看一眼再回来。
- `codeGeneration`：hashCode/equals 用 Objects 写法和 instanceof；生成的 if/for 带大括号；新成员插到类末尾并加 final；toString 用链式 StringBuilder，跳过 null 字段，集合最多列 10 个。
- `compile.nullAnalysis`：注册 @Nonnull/@Nullable 之后 mode 用 automatic，项目里一出现注解就自动开空值分析，null 问题提前到编译期。
- `inlayHints` 全开：参数名提示（"all"），同名仅编号不同的抑制；变量类型、参数类型、格式化占位符都开。刚开那阵子界面有点吵，习惯了就离不开了。
- `jdt.ls.javac` 和 `jdt.ls.protobufSupport` 是实验性开关：javac 打开后编译行为和标准 javac 一致，比 JDT 内置编译器慢；protobufSupport 给 .proto 文件。
- `telemetry.enabled` 保持默认 false。

宿主的 JDK 在 home 下（`~/.sdkman`、`~/.jdks`），Flatpak 沙箱有 home 访问权限，不用额外放行。Zed 报 JDK 找不到时先 `ls ~/.jdks/`，SDKMAN 升级重装之后软链可能悬空，重新指一下就行。

这里只是我启用的一部分。JDTLS 能调的项还很多（引用搜索、符号搜索、文件关联、导入策略、折叠范围、成员排序），完整的中文注释版在「JDTLS 偏好配置全文」。

## Rust（rust-analyzer）

rust-analyzer 不用自己装，Zed 先找 PATH 里的，找不到就用自己的稳定版。工具链用 rustup：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

rustup 只往 home 写（工具链在 `~/.rustup`，二进制在 `~/.cargo/bin`），不动系统层。这一点和 Java 不一样：JDK 装了两份（toolbox 里一份、宿主一份），Rust 一份就够，home 两边共享，宿主的 rust-analyzer 和容器里的构建用同一套工具链。容器里 cargo 不在 PATH 就补一行 export PATH="$HOME/.cargo/bin:$PATH"。

Flatpak 这里有个 Java 没有的坑。java_home 是绝对路径，沙箱直接读；rust-analyzer 找 cargo 靠 PATH，而 Flatpak 版 Zed 的沙箱 PATH 只有 `/app/bin` 和 `/usr/bin`，没有 `~/.cargo/bin`，工作区加载直接失败，日志里是 cargo --version failed。解法是给 Flatpak 注入 PATH：

```bash
flatpak override --user --env=PATH=/app/bin:/usr/bin:/home/zhangtianci/.cargo/bin dev.zed.Zed
```

或者 Rust 项目换官方脚本装的 Zed，原生安装继承宿主 PATH，没这层沙箱。路径换成你的用户名，改完重启。

`enable_lsp_tasks` 让 cargo 任务（run / test）出现在任务列表和行号槽的运行按钮。

`initialization_options` 我动了三处。check.command 换成 clippy，保存时用 clippy 替代 check，lint 在写的时候就冒出来，不用等 CI 打回，clippy 比 check 慢一点，能接受。cargo.features 写成 all，分析时启用全部 feature，`#[cfg(feature = "…")]` 里的代码不会再被灰掉或误报，索引范围会变大。cargo.targetDir 开成 true，rust-analyzer 用 `target/rust-analyzer` 子目录，和终端里的 cargo build 各写各的，不然两个 cargo 进程抢一个 target 目录的锁，互相等。

内联提示：lifetimeElisionHints 用 skip_trivial，只在返回类型涉及生命周期时提示；closureReturnTypeHints 用 always，链式迭代器那种长到吓人的闭包返回类型直接标出来；maxLength 写成 null 不截断。diagnostics.experimental 开着，实验性诊断可能误报，烦就关。

大项目第一次打开要索引一会儿，之后是增量。

## 常用快捷键

Zed 原生键位（`base_keymap: "Zed"`）在 Linux 上的常用绑定：

| 快捷键 | 动作 |
| --- | --- |
| Ctrl+P | 文件跳转 |
| Ctrl+Shift+P | 命令面板 |
| Ctrl+Shift+F | 项目全局搜索 |
| Ctrl+Shift+E | 项目面板 |
| Ctrl+ 反引号键 | 终端面板 |
| Ctrl+/ | 注释切换 |
| Ctrl+Shift+Enter | 立即发送 Agent 消息 |

再补两个高频的：Ctrl+K Ctrl+T 打开主题选择器，Ctrl+K Ctrl+Shift+T 强制切换明暗主题；多光标按住 Alt 点击（`multi_cursor_modifier` 默认就是 Alt）。

想改键位，在命令面板执行 “zed: open keymap”，`keymap.json` 是覆盖式写法：

```jsonc
[
  {
    "bindings": {
      // 给格式化加一个顺手键
      "ctrl-alt-f": "editor::Format",
    },
  },
]
```

绑定还可以加 "context" 字段限定生效范围（比如只对编辑器生效），不写就是全局。动作名在 default keymap 里查，命令面板执行 “zed: open default keymap”。

# JDTLS 偏好配置全文

上一节的 java 块是我实际启用的配置。JDTLS 还有大量可调项：引用搜索（是否包含 getter / 反编译源码）、符号搜索、文件关联、Maven / Gradle 导入策略、折叠范围、成员排序、内容提供者等等。下面是完整的 java.* 偏好清单，中文注释版，对应的 Java 配置类为 `org.eclipse.jdt.ls.core.internal.preferences.Preferences`。整个文件作为 `lsp.jdtls.settings.initialization_options.settings` 的内容使用；里面的示例路径（/usr/local/jdk-17.0.1、/usr/lib/jvm/…、/path/to/… 等）按需替换。

```jsonc
{
  // =========================================================================
  //  Eclipse JDTLS (Java Development Tooling Language Server) 配置文件
  //  =========================================================================
  //  此文件采用 JSONC 格式（支持注释与尾随逗号）。
  //  配置项按功能模块分组，涵盖 JDK、编译、格式化、补全、项目导入等方面。
  //
  //  对应的 Java 配置类:
  //    org.eclipse.jdt.ls.core.internal.preferences.Preferences
  //
  //  所有设置均以前缀 `java.` 开头，可在 VS Code / Neovim / Zed 等支持
  //  JDTLS 的编辑器中使用。

  "java": {
    // =====================================================================
    //  JDK 与编辑行为
    // =====================================================================

    /// JDK 安装根目录的绝对路径。
    /// 类型: string | null
    /// 默认值: null
    "home": "/usr/local/jdk-17.0.1",

    "edit": {
      /// 智能分号检测。
      /// 启用后，输入分号时编辑器会智能地将其置于合适的位置（例如
      /// 自动跳过已存在的分号），而不是简单地追加。
      /// 类型: boolean
      /// 默认值: false
      "smartSemicolonDetection": {
        "enabled": true,
      },

      /// 编辑时是否验证所有已打开的缓冲区（触发生成诊断信息）。
      /// 关闭此选项可能略微提升编辑性能，但会导致部分文件的诊断信息滞后。
      /// 类型: boolean
      /// 默认值: true
      "validateAllOpenBuffersOnChanges": true,
    },

    // =====================================================================
    //  引用与符号搜索
    // =====================================================================

    /// 引用搜索相关配置
    "references": {
      /// 查找引用时是否包含 getter、setter、构造器及 Builder 方法。
      /// 类型: boolean
      /// 默认值: true
      "includeAccessors": true,

      /// 查找引用时是否包含反编译的 class 源码（如通过 FernFlower 反编译）。
      /// 类型: boolean
      /// 默认值: true
      "includeDecompiledSources": true,
    },

    /// 符号搜索（workspace/symbol、document/symbol）相关配置
    "symbols": {
      /// 在符号搜索中是否包含源文件内的方法声明。
      /// 类型: boolean
      /// 默认值: false
      "includeSourceMethodDeclarations": false,

      /// 在文档符号中是否包含由工具生成的代码。
      /// 例如 Lombok 注解生成的 getter/setter/构造器等。
      /// 类型: boolean
      /// 默认值: false
      "includeGeneratedCode": false,
    },

    // =====================================================================
    //  全局设置与文件关联
    // =====================================================================

    /// 全局 Java 设置文件（.prefs 格式，即 Eclipse 的项目属性导出文件）。
    /// 类型: string | null
    /// 默认值: null
    "settings": {
      "url": "/path/to/org.eclipse.jdt.core.prefs",
    },

    // =====================================================================
    //  代码格式化
    // =====================================================================

    "format": {
      /// 是否启用 JDTLS 内置格式化器。
      /// 类型: boolean
      /// 默认值: true
      "enabled": true,

      /// 键入时自动格式化（例如输入分号或闭合大括号时触发）。
      "onType": {
        /// 类型: boolean
        /// 默认值: false
        "enabled": false,
      },

      /// 按 Tab 键时插入空格而非制表符（\t）。
      /// 类型: boolean
      /// 默认值: true
      "insertSpaces": true,

      /// 一个制表符等价于多少个空格（即缩进宽度）。
      /// 类型: integer
      /// 默认值: 4
      "tabSize": 4,

      /// 格式化时是否包含注释区域的格式化。
      "comments": {
        /// 类型: boolean
        /// 默认值: true
        "enabled": true,
      },

      /// 外部格式化器配置文件（Eclipse Formatter XML 格式）。
      /// 该文件需由 Eclipse IDE 导出（Window → Preferences → Java → Code Style → Formatter）。
      "settings": {
        /// URL 或本地文件路径。
        /// 类型: string | null
        /// 默认值: null
        "url": "/path/to/eclipse-formatter.xml",

        /// 当配置文件包含多个格式化方案时，指定要使用的方案名称。
        /// 类型: string | null
        /// 默认值: null
        "profile": "MyProfile",
      },
    },

    /// 文件关联配置 —— 将自定义文件后缀视为 Java 源文件处理。
    /// 仅支持简单的 `*.扩展名` 模式（不支持 `?` 和嵌套 `*`，如 `*.myjava`）。
    /// 类型: object
    /// 默认值: {}
    "associations": {
      "*.myjava": "java",
    },

    // =====================================================================
    //  项目配置（JDK 运行时、Maven 设置、构建更新策略）
    // =====================================================================

    "configuration": {
      /// 当 `pom.xml` 或 `build.gradle` 等构建配置文件发生变化时，
      /// 如何处理项目自动更新。
      /// 可选值:
      ///   - "disabled"    : 禁用自动更新，需手动触发（通过命令或按钮）
      ///   - "interactive" : 检测到变化时弹出提示，由用户决定是否更新
      ///   - "automatic"   : 自动静默更新
      /// 类型: string
      /// 默认值: "interactive"
      "updateBuildConfiguration": "interactive",

      /// Java 执行环境（JDK 运行时）列表。
      /// 用于多版本项目或不同模块指定不同 JDK。
      /// 类型: array of object
      /// 默认值: []
      "runtimes": [
        {
          /// 执行环境名称，如 "JavaSE-11" / "JavaSE-17"
          "name": "JavaSE-11",
          /// JDK 安装路径
          "path": "/usr/lib/jvm/java-11-openjdk",
          /// 是否设为默认 JDK（可选）
          "default": true,
          /// 源代码附件的路径（可选）
          // "sources": "/path/to/src.zip",
          /// Javadoc 附件的路径（可选）
          // "javadoc": "https://docs.oracle.com/en/java/javase/11/docs/api",
        },
        {
          "name": "JavaSE-17",
          "path": "/usr/lib/jvm/java-17-openjdk",
        },
      ],

      /// Maven 相关配置
      "maven": {
        /// 用户级 `settings.xml` 文件路径。
        /// 通常位于 `~/.m2/settings.xml`。
        /// 类型: string | null
        /// 默认值: null
        "userSettings": "~/.m2/settings.xml",

        /// 全局 `settings.xml` 文件路径。
        /// 通常位于 `${MAVEN_HOME}/conf/settings.xml`。
        /// 类型: string | null
        /// 默认值: null
        "globalSettings": "/usr/share/maven/conf/settings.xml",

        /// Maven 生命周期映射文件路径。
        /// 用于控制未映射插件执行的处理方式。
        /// 类型: string | null
        /// 默认值: null（自动使用 workspace 下的 `lifecycle-mapping-metadata.xml`）
        "lifecycleMappings": null,

        /// 当 Maven 插件的某个执行步骤没有被生命周期映射覆盖时的处理策略。
        /// 枚举值（按严重程度升序）:
        ///   - "ignore"  : 静默忽略
        ///   - "log"     : 仅记录日志
        ///   - "info"    : 显示信息提示
        ///   - "warning" : 显示警告
        ///   - "error"   : 显示为错误
        /// 类型: string
        /// 默认值: "ignore"
        "notCoveredPluginExecutionSeverity": "warning",

        /// 默认的 Mojo（Maven 插件目标）执行动作。
        /// 当插件目标没有明确的生命周期映射时生效。
        /// 可选值:
        ///   - "ignore"  : 忽略
        ///   - "execute" : 执行
        ///   - "warn"    : 警告后忽略
        ///   - "error"   : 错误后忽略
        /// 类型: string
        /// 默认值: "ignore"
        "defaultMojoExecutionAction": "execute",
      },
    },

    // =====================================================================
    //  项目导入（Gradle / Maven）
    // =====================================================================

    "import": {
      /// Gradle 项目导入配置
      "gradle": {
        /// 是否启用 Gradle 项目导入器。
        /// 类型: boolean
        /// 默认值: true
        "enabled": true,

        /// 是否启用 Gradle 离线模式（`--offline`）。
        /// 离线模式下仅使用本地缓存，不会尝试从远程仓库下载依赖。
        "offline": {
          /// 类型: boolean
          /// 默认值: false
          "enabled": false,
        },

        /// 是否使用项目自带的 Gradle Wrapper（`gradlew`）。
        "wrapper": {
          /// 类型: boolean
          /// 默认值: true
          "enabled": true,

          /// 允许的 Gradle Wrapper JAR 文件的 SHA-256 校验和。
          /// 用于安全校验，防止执行被篡改的 Wrapper。
          /// 每项可以是一个字符串（仅校验和），或一个包含 `checksum` 与
          /// `allowed` 字段的对象。
          /// 类型: array of string | object
          /// 默认值: []
          "checksums": [],
        },

        /// 在不使用 Wrapper 时，指定要使用的 Gradle 版本号。
        /// 类型: string | null
        /// 默认值: null
        "version": null,

        /// 传递给 Gradle 的额外命令行参数。
        /// 例: `["--stacktrace", "--info"]`
        /// 类型: array of string
        /// 默认值: []
        "arguments": [],

        /// 传递给 Gradle 守护进程的 JVM 参数。
        /// 例: `["-Xmx2048m", "-Dfile.encoding=UTF-8"]`
        /// 类型: array of string
        /// 默认值: []
        "jvmArguments": [],

        /// Gradle 安装目录（`GRADLE_HOME`）。
        /// 类型: string | null
        /// 默认值: null
        "home": null,

        /// 用于运行 Gradle 守护进程的 JDK 路径。
        /// 可与项目编译所用的 JDK 不同。
        "java": {
          /// 类型: string | null
          /// 默认值: null
          "home": null,
        },

        /// Gradle 用户目录（`GRADLE_USER_HOME`）。
        /// 通常为 `~/.gradle`。
        "user": {
          /// 类型: string | null
          /// 默认值: null
          "home": null,
        },

        /// 是否启用 Gradle 注解处理器（annotationProcessor）。
        "annotationProcessing": {
          /// 类型: boolean
          /// 默认值: true
          "enabled": true,
        },
      },

      /// Maven 项目导入配置
      "maven": {
        /// 是否启用 Maven 项目导入器。
        /// 类型: boolean
        /// 默认值: true
        "enabled": true,

        /// 是否启用 Maven 离线模式（`--offline`）。
        /// 离线模式下仅使用本地 `~/.m2/repository` 缓存。
        "offline": {
          /// 类型: boolean
          /// 默认值: false
          "enabled": false,
        },

        /// 是否禁用 Maven 测试 classpath 标志。
        /// 启用此标志后，测试目录不会被标记为测试源码（即被视作普通源码），
        /// 这可能影响编译范围与依赖解析。
        /// 类型: boolean
        /// 默认值: false
        "disableTestClasspathFlag": false,
      },

      /// 导入项目时要排除的目录（ANT 风格 glob 模式）。
      /// 匹配的目录不会被导入为项目的一部分。
      /// 类型: array of string
      /// 默认值: ["**/node_modules/**", "**/.metadata/**", "**/archetype-resources/**", "**/META-INF/maven/**"]
      "exclusions": ["**/node_modules/**", "**/.metadata/**", "**/archetype-resources/**", "**/META-INF/maven/**"],
    },

    /// Maven 依赖管理相关配置（独立于项目导入）
    "maven": {
      /// 是否自动下载 Maven 依赖的源代码（sources.jar）。
      /// 类型: boolean
      /// 默认值: false
      "downloadSources": true,

      /// 是否每次同步时强制更新 SNAPSHOT 和 RELEASE 版本依赖。
      /// 类型: boolean
      /// 默认值: false
      "updateSnapshots": false,
    },

    /// Eclipse 项目特定配置
    "eclipse": {
      /// 是否自动下载 Eclipse 项目的源代码。
      /// 类型: boolean
      /// 默认值: false
      "downloadSources": false,
    },

    // =====================================================================
    //  Code Lens（代码透镜）
    // =====================================================================

    /// 是否在方法 / 字段上方显示引用计数。
    /// 类型: boolean
    /// 默认值: true
    "referencesCodeLens": {
      "enabled": true,
    },

    /// 接口 / 抽象类方法的实现显示策略。
    /// 可选值:
    ///   - "none"       : 不显示
    ///   - "all"        : 显示该接口/抽象方法的所有实现
    ///   - "overridden" : 仅显示被重写（有具体实现）的方法
    /// 类型: string
    /// 默认值: "none"
    "implementationCodeLens": "all",

    // =====================================================================
    //  保存操作与粘贴行为
    // =====================================================================

    "saveActions": {
      /// 保存文件时是否自动组织（排序并移除未使用的）导入语句。
      /// 类型: boolean
      /// 默认值: false
      "organizeImports": true,

      /// 保存时是否运行代码清理操作。
      /// 具体执行的清理项由 `java.cleanup.actions` 控制。
      /// 类型: boolean
      /// 默认值: false
      "cleanup": false,
    },

    /// 粘贴代码时是否自动添加所需导入语句并移除冲突导入。
    "updateImportsOnPaste": {
      /// 类型: boolean
      /// 默认值: true
      "enabled": true,
    },

    // =====================================================================
    //  签名帮助、悬停提示、重命名与命令执行
    // =====================================================================

    /// 方法签名帮助（在输入方法参数时显示参数列表与文档）。
    "signatureHelp": {
      /// 类型: boolean
      /// 默认值: false
      "enabled": true,

      /// 是否在签名帮助中显示方法的 Javadoc 概要描述。
      "description": {
        /// 类型: boolean
        /// 默认值: false
        "enabled": true,
      },
    },

    /// 鼠标悬停提示中是否显示 Javadoc。
    "hover": {
      "javadoc": {
        /// 类型: boolean
        /// 默认值: true
        "enabled": true,
      },
    },

    /// 是否启用符号重命名（refactor → rename）。
    "rename": {
      /// 类型: boolean
      /// 默认值: true
      "enabled": true,
    },

    /// 是否启用 `workspace/executeCommand`，即允许客户端执行
    /// JDTLS 提供的自定义命令（如应用重构、组织导入等）。
    "executeCommand": {
      /// 类型: boolean
      /// 默认值: true
      "enabled": true,
    },

    /// 是否启用自动构建（自动编译）。
    /// 关闭后需手动触发编译。
    "autobuild": {
      /// 类型: boolean
      /// 默认值: true
      "enabled": true,
    },

    /// 最大并发构建任务数。
    /// 类型: integer (>= 1)
    /// 默认值: 1（由 Eclipse 平台决定）
    "maxConcurrentBuilds": 2,

    // =====================================================================
    //  项目资源与隐式项目
    // =====================================================================

    "project": {
      /// 项目资源过滤器 —— 构建资源树时排除的文件/目录名称模式。
      /// 每项是一个正则表达式（而非 glob）。
      /// 类型: array of string
      /// 默认值: ["node_modules", "\\.git"]
      "resourceFilters": ["node_modules", "\\.git", "target"],

      /// 隐式项目（没有构建脚本的单文件 / 零散文件夹项目）的引用库配置。
      /// 类型: object | array of string
      /// 默认值: { "include": ["lib/**"], "exclude": [], "sources": {} }
      "referencedLibraries": {
        /// 要包含的库文件（glob 模式）
        "include": ["lib/**/*.jar", "**/libs/*.jar"],
        /// 要排除的库文件（glob 模式）
        "exclude": ["**/test-libs/*.jar"],
        /// 库文件到源码的映射（key = jar 的 glob， value = 源码路径）
        "sources": {
          "lib/foo.jar": "lib/foo-sources.jar",
        },
      },

      /// 隐式项目的编译输出路径。
      /// 类型: string | null
      /// 默认值: null
      "outputPath": "target/classes",

      /// 隐式项目的源码目录列表。
      /// 类型: array of string | null
      /// 默认值: null
      "sourcePaths": ["src", "gen"],

      /// 当项目没有显式指定编码时，如何处理编码问题。
      /// 可选值:
      ///   - "ignore"  : 忽略，使用系统默认编码
      ///   - "replace" : 强制使用工作区编码覆盖
      ///   - "prompt"  : 弹窗提示用户选择编码
      /// 类型: string
      /// 默认值: "ignore"
      "encoding": "ignore",
    },

    // =====================================================================
    //  代码补全
    // =====================================================================

    "completion": {
      /// 是否启用代码补全。
      /// 类型: boolean
      /// 默认值: true
      "enabled": true,

      /// 是否启用后置补全（如 `.var` / `.for` / `.sout` 等）。
      "postfix": {
        /// 类型: boolean
        /// 默认值: true
        "enabled": true,
      },

      /// 补全时的大小写匹配模式。
      /// 可选值:
      ///   - "off"         : 不区分大小写
      ///   - "firstLetter" : 仅首字母区分大小写
      ///   - "exact"       : 严格区分大小写
      /// 类型: string
      /// 默认值: "off"
      "matchCase": "off",

      /// 是否启用补全项的延迟解析（Lazy Resolve）。
      /// 启用后补全列表会更快返回，但单项的详细信息（如 Javadoc）
      /// 需要被选中时才加载，可显著提升弱网或低性能环境下的体验。
      "lazyResolveTextEdit": {
        /// 类型: boolean
        /// 默认值: false
        "enabled": true,
      },

      /// 补全文本是覆盖当前光标后文本（true）还是直接插入（false）。
      /// 类型: boolean
      /// 默认值: true
      "overwrite": true,

      /// 从补全列表中选择方法后，如何处理方法参数。
      /// 可选值:
      ///   - "insertParameterNames"           : 插入参数名作为占位符
      ///   - "insertBestGuessedArguments"     : 插入 JDTLS 最佳猜测的实际参数
      ///   - "insertBestGuessedArgumentsWithType": 插入带类型的最佳猜测参数
      ///   - true  (已弃用) : 等同于 "insertBestGuessedArguments"
      ///   - false (已弃用) : 等同于 "insertParameterNames"
      /// 类型: boolean | string
      /// 默认值: "insertParameterNames"
      "guessMethodArguments": "insertBestGuessedArguments",

      /// 是否折叠补全列表中的重载方法（将同名方法合并为一条）。
      /// 类型: boolean
      /// 默认值: false
      "collapseCompletionItems": false,

      /// 补全列表最多返回的条数（不包括代码片段和 Javadoc 提案）。
      /// 0 表示无限制。推荐设置在 50 ~ 200 之间。
      /// 类型: integer
      /// 默认值: 50
      "maxResults": 100,

      /// 收藏的静态成员 —— 这些成员在补全列表中会总是优先显示。
      /// 格式: `全限定类名.*` 或 `全限定类名.方法名`
      /// 类型: array of string
      /// 默认值:
      ///   ["org.junit.Assert.*", "org.junit.Assume.*",
      ///    "org.junit.jupiter.api.Assertions.*", "...Assumptions.*",
      ///    "...DynamicContainer.*", "...DynamicTest.*"]
      "favoriteStaticMembers": [
        "org.junit.Assert.*",
        "org.junit.Assume.*",
        "org.junit.jupiter.api.Assertions.*",
        "org.junit.jupiter.api.Assumptions.*",
        "org.junit.jupiter.api.DynamicContainer.*",
        "org.junit.jupiter.api.DynamicTest.*",
        "java.util.Objects.*",
        "java.util.Collections.*",
      ],

      /// 组织导入时各包（package）的排序顺序。
      /// 排在越前的包越先出现在 import 区块中。
      /// 类型: array of string
      /// 默认值: ["java", "javax", "org", "com"]
      "importOrder": ["java", "javax", "org", "com", "io"],

      /// 补全列表中要过滤（隐藏）的类型列表。
      /// 支持通配符 `*`，如 `com.sun.*`。
      /// 类型: array of string
      /// 默认值:
      ///   ["com.sun.*", "io.micrometer.shaded.*", "java.awt.*",
      ///    "jdk.*", "org.graalvm.*", "sun.*"]
      "filteredTypes": ["com.sun.*", "io.micrometer.shaded.*", "java.awt.*", "jdk.*", "org.graalvm.*", "sun.*"],

      /// 是否启用链式补全。
      /// 启用后，在输入 `a.b().` 时会自动探测 `.b()` 返回类型的可用方法
      /// 并纳入补全列表，无需手动逐级展开。
      "chain": {
        /// 类型: boolean
        /// 默认值: false
        "enabled": true,
      },
    },

    // =====================================================================
    //  折叠范围、选择范围与成员排序
    // =====================================================================

    /// 是否启用代码折叠（foldingRange）。
    "foldingRange": {
      /// 类型: boolean
      /// 默认值: true
      "enabled": true,
    },

    /// 是否启用智能选择扩展（selectionRange）。
    /// 即双击/快捷键逐步扩展选区至更大的语法节点。
    "selectionRange": {
      /// 类型: boolean
      /// 默认值: true
      "enabled": true,
    },

    /// 成员的排序顺序（用于"排序成员"等代码操作）。
    /// 一个逗号分隔的排序标识符字符串，标识符含义如下:
    ///   T   — 类型（内部类/接口）
    ///   C   — 构造器
    ///   I   — 初始化器
    ///   M   — 方法
    ///   F   — 字段
    ///   SI  — 静态初始化器
    ///   SM  — 静态方法
    ///   SF  — 静态字段
    /// 类型: string
    /// 默认值: "T,SF,SI,SM,F,I,C,M"（Eclipse 内置默认值）
    "memberSortOrder": "T,SF,SI,SM,F,I,C,M",

    // =====================================================================
    //  组织导入 & 内容提供者
    // =====================================================================

    "sources": {
      "organizeImports": {
        /// 当来自同一包的普通导入数量超过此阈值时，压缩为星号导入
        /// （如 `import java.util.*`）。
        /// 类型: integer
        /// 默认值: 99
        "starThreshold": 50,

        /// 当来自同一包的静态导入数量超过此阈值时，压缩为静态星号导入
        /// （如 `import static org.mockito.Mockito.*`）。
        /// 类型: integer
        /// 默认值: 99
        "staticStarThreshold": 10,
      },
    },

    /// 首选的内容提供者 ID 列表。
    /// 用于指定特定功能（如代码折叠、大纲视图）使用的内容提供者。
    /// 类型: array of string | null
    /// 默认值: null
    "contentProvider": {
      "preferred": null,
    },

    // =====================================================================
    //  代码生成
    // =====================================================================

    "codeGeneration": {
      /// hashCode / equals 方法生成策略
      "hashCodeEquals": {
        /// 是否使用 Java 7 引入的 `Objects.hash(...)` 和 `Objects.equals(...)`。
        /// 关掉则生成传统的自实现 hash 算法。
        /// 类型: boolean
        /// 默认值: false
        "useJava7Objects": true,

        /// 在 `equals` 方法中是否使用 `instanceof` 来比较类型
        /// （而非 `getClass()`）。
        /// 使用 `instanceof` 允许子类相等性判断，更符合 Liskov 替换原则。
        /// 类型: boolean
        /// 默认值: false
        "useInstanceof": true,
      },

      /// 生成方法体时是否为 `if` / `for` 等控制流语句使用块（`{ }`）。
      /// 类型: boolean
      /// 默认值: false
      "useBlocks": true,

      /// 是否在生成的方法上生成 Javadoc 注释。
      /// 类型: boolean
      /// 默认值: false
      "generateComments": true,

      /// 生成代码的插入位置。
      /// 可选值:
      ///   - "lastMember"   : 类/接口的最后一个成员之后
      ///   - "afterCursor"  : 当前光标位置之后
      ///   - "beforeCursor" : 当前光标位置之前
      /// 类型: string | null
      /// 默认值: null（由 Eclipse 内部策略决定）
      "insertionLocation": "lastMember",

      /// 是否为代码生成所创建的新字段添加 `final` 修饰符。
      /// 可选值:
      ///   - "none"    : 不添加 final
      ///   - "all"     : 所有新字段都声明为 final
      ///   - "private" : 仅 private 字段声明为 final
      ///   - "package" : package 可见性及更窄可见性的字段声明为 final
      /// 类型: string
      /// 默认值: "none"
      "addFinalForNewDeclaration": "none",

      /// toString 方法生成配置
      "toString": {
        /// 生成的 toString 使用的模板。
        /// 可选值:
        ///   - "STRING_BUILDER"            : 使用 StringBuilder 拼接
        ///   - "STRING_BUILDER_CHAINED"    : 使用链式 StringBuilder
        ///   - "STRING_FORMAT"             : 使用 String.format()
        ///   - "TO_STRING_BUILDER"         : Apache Commons Lang3 ToStringBuilder
        ///   - "SPRING_TO_STRING_CREATOR"  : Spring ToStringCreator
        ///   - "GUAVA_TO_STRING_HELPER"    : Guava MoreObjects.toStringHelper
        ///   - "OBJECTS_TO_STRING"         : Java 7 Objects.toString()
        /// 类型: string | null
        /// 默认值: null
        "template": "STRING_BUILDER",

        /// 代码风格（决定生成逻辑的实现方式）。
        /// 可选值:
        ///   - "Eclipse" : Eclipse 内置风格（StringBuilder）
        ///   - "Apache"  : Apache Commons Lang
        ///   - "Guava"   : Google Guava
        ///   - "Spring"  : Spring Framework
        ///   - "Custom"  : 使用自定义模板（需同时指定 template）
        /// 类型: string | null
        /// 默认值: null
        "codeStyle": "Eclipse",

        /// toString 输出中是否跳过值为 null 的字段。
        /// 类型: boolean
        /// 默认值: false
        "skipNullValues": true,

        /// 是否列出数组 / 集合的内容（而非仅显示其引用地址）。
        /// 类型: boolean
        /// 默认值: true
        "listArrayContents": true,

        /// toString 输出中数组 / 集合的最大元素数量。
        /// 0 表示无限制。
        /// 类型: integer
        /// 默认值: 0
        "limitElements": 10,
      },
    },

    // =====================================================================
    //  文件模板
    // =====================================================================

    "templates": {
      /// 新建 Java 文件时自动插入的文件头注释。
      /// 每行一个字符串，可使用 `${user}`、`${date}`、`${time}` 等变量。
      /// 类型: array of string
      /// 默认值: []
      "fileHeader": ["/*", " * Copyright © ${year} My Company. All rights reserved.", " */"],

      /// 新建 Java 类型（类/接口/枚举等）时自动插入的类型注释。
      /// 类型: array of string
      /// 默认值: []
      "typeComment": ["/**", " * @author ${user}", " * @since ${date}", " */"],
    },

    // =====================================================================
    //  遥测与诊断
    // =====================================================================

    /// 是否允许 JDTLS 收集匿名遥测数据。
    /// 类型: boolean
    /// 默认值: false
    "telemetry": {
      "enabled": false,
    },

    /// 诊断过滤器 —— 匹配正则表达式的诊断信息将被隐藏。
    /// 每项是一个正则表达式（Java Pattern 语法）。
    /// 类型: array of string
    /// 默认值: []
    "diagnostic": {
      "filter": [],
    },

    // =====================================================================
    //  内嵌提示（Inlay Hints）
    // =====================================================================

    "inlayHints": {
      /// 参数名称内嵌提示。
      "parameterNames": {
        /// 显示模式:
        ///   - "none"     : 不显示
        ///   - "literals" : 仅当实参为字面量（如 `true`、`42`、`"hello"`）时显示
        ///   - "all"      : 始终显示所有参数的名称
        /// 类型: string
        /// 默认值: "literals"
        "enabled": "literals",

        /// 当参数名与传入的变量名完全一致且仅带数字后缀时，
        /// 是否抑制提示。
        /// 例: `foo(index1)` 传入 `int index1` → 抑制提示。
        /// 类型: boolean
        /// 默认值: false
        "suppressWhenSameNameNumbered": false,

        /// 排除列表 —— 不显示参数名提示的方法模式。
        /// 格式: `全限定类名#方法名(参数类型...)`，支持通配符。
        /// 例: `java.lang.*`, `*#equals(java.lang.Object)`
        /// 类型: array of string
        /// 默认值: []
        "exclusions": [],
      },

      /// 是否显示局部变量及字段的类型内嵌提示。
      /// 例: `var foo = bar();` → 显示 `<String> foo = bar();`
      "variableTypes": {
        /// 类型: boolean
        /// 默认值: false
        "enabled": true,
      },

      /// 是否在方法调用处显示参数类型提示。
      /// 例: `process(null, null)` → `process(String name, Integer value)`
      "parameterTypes": {
        /// 类型: boolean
        /// 默认值: false
        "enabled": false,
      },

      /// 是否在格式化方法（`String#format`, `Console#printf` 等）的参数
      /// 旁显示其对应的格式化占位符（`%s`, `%d` 等）。
      "formatParameters": {
        /// 类型: boolean
        /// 默认值: false
        "enabled": false,
      },
    },

    // =====================================================================
    //  代码操作
    // =====================================================================

    "codeAction": {
      /// 排序成员时的行为调整。
      "sortMembers": {
        /// 是否避免易失性（volatile）更改。
        /// 开启后，排序成员时不会改变字段的相对顺序，
        /// 防止因字段顺序变化导致的序列化 / 反射问题。
        /// 类型: boolean
        /// 默认值: true
        "avoidVolatileChanges": true,
      },
    },

    // =====================================================================
    //  JDT 语言服务器特性开关
    // =====================================================================

    "jdt": {
      "ls": {
        /// 是否启用 Protobuf（Protocol Buffers）文件支持。
        /// 类型: boolean
        /// 默认值: false
        "protobufSupport": {
          "enabled": true,
        },

        /// 是否启用 Android 项目支持。
        /// 类型: boolean
        /// 默认值: false
        "androidSupport": {
          "enabled": false,
        },

        /// 是否启用 AspectJ 语法支持。
        /// 类型: boolean
        /// 默认值: false
        "aspectjSupport": {
          "enabled": false,
        },

        /// 是否启用 Kotlin 语言支持（实验性）。
        /// 类型: boolean
        /// 默认值: false
        "kotlinSupport": {
          "enabled": false,
        },

        /// 是否启用 Groovy 语言支持（实验性）。
        /// 类型: boolean
        /// 默认值: false
        "groovySupport": {
          "enabled": false,
        },

        /// 是否启用 Scala 语言支持（实验性）。
        /// 类型: boolean
        /// 默认值: false
        "scalaSupport": {
          "enabled": false,
        },

        /// 是否启用 javac 编译器（实验性）。
        /// 开启后使用 javac 而非 Eclipse JDT 内置编译器进行编译，
        /// 可获得与标准 javac 完全一致的编译行为。
        /// 类型: boolean
        /// 默认值: false
        "javac": {
          "enabled": false,
        },
      },
    },

    // =====================================================================
    //  编译时注解与空值分析
    // =====================================================================

    "compile": {
      "nullAnalysis": {
        /// 用户自定义的 `@Nonnull` 注解类型（全限定名）。
        /// 用于空值分析，检测可能为 null 的赋值/传递。
        /// 类型: array of string
        /// 默认值: []
        "nonnull": ["javax.annotation.Nonnull", "org.eclipse.jdt.annotation.NonNull"],

        /// 用户自定义的 `@Nullable` 注解类型（全限定名）。
        /// 标记可为 null 的类型。
        /// 类型: array of string
        /// 默认值: []
        "nullable": ["javax.annotation.Nullable", "org.eclipse.jdt.annotation.Nullable"],

        /// 用户自定义的 `@NonNullByDefault` 注解类型（全限定名）。
        /// 标记包或类级别默认为非空。
        /// 类型: array of string
        /// 默认值: []
        "nonnullbydefault": [
          "javax.annotation.ParametersAreNonnullByDefault",
          "org.eclipse.jdt.annotation.NonNullByDefault",
        ],

        /// 空值分析模式。
        /// 可选值:
        ///   - "disabled"    : 禁用空值分析
        ///   - "interactive" : 检测到注解时提示用户是否启用分析
        ///   - "automatic"   : 检测到注解后自动启用分析
        /// 类型: string
        /// 默认值: "disabled"
        "mode": "automatic",
      },
    },

    // =====================================================================
    //  清理操作与重构
    // =====================================================================

    /// 保存时要运行的清理操作列表。
    /// 需要同时启用 `java.saveActions.cleanup`。
    /// 常用值:
    ///   - "qualifyMembers"       : 为成员添加 this. 前缀
    ///   - "addMissingAnnotations": 添加缺失的注解（如 @Override）
    ///   - "qualifyStaticMembers" : 为静态成员添加类名前缀
    /// 类型: array of string
    /// 默认值: []
    "cleanup": {
      "actions": [],
    },

    /// 重构行为配置
    "refactoring": {
      "extract": {
        "interface": {
          /// 提取接口时是否将原类所有引用替换为新接口类型。
          /// 类型: boolean
          /// 默认值: false
          "replace": false,
        },
      },
    },

    // =====================================================================
    //  快速修复与搜索
    // =====================================================================

    /// 快速修复建议的显示级别。
    /// 可选值:
    ///   - "line"    : 按行显示快速修复（点击行号旁的灯泡）
    ///   - "problem" : 直接在问题标记处显示快速修复
    /// 类型: string
    /// 默认值: "line"
    "quickfix": {
      "showAt": "line",
    },

    /// 代码搜索的作用域。
    /// 可选值:
    ///   - "all"  : 搜索所有代码（主代码 + 测试代码）
    ///   - "main" : 仅搜索主代码，排除测试目录
    /// 类型: string
    /// 默认值: "all"
    "search": {
      "scope": "all",
    },
  },
}
```

# 格式化器：Eclipse Formatter XML 全文

格式化规则我交给一份 Eclipse Formatter XML，而不是在 Zed 里零散配置。好处是同一份文件在 Eclipse、IDEA（导入）、JDTLS 之间通用，团队共享也方便。文件可以从 Eclipse 导出（Window → Preferences → Java → Code Style → Formatter），也可以直接用下面这份：保存为 `eclipse-java-formatter.xml`（我放在 `~/.config/formatter/`），把绝对路径填进 `java.format.settings.url`。

核心行为：空格缩进每级 4 格、行宽 120、续行缩进 2 级、已换行的行不自动合并。全文 1613 行，三百多个 setting，每个都带中文注释，微调时直接改 XML。

```xml
<?xml version="1.0" encoding="utf-8" ?>
<profiles version="21">
	<profile kind="CodeFormatterProfile" name="Default" version="21">

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                      缩进 (Indentation)                      ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 制表符类型：使用空格还是制表符进行缩进 -->
		<!-- 可选值：space（空格）、tab（制表符）、mixed（混合） -->
		<setting id="org.eclipse.jdt.core.formatter.tabulation.char" value="space" />

		<!-- 仅在行首缩进处使用制表符，其余位置用空格 -->
		<!-- 仅当 tabulation.char 为 mixed 时生效 -->
		<!-- 可选值：true、false -->
		<setting id="org.eclipse.jdt.core.formatter.use_tabs_only_for_leading_indentations" value="false" />

		<!-- 缩进大小：每个缩进级别对应的空格数 -->
		<setting id="org.eclipse.jdt.core.formatter.indentation.size" value="4" />

		<!-- 制表符大小：一个制表符对应的空格数 -->
		<setting id="org.eclipse.jdt.core.formatter.tabulation.size" value="4" />

		<!-- 文本块（Text Block，Java 13+）的缩进级别数 -->
		<!-- 0 = 不缩进，1 = 缩进一级，2 = 缩进两级 -->
		<setting id="org.eclipse.jdt.core.formatter.text_block_indentation" value="1" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 缩进比较 (Indent Comparison)                 ║ -->
		<!-- ║              控制各类声明体相对于其声明头的缩进              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 类型声明（class/interface）的成员是否相对于类型头缩进 -->
		<!-- 可选值：true（缩进）、false（不缩进） -->
		<setting id="org.eclipse.jdt.core.formatter.indent_body_declarations_compare_to_type_header" value="true" />

		<!-- 枚举声明（enum）的成员是否相对于枚举声明头缩进 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_body_declarations_compare_to_enum_declaration_header" value="true" />

		<!-- 枚举常量的成员是否相对于枚举常量头缩进 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_body_declarations_compare_to_enum_constant_header" value="true" />

		<!-- 注解类型声明的成员是否相对于注解类型声明头缩进 -->
		<setting
            id="org.eclipse.jdt.core.formatter.indent_body_declarations_compare_to_annotation_declaration_header"
            value="true"
        />

		<!-- Record 声明的成员是否相对于 Record 头缩进 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_body_declarations_compare_to_record_header" value="true" />

		<!-- 语句是否相对于方法体缩进 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_statements_compare_to_body" value="true" />

		<!-- 语句是否相对于代码块缩进 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_statements_compare_to_block" value="true" />

		<!-- switch 语句体是否相对于 switch 缩进 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_switchstatements_compare_to_switch" value="true" />

		<!-- case 分支内的语句是否相对于 case 标签缩进 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_switchstatements_compare_to_cases" value="true" />

		<!-- break 语句是否相对于 case 标签缩进 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_breaks_compare_to_cases" value="true" />

		<!-- 是否缩进空行 -->
		<setting id="org.eclipse.jdt.core.formatter.indent_empty_lines" value="false" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                      对齐 (Alignment)                        ║ -->
		<!-- ║      控制类型成员、变量声明、赋值语句等是否按列对齐          ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 类型成员（字段、方法等）是否按列对齐 -->
		<!-- 可选值：true（按列对齐）、false（不对齐） -->
		<setting id="org.eclipse.jdt.core.formatter.align_type_members_on_columns" value="false" />

		<!-- 变量声明是否按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.align_variable_declarations_on_columns" value="false" />

		<!-- 赋值语句是否按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.align_assignment_statements_on_columns" value="false" />

		<!-- 对齐时是否使用空格填充（而非制表符） -->
		<setting id="org.eclipse.jdt.core.formatter.align_with_spaces" value="true" />

		<!-- 字段分组对齐时的空行间隔数 -->
		<!-- 0 = 不按空行分组，1 = 每隔 1 个空行分组，2147483647 = 所有字段同组 -->
		<setting id="org.eclipse.jdt.core.formatter.align_fields_grouping_blank_lines" value="1" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║               大括号位置 (Brace Position)                    ║ -->
		<!-- ║   可选值：end_of_line / next_line / next_line_shifted /      ║ -->
		<!-- ║           next_line_on_wrap                                  ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 类型声明（class/interface）的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_type_declaration" value="end_of_line" />

		<!-- 匿名类型声明的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_anonymous_type_declaration" value="end_of_line" />

		<!-- 构造器声明的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_constructor_declaration" value="end_of_line" />

		<!-- 方法声明的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_method_declaration" value="end_of_line" />

		<!-- 枚举声明的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_enum_declaration" value="end_of_line" />

		<!-- 枚举常量的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_enum_constant" value="end_of_line" />

		<!-- Record 声明的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_record_declaration" value="end_of_line" />

		<!-- Record 紧凑构造器的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_record_constructor" value="end_of_line" />

		<!-- 注解类型声明的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_annotation_type_declaration" value="end_of_line" />

		<!-- 代码块的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_block" value="end_of_line" />

		<!-- case 分支中代码块的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_block_in_case" value="end_of_line" />

		<!-- switch 语句的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_switch" value="end_of_line" />

		<!-- 数组初始化器的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_array_initializer" value="end_of_line" />

		<!-- 空数组初始化器是否保持在同一行 -->
		<!-- 可选值：true（同一行）、false（不强制） -->
		<setting id="org.eclipse.jdt.core.formatter.keep_empty_array_initializer_on_one_line" value="true" />

		<!-- Lambda 表达式体的左大括号位置 -->
		<setting id="org.eclipse.jdt.core.formatter.brace_position_for_lambda_body" value="end_of_line" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║             括号位置 (Parentheses Positions)                 ║ -->
		<!-- ║   可选值：preserve_positions / common_lines /                ║ -->
		<!-- ║           separate_lines_if_wrapped / separate_lines /       ║ -->
		<!-- ║           force_except_singleton                             ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 方法声明中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_method_delcaration" value="preserve_positions" />

		<!-- 方法调用中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_method_invocation" value="preserve_positions" />

		<!-- 枚举常量声明中括号的位置 -->
		<setting
            id="org.eclipse.jdt.core.formatter.parentheses_positions_in_enum_constant_declaration"
            value="preserve_positions"
        />

		<!-- Record 声明中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_record_declaration" value="preserve_positions" />

		<!-- 注解中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_annotation" value="preserve_positions" />

		<!-- Lambda 声明中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_lambda_declaration" value="preserve_positions" />

		<!-- if/while 语句中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_if_while_statement" value="preserve_positions" />

		<!-- for 语句中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_for_statment" value="preserve_positions" />

		<!-- switch 语句中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_switch_statement" value="preserve_positions" />

		<!-- try 子句中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_try_clause" value="preserve_positions" />

		<!-- catch 子句中括号的位置 -->
		<setting id="org.eclipse.jdt.core.formatter.parentheses_positions_in_catch_clause" value="preserve_positions" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - 类型声明                              ║ -->
		<!-- ║   以下 insert_space_* 设置的 value 可选：                    ║ -->
		<!-- ║   insert（插入空格）、do not insert（不插入空格）            ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 类型声明左大括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_type_declaration" value="insert" />

		<!-- 匿名类型声明左大括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_anonymous_type_declaration"
            value="insert"
        />

		<!-- implements/extends 列表中逗号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_superinterfaces" value="do not insert" />

		<!-- implements/extends 列表中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_superinterfaces" value="insert" />

		<!-- 多字段声明中逗号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_multiple_field_declarations"
            value="do not insert"
        />

		<!-- 多字段声明中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_multiple_field_declarations" value="insert" />

		<!-- 多局部变量声明中逗号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_multiple_local_declarations"
            value="do not insert"
        />

		<!-- 多局部变量声明中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_multiple_local_declarations" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - 构造器声明                            ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 构造器声明左小括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_constructor_declaration"
            value="do not insert"
        />

		<!-- 构造器声明左小括号后是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_constructor_declaration"
            value="do not insert"
        />

		<!-- 构造器声明右小括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_constructor_declaration"
            value="do not insert"
        />

		<!-- 构造器声明空参数列表的小括号之间是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_between_empty_parens_in_constructor_declaration"
            value="do not insert"
        />

		<!-- 构造器声明左大括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_constructor_declaration"
            value="insert"
        />

		<!-- 构造器参数列表中逗号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_constructor_declaration_parameters"
            value="do not insert"
        />

		<!-- 构造器参数列表中逗号后是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_constructor_declaration_parameters"
            value="insert"
        />

		<!-- 构造器 throws 子句中逗号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_constructor_declaration_throws"
            value="do not insert"
        />

		<!-- 构造器 throws 子句中逗号后是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_constructor_declaration_throws"
            value="insert"
        />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - 方法声明                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 方法声明左小括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_method_declaration"
            value="do not insert"
        />

		<!-- 方法声明左小括号后是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_method_declaration"
            value="do not insert"
        />

		<!-- 方法声明右小括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_method_declaration"
            value="do not insert"
        />

		<!-- 方法声明空参数列表的小括号之间是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_between_empty_parens_in_method_declaration"
            value="do not insert"
        />

		<!-- 方法声明左大括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_method_declaration" value="insert" />

		<!-- 方法参数列表中逗号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_method_declaration_parameters"
            value="do not insert"
        />

		<!-- 方法参数列表中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_method_declaration_parameters" value="insert" />

		<!-- 可变参数（...）省略号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_ellipsis" value="do not insert" />

		<!-- 可变参数（...）省略号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_ellipsis" value="insert" />

		<!-- 方法 throws 子句中逗号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_method_declaration_throws"
            value="do not insert"
        />

		<!-- 方法 throws 子句中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_method_declaration_throws" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - 标签语句                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 标签语句（label:）冒号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_colon_in_labeled_statement" value="do not insert" />

		<!-- 标签语句（label:）冒号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_colon_in_labeled_statement" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - 注解 (Annotation)                     ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 注解中 @ 符号后是否插入空格（如 @Override → 不插入） -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_at_in_annotation" value="do not insert" />

		<!-- 注解左小括号前是否插入空格（如 @SuppressWarnings("...")） -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_annotation" value="do not insert" />

		<!-- 注解左小括号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_annotation" value="do not insert" />

		<!-- 注解参数中逗号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_annotation" value="do not insert" />

		<!-- 注解参数中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_annotation" value="insert" />

		<!-- 注解右小括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_annotation" value="do not insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - 枚举声明 (Enum)                       ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 枚举声明左大括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_enum_declaration" value="insert" />

		<!-- 枚举常量列表中逗号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_enum_declarations" value="do not insert" />

		<!-- 枚举常量列表中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_enum_declarations" value="insert" />

		<!-- 枚举常量左小括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_enum_constant" value="do not insert" />

		<!-- 枚举常量左小括号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_enum_constant" value="do not insert" />

		<!-- 枚举常量空参数列表的小括号之间是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_between_empty_parens_in_enum_constant" value="do not insert" />

		<!-- 枚举常量参数中逗号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_enum_constant_arguments"
            value="do not insert"
        />

		<!-- 枚举常量参数中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_enum_constant_arguments" value="insert" />

		<!-- 枚举常量右小括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_enum_constant" value="do not insert" />

		<!-- 枚举常量左大括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_enum_constant" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║            空格 - 注解类型声明 (Annotation Type)             ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 注解类型声明中 @ 前是否插入空格（如 @interface → 前加空格） -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_at_in_annotation_type_declaration" value="insert" />

		<!-- 注解类型声明中 @ 后是否插入空格（如 @ interface → 不插入） -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_at_in_annotation_type_declaration"
            value="do not insert"
        />

		<!-- 注解类型声明左大括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_annotation_type_declaration"
            value="insert"
        />

		<!-- 注解类型成员声明左小括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_annotation_type_member_declaration"
            value="do not insert"
        />

		<!-- 注解类型成员声明空参数列表的小括号之间是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_between_empty_parens_in_annotation_type_member_declaration"
            value="do not insert"
        />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - Record 声明                           ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- Record 声明左小括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_record_declaration"
            value="do not insert"
        />

		<!-- Record 声明左小括号后是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_record_declaration"
            value="do not insert"
        />

		<!-- Record 组件列表中逗号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_record_components" value="do not insert" />

		<!-- Record 组件列表中逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_record_components" value="insert" />

		<!-- Record 声明右小括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_record_declaration"
            value="do not insert"
        />

		<!-- Record 声明左大括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_record_declaration" value="insert" />

		<!-- Record 紧凑构造器左大括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_record_constructor" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - Lambda 表达式                         ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- Lambda 箭头（->）前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_lambda_arrow" value="insert" />

		<!-- Lambda 箭头（->）后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_lambda_arrow" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                 空格 - 代码块 (Block)                        ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 代码块左大括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_block" value="insert" />

		<!-- 代码块右大括号后是否插入空格（如 } else、} catch） -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_closing_brace_in_block" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║              空格 - 控制语句 (Control Flow)                  ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- if 语句左小括号前是否插入空格（如 if ( → if后加空格） -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_if" value="insert" />

		<!-- if 语句左小括号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_if" value="do not insert" />

		<!-- if 语句右小括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_if" value="do not insert" />

		<!-- for 语句左小括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_for" value="insert" />

		<!-- for 循环左括号后是否插入空格。示例：for ( int i... vs for (int i... -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_for" value="do not insert" />

		<!-- for 循环右括号前是否插入空格。示例：for (...int i ) vs for (...int i) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_for" value="do not insert" />

		<!-- for 循环初始化部分逗号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_for_inits" value="do not insert" />

		<!-- for 循环初始化部分逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_for_inits" value="insert" />

		<!-- for 循环递增部分逗号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_for_increments" value="do not insert" />

		<!-- for 循环递增部分逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_for_increments" value="insert" />

		<!-- for 循环分号前是否插入空格。示例：for (int i = 0 ; ...) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_semicolon_in_for" value="do not insert" />

		<!-- for 循环分号后是否插入空格。示例：for (int i = 0; int j...) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_semicolon_in_for" value="insert" />

		<!-- 增强 for 中冒号前是否插入空格。示例：for (String s : list) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_colon_in_for" value="insert" />

		<!-- 增强 for 中冒号后是否插入空格。示例：for (String s : list) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_colon_in_for" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                   switch / case 语句                         ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- case 标签冒号前是否插入空格。示例：case 1: vs case 1 : -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_colon_in_case" value="do not insert" />

		<!-- default 标签冒号前是否插入空格。示例：default: vs default : -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_colon_in_default" value="do not insert" />

		<!-- switch case 箭头（Java 14+）前是否插入空格。示例：case 1 -> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_arrow_in_switch_case" value="insert" />

		<!-- switch case 箭头（Java 14+）后是否插入空格。示例：-> value -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_arrow_in_switch_case" value="insert" />

		<!-- switch default 箭头（Java 14+）前是否插入空格。示例：default -> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_arrow_in_switch_default" value="insert" />

		<!-- switch default 箭头（Java 14+）后是否插入空格。示例：-> value -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_arrow_in_switch_default" value="insert" />

		<!-- case 标签冒号后是否插入空格。示例：case 1: break vs case 1:break -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_colon_in_case" value="insert" />

		<!-- switch case 表达式（Java 14+ 多标签 case）逗号前是否插入空格。示例：case A, B -> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_switch_case_expressions"
            value="do not insert"
        />

		<!-- switch case 表达式（Java 14+ 多标签 case）逗号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_switch_case_expressions" value="insert" />

		<!-- switch 关键字与左括号之间是否插入空格。示例：switch (x) vs switch(x) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_switch" value="insert" />

		<!-- switch 左括号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_switch" value="do not insert" />

		<!-- switch 右括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_switch" value="do not insert" />

		<!-- switch 左花括号前是否插入空格。示例：switch (x) { vs switch (x){ -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_switch" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                      while 循环                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- while 关键字与左括号之间是否插入空格。示例：while (cond) vs while(cond) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_while" value="insert" />

		<!-- while 左括号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_while" value="do not insert" />

		<!-- while 右括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_while" value="do not insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                    synchronized 块                           ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- synchronized 关键字与左括号之间是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_synchronized" value="insert" />

		<!-- synchronized 左括号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_synchronized" value="do not insert" />

		<!-- synchronized 右括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_synchronized" value="do not insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                    try / catch 语句                          ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- try-with-resources 中 try 关键字与左括号之间是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_try" value="insert" />

		<!-- try-with-resources 左括号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_try" value="do not insert" />

		<!-- try-with-resources 分号前是否插入空格。示例：try (Res a ; Res b) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_semicolon_in_try_resources" value="do not insert" />

		<!-- try-with-resources 分号后是否插入空格。示例：try (Res a; Res b) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_semicolon_in_try_resources" value="insert" />

		<!-- try-with-resources 右括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_try" value="do not insert" />

		<!-- catch 关键字与左括号之间是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_catch" value="insert" />

		<!-- catch 左括号后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_catch" value="do not insert" />

		<!-- catch 右括号前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_catch" value="do not insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                   assert / return / throw                    ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- assert 语句冒号前是否插入空格。示例：assert condition : message -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_colon_in_assert" value="insert" />

		<!-- assert 语句冒号后是否插入空格。示例：assert condition : message -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_colon_in_assert" value="insert" />

		<!-- return 语句中括号表达式前是否插入空格。示例：return (value) vs return(value) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_parenthesized_expression_in_return" value="insert" />

		<!-- throw 语句中括号表达式前是否插入空格。示例：throw (exception) vs throw(exception) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_parenthesized_expression_in_throw" value="insert" />

		<!-- 语句末尾分号前是否插入空格。示例：int i ; vs int i; -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_semicolon" value="do not insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                        方法调用                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 方法调用时方法名与左括号之间是否插入空格。示例：foo (arg) vs foo(arg) -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_method_invocation"
            value="do not insert"
        />

		<!-- 方法调用左括号后是否插入空格。示例：foo( arg) vs foo(arg) -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_method_invocation"
            value="do not insert"
        />

		<!-- 方法调用右括号前是否插入空格。示例：foo(arg ) vs foo(arg) -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_method_invocation"
            value="do not insert"
        />

		<!-- 无参方法调用空括号之间是否插入空格。示例：foo( ) vs foo() -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_between_empty_parens_in_method_invocation"
            value="do not insert"
        />

		<!-- 方法调用参数逗号前是否插入空格。示例：foo(a , b) vs foo(a, b) -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_method_invocation_arguments"
            value="do not insert"
        />

		<!-- 方法调用参数逗号后是否插入空格。示例：foo(a, b) vs foo(a,b) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_method_invocation_arguments" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                     对象创建与构造调用                       ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 对象创建（new）参数逗号前是否插入空格。示例：new Foo(a , b) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_allocation_expression" value="do not insert" />

		<!-- 对象创建（new）参数逗号后是否插入空格。示例：new Foo(a, b) vs new Foo(a,b) -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_allocation_expression" value="insert" />

		<!-- 显式构造函数调用（super/this）参数逗号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_explicitconstructorcall_arguments"
            value="do not insert"
        />

		<!-- 显式构造函数调用（super/this）参数逗号后是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_explicitconstructorcall_arguments"
            value="insert"
        />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                        运算符                                ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- ── 一元 / 后缀 / 前缀运算符 ── -->

		<!-- 后缀运算符（如 i++）前是否插入空格。示例：i ++ vs i++ -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_postfix_operator" value="do not insert" />

		<!-- 后缀运算符（如 i++）后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_postfix_operator" value="do not insert" />

		<!-- 前缀运算符（如 ++i）前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_prefix_operator" value="do not insert" />

		<!-- 前缀运算符（如 ++i）后是否插入空格。示例：++ i vs ++i -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_prefix_operator" value="do not insert" />

		<!-- 一元运算符（如 -x）前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_unary_operator" value="do not insert" />

		<!-- 一元运算符（如 -x）后是否插入空格。示例：- x vs -x -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_unary_operator" value="do not insert" />

		<!-- 逻辑非运算符（!）后是否插入空格。示例：! flag vs !flag -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_not_operator" value="do not insert" />

		<!-- ── 乘法类运算符（* / %） ── -->

		<!-- 乘法类运算符前是否插入空格。示例：a * b vs a*b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_multiplicative_operator" value="insert" />

		<!-- 乘法类运算符后是否插入空格。示例：a * b vs a* b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_multiplicative_operator" value="insert" />

		<!-- ── 加法类运算符（+ -） ── -->

		<!-- 加法类运算符前是否插入空格。示例：a + b vs a+b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_additive_operator" value="insert" />

		<!-- 加法类运算符后是否插入空格。示例：a + b vs a+ b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_additive_operator" value="insert" />

		<!-- ── 字符串拼接运算符（+） ── -->

		<!-- 字符串拼接运算符前是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_string_concatenation" value="insert" />

		<!-- 字符串拼接运算符后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_string_concatenation" value="insert" />

		<!-- ── 移位运算符（<< >> >>>） ── -->

		<!-- 移位运算符前是否插入空格。示例：a << b vs a<<b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_shift_operator" value="insert" />

		<!-- 移位运算符后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_shift_operator" value="insert" />

		<!-- ── 关系运算符（< > <= >= == != instanceof） ── -->

		<!-- 关系运算符前是否插入空格。示例：a < b vs a<b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_relational_operator" value="insert" />

		<!-- 关系运算符后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_relational_operator" value="insert" />

		<!-- ── 位运算符（& | ^） ── -->

		<!-- 位运算符前是否插入空格。示例：a & b vs a&b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_bitwise_operator" value="insert" />

		<!-- 位运算符后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_bitwise_operator" value="insert" />

		<!-- ── 逻辑运算符（&& ||） ── -->

		<!-- 逻辑运算符前是否插入空格。示例：a && b vs a&&b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_logical_operator" value="insert" />

		<!-- 逻辑运算符后是否插入空格 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_logical_operator" value="insert" />

		<!-- ── 三元条件运算符（? :） ── -->

		<!-- 三元条件运算符问号前是否插入空格。示例：a ? b : c vs a? b : c -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_question_in_conditional" value="insert" />

		<!-- 三元条件运算符问号后是否插入空格。示例：a ? b : c vs a?b : c -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_question_in_conditional" value="insert" />

		<!-- 三元条件运算符冒号前是否插入空格。示例：a ? b : c vs a ? b: c -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_colon_in_conditional" value="insert" />

		<!-- 三元条件运算符冒号后是否插入空格。示例：a ? b : c vs a ? b :c -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_colon_in_conditional" value="insert" />

		<!-- ── 赋值运算符（= += -= 等） ── -->

		<!-- 赋值运算符前是否插入空格。示例：a = b vs a=b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_assignment_operator" value="insert" />

		<!-- 赋值运算符后是否插入空格。示例：a = b vs a= b -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_assignment_operator" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                      括号表达式                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 括号表达式左括号前是否插入空格 -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_paren_in_parenthesized_expression"
            value="do not insert"
        />

		<!-- 括号表达式左括号后是否插入空格。示例：( a + b) vs (a + b) -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_parenthesized_expression"
            value="do not insert"
        />

		<!-- 括号表达式右括号前是否插入空格。示例：(a + b ) vs (a + b) -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_parenthesized_expression"
            value="do not insert"
        />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                       类型转换                               ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 强制类型转换左括号后是否插入空格。示例：( int) x vs (int) x -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_opening_paren_in_cast" value="do not insert" />

		<!-- 强制类型转换右括号前是否插入空格。示例：(int ) x vs (int) x -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_closing_paren_in_cast" value="do not insert" />

		<!-- 强制类型转换右括号后是否插入空格。示例：(int) x vs (int)x -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_closing_paren_in_cast" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                         数组                                 ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- ── 数组类型声明 ── -->

		<!-- 数组类型声明左方括号前是否插入空格。示例：String [] vs String[] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_bracket_in_array_type_reference"
            value="do not insert"
        />

		<!-- 数组类型声明空方括号之间是否插入空格。示例：int[ ][] vs int[][] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_between_brackets_in_array_type_reference"
            value="do not insert"
        />

		<!-- ── 数组分配表达式（new 数组） ── -->

		<!-- 数组分配表达式左方括号前是否插入空格。示例：new int [5] vs new int[5] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_bracket_in_array_allocation_expression"
            value="do not insert"
        />

		<!-- 数组分配表达式左方括号后是否插入空格。示例：new int[ 5] vs new int[5] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_bracket_in_array_allocation_expression"
            value="do not insert"
        />

		<!-- 数组分配表达式右方括号前是否插入空格。示例：new int[5 ] vs new int[5] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_bracket_in_array_allocation_expression"
            value="do not insert"
        />

		<!-- 数组分配表达式空方括号之间是否插入空格。示例：new String[ ][ ] vs new String[][] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_between_empty_brackets_in_array_allocation_expression"
            value="do not insert"
        />

		<!-- ── 数组初始化器 ── -->

		<!-- 数组初始化器左花括号前是否插入空格。示例：new int[] {1, 2} vs new int[]{1, 2} -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_brace_in_array_initializer"
            value="do not insert"
        />

		<!-- 数组初始化器左花括号后是否插入空格。示例：{ 1, 2} vs {1, 2} -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_brace_in_array_initializer"
            value="do not insert"
        />

		<!-- 数组初始化器右花括号前是否插入空格。示例：{1, 2 } vs {1, 2} -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_brace_in_array_initializer"
            value="do not insert"
        />

		<!-- 数组初始化器逗号前是否插入空格。示例：{1 , 2} vs {1, 2} -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_array_initializer" value="do not insert" />

		<!-- 数组初始化器逗号后是否插入空格。示例：{1, 2} vs {1,2} -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_array_initializer" value="insert" />

		<!-- 空数组初始化器花括号之间是否插入空格。示例：{ } vs {} -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_between_empty_braces_in_array_initializer"
            value="do not insert"
        />

		<!-- ── 数组元素访问 ── -->

		<!-- 数组元素访问左方括号前是否插入空格。示例：arr [0] vs arr[0] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_bracket_in_array_reference"
            value="do not insert"
        />

		<!-- 数组元素访问左方括号后是否插入空格。示例：arr[ 0] vs arr[0] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_bracket_in_array_reference"
            value="do not insert"
        />

		<!-- 数组元素访问右方括号前是否插入空格。示例：arr[0 ] vs arr[0] -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_bracket_in_array_reference"
            value="do not insert"
        />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                        泛型                                  ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- ── 参数化类型引用（如变量声明中的泛型） ── -->

		<!-- 参数化类型引用左尖括号前是否插入空格。示例：List <String> vs List<String> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_angle_bracket_in_parameterized_type_reference"
            value="do not insert"
        />

		<!-- 参数化类型引用左尖括号后是否插入空格。示例：List< String> vs List<String> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_angle_bracket_in_parameterized_type_reference"
            value="do not insert"
        />

		<!-- 参数化类型引用逗号前是否插入空格。示例：Map<String , Integer> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_parameterized_type_reference"
            value="do not insert"
        />

		<!-- 参数化类型引用逗号后是否插入空格。示例：Map<String, Integer> vs Map<String,Integer> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_parameterized_type_reference" value="insert" />

		<!-- 参数化类型引用右尖括号前是否插入空格。示例：List<String > vs List<String> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_angle_bracket_in_parameterized_type_reference"
            value="do not insert"
        />

		<!-- ── 类型实参（如方法调用中的泛型） ── -->

		<!-- 类型实参左尖括号前是否插入空格。示例：Collections.<String>emptyList() -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_angle_bracket_in_type_arguments"
            value="do not insert"
        />

		<!-- 类型实参左尖括号后是否插入空格。示例：< String> vs <String> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_angle_bracket_in_type_arguments"
            value="do not insert"
        />

		<!-- 类型实参逗号前是否插入空格。示例：<String , Integer> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_type_arguments" value="do not insert" />

		<!-- 类型实参逗号后是否插入空格。示例：<String, Integer> vs <String,Integer> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_type_arguments" value="insert" />

		<!-- 类型实参右尖括号前是否插入空格。示例：<String > vs <String> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_angle_bracket_in_type_arguments"
            value="do not insert"
        />

		<!-- 类型实参右尖括号后是否插入空格。示例：<String> list vs <String>list -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_closing_angle_bracket_in_type_arguments"
            value="do not insert"
        />

		<!-- ── 类型形参（如类/方法定义中的泛型声明） ── -->

		<!-- 类型形参左尖括号前是否插入空格。示例：class Foo <T> vs class Foo<T> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_opening_angle_bracket_in_type_parameters"
            value="do not insert"
        />

		<!-- 类型形参左尖括号后是否插入空格。示例：< T> vs <T> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_opening_angle_bracket_in_type_parameters"
            value="do not insert"
        />

		<!-- 类型形参逗号前是否插入空格。示例：<T , U> vs <T, U> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_comma_in_type_parameters" value="do not insert" />

		<!-- 类型形参逗号后是否插入空格。示例：<T, U> vs <T,U> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_comma_in_type_parameters" value="insert" />

		<!-- 类型形参右尖括号前是否插入空格。示例：<T > vs <T> -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_before_closing_angle_bracket_in_type_parameters"
            value="do not insert"
        />

		<!-- 类型形参右尖括号后是否插入空格。示例：<T> class vs <T>class -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_space_after_closing_angle_bracket_in_type_parameters"
            value="insert"
        />

		<!-- ── 类型参数中的 &（多边界） ── -->

		<!-- 类型参数多边界 & 前是否插入空格。示例：<T extends A & B> vs <T extends A&B> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_and_in_type_parameter" value="insert" />

		<!-- 类型参数多边界 & 后是否插入空格。示例：<T extends A & B> vs <T extends A &B> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_and_in_type_parameter" value="insert" />

		<!-- ── 通配符 ── -->

		<!-- 通配符问号前是否插入空格。示例：List< ? extends Number> vs List<? extends Number> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_before_question_in_wildcard" value="do not insert" />

		<!-- 通配符问号后是否插入空格。示例：<? extends Number> vs <?extends Number> -->
		<setting id="org.eclipse.jdt.core.formatter.insert_space_after_question_in_wildcard" value="do not insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                        空行                                  ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 保留的连续空行数量。当源码中存在连续空行时，最多保留多少行 -->
		<setting id="org.eclipse.jdt.core.formatter.number_of_empty_lines_to_preserve" value="2" />

		<!-- package 声明前的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_before_package" value="0" />

		<!-- package 声明后的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_after_package" value="1" />

		<!-- import 声明前的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_before_imports" value="1" />

		<!-- import 声明后的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_after_imports" value="1" />

		<!-- 顶层类型声明之间的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_between_type_declarations" value="1" />

		<!-- 类体第一个声明前的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_before_first_class_body_declaration" value="0" />

		<!-- 类体最后一个声明后的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_after_last_class_body_declaration" value="0" />

		<!-- 新代码块（字段/方法分组）前的空行数。用于在类体中分隔不同逻辑分组 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_before_new_chunk" value="1" />

		<!-- 成员类型（内部类/接口/枚举）前的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_before_member_type" value="1" />

		<!-- 字段声明前的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_before_field" value="0" />

		<!-- 抽象方法声明前的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_before_abstract_method" value="1" />

		<!-- 方法声明前的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.blank_lines_before_method" value="1" />

		<!-- 方法体开头的空行数 -->
		<setting id="org.eclipse.jdt.core.formatter.number_of_blank_lines_at_beginning_of_method_body" value="0" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║                        其他                                  ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 空语句是否放在新行。示例：if (cond) ; → 当为 true 时分号单独占一行 -->
		<setting id="org.eclipse.jdt.core.formatter.put_empty_statement_on_new_line" value="true" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  数组初始化器换行                                            ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 数组初始化器开括号 { 后是否换行 -->
		<!-- Value: insert | do not insert -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_new_line_after_opening_brace_in_array_initializer"
            value="do not insert"
        />

		<!-- 数组初始化器闭括号 } 前是否换行 -->
		<!-- Value: insert | do not insert -->
		<setting
            id="org.eclipse.jdt.core.formatter.insert_new_line_before_closing_brace_in_array_initializer"
            value="do not insert"
        />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  文件与控制语句换行                                          ║ -->
		<!-- ║  以下 insert_new_line_* 设置的 value 可选：                  ║ -->
		<!-- ║  insert（插入新行）、do not insert（不插入新行）             ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 文件末尾缺少换行符时是否补上 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_at_end_of_file_if_missing" value="do not insert" />

		<!-- if 语句中的 else 前是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_before_else_in_if_statement" value="do not insert" />

		<!-- try 语句中的 catch 前是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_before_catch_in_try_statement" value="do not insert" />

		<!-- try 语句中的 finally 前是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_before_finally_in_try_statement" value="do not insert" />

		<!-- do-while 语句中的 while 前是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_before_while_in_do_statement" value="do not insert" />

		<!-- 标签 (label) 后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_label" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  语句同行与紧凑控制                                          ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 是否将 if 的 then 子句保持在同一行（不换行） -->
		<setting id="org.eclipse.jdt.core.formatter.keep_then_statement_on_same_line" value="true" />

		<!-- 是否将简单的 if-else 保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_imple_if_on_one_line" value="false" />

		<!-- 是否将 else 子句保持在同一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_else_statement_on_same_line" value="true" />

		<!-- 是否将 else if 紧凑排列（不额外缩进） -->
		<setting id="org.eclipse.jdt.core.formatter.compact_else_if" value="true" />

		<!-- 是否将简单 for 循环体保持在同一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_simple_for_body_on_same_line" value="true" />

		<!-- 是否将简单 while 循环体保持在同一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_simple_while_body_on_same_line" value="true" />

		<!-- 是否将简单 do-while 循环体保持在同一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_simple_do_while_body_on_same_line" value="true" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  注解后换行                                                  ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 包声明上的注解后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_annotation_on_package" value="insert" />

		<!-- 类型（类/接口/枚举）声明上的注解后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_annotation_on_type" value="insert" />

		<!-- 枚举常量上的注解后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_annotation_on_enum_constant" value="do not insert" />

		<!-- 方法参数上的注解后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_annotation_on_parameter" value="do not insert" />

		<!-- 方法上的注解后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_annotation_on_method" value="insert" />

		<!-- 局部变量上的注解后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_annotation_on_local_variable" value="do not insert" />

		<!-- 类型注解（TYPE_USE）后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_type_annotation" value="do not insert" />

		<!-- 字段上的注解后是否换行 -->
		<setting id="org.eclipse.jdt.core.formatter.insert_new_line_after_annotation_on_field" value="insert" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  代码块/声明单行控制                                         ║ -->
		<!-- ║  Value: one_line_never | one_line_if_empty                   ║ -->
		<!-- ║        | one_line_if_single_item | one_line_always           ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 循环体块是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_loop_body_block_on_one_line" value="one_line_never" />

		<!-- if-then 体块是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_if_then_body_block_on_one_line" value="one_line_never" />

		<!-- Lambda 体块是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_lambda_body_block_on_one_line" value="one_line_never" />

		<!-- 普通代码块是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_code_block_on_one_line" value="one_line_never" />

		<!-- 方法体是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_method_body_on_one_line" value="one_line_never" />

		<!-- 是否将简单的 getter/setter 方法保持在一行 -->
		<!-- Value: true | false -->
		<setting id="org.eclipse.jdt.core.formatter.keep_simple_getter_setter_on_one_line" value="false" />

		<!-- 类型声明是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_type_declaration_on_one_line" value="one_line_never" />

		<!-- 匿名类型声明是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_anonymous_type_declaration_on_one_line" value="one_line_never" />

		<!-- 枚举声明是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_enum_declaration_on_one_line" value="one_line_never" />

		<!-- 枚举常量声明是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_enum_constant_declaration_on_one_line" value="one_line_never" />

		<!-- Record 声明是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_record_declaration_on_one_line" value="one_line_never" />

		<!-- Record 构造器是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_record_constructor_on_one_line" value="one_line_never" />

		<!-- 注解声明是否保持在一行 -->
		<setting id="org.eclipse.jdt.core.formatter.keep_annotation_declaration_on_one_line" value="one_line_never" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  行宽与续行缩进                                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 每行最大字符数（行宽限制） -->
		<setting id="org.eclipse.jdt.core.formatter.lineSplit" value="120" />

		<!-- 续行缩进级别（相对基础缩进的额外缩进数） -->
		<setting id="org.eclipse.jdt.core.formatter.continuation_indentation" value="2" />

		<!-- 数组初始化器的续行缩进级别 -->
		<setting id="org.eclipse.jdt.core.formatter.continuation_indentation_for_array_initializer" value="2" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  换行策略                                                    ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 是否合并已被换行的行（当行宽足够时是否将换行恢复为一行） -->
		<setting id="org.eclipse.jdt.core.formatter.join_wrapped_lines" value="false" />

		<!-- 嵌套表达式换行时是否优先包裹外层表达式 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_outer_expressions_when_nested" value="false" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  对齐与换行 (Alignment & Wrapping)                           ║ -->
		<!-- ║  Alignment 值为位掩码：0=不换行, 1=缩进一级, 2=按列对齐      ║ -->
		<!-- ║  +16=强制换行, +32=首元素独占一行                            ║ -->
		<!-- ║  常见组合：0=不换行, 2=按需换行按列对齐, 16=强制换行缩进一级 ║ -->
		<!-- ║  18=强制换行按列对齐(16+2), 49=强制换行首元素独占一行缩进一级║ -->
		<!-- ║  (32+16+1)                                                   ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 类型声明中 extends/superclass 的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_superclass_in_type_declaration" value="2" />

		<!-- 类型声明中 implements 列表的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_superinterfaces_in_type_declaration" value="2" />

		<!-- 多字段声明（同一行声明的多个字段）的对齐方式。16=强制换行缩进一级 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_multiple_fields" value="16" />

		<!-- 构造器声明中参数的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_parameters_in_constructor_declaration" value="2" />

		<!-- 构造器声明中 throws 子句的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_throws_clause_in_constructor_declaration" value="2" />

		<!-- 方法声明整体的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_method_declaration" value="0" />

		<!-- 方法声明中 throws 子句的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_throws_clause_in_method_declaration" value="2" />

		<!-- 方法声明中参数的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_parameters_in_method_declaration" value="0" />

		<!-- 方法调用中参数的对齐方式（限定调用链）。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_parameters_in_method_invocation" value="0" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  枚举对齐                                                    ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 枚举常量的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_enum_constants" value="0" />

		<!-- 枚举常量构造调用的参数对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_arguments_in_enum_constant" value="2" />

		<!-- 枚举声明中 implements 列表的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_superinterfaces_in_enum_declaration" value="2" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  Record 对齐                                                 ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- Record 组件的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_record_components" value="2" />

		<!-- Record 声明中 implements 列表的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_superinterfaces_in_record_declaration" value="2" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  方法调用/对象创建参数对齐                                   ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 方法调用中参数的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_arguments_in_method_invocation" value="2" />

		<!-- 方法调用链中选择器（.方法名）的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_selector_in_method_invocation" value="2" />

		<!-- 显式构造调用（super()/this()）的参数对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_arguments_in_explicit_constructor_call" value="2" />

		<!-- 对象创建表达式（new Type(...)）的参数对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_arguments_in_allocation_expression" value="2" />

		<!-- 限定对象创建表达式（outer.new Inner(...)）的参数对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_arguments_in_qualified_allocation_expression" value="2" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  运算符对齐与换行                                            ║ -->
		<!-- ║  每个「对齐」setting 控制换行时运算符的对齐方式；            ║ -->
		<!-- ║  对应的「wrap_before」setting 控制是否在运算符前换行。       ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 乘法运算符（* / %）的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_multiplicative_operator" value="2" />
		<!-- 是否在乘法运算符前换行（而非运算符后） -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_multiplicative_operator" value="true" />

		<!-- 加法运算符（+ -）的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_additive_operator" value="2" />
		<!-- 是否在加法运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_additive_operator" value="true" />

		<!-- 字符串拼接（+）的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_string_concatenation" value="2" />
		<!-- 是否在字符串拼接运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_string_concatenation" value="true" />

		<!-- 移位运算符（<< >> >>>）的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_shift_operator" value="2" />
		<!-- 是否在移位运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_shift_operator" value="true" />

		<!-- 关系运算符（< > <= >= instanceof）的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_relational_operator" value="2" />
		<!-- 是否在关系运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_relational_operator" value="true" />

		<!-- 位运算符（& | ^）的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_bitwise_operator" value="2" />
		<!-- 是否在位运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_bitwise_operator" value="true" />

		<!-- 逻辑运算符（&& ||）的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_logical_operator" value="2" />
		<!-- 是否在逻辑运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_logical_operator" value="true" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  条件表达式对齐                                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 三元条件表达式（? :）的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_conditional_expression" value="0" />

		<!-- 链式三元条件表达式的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_conditional_expression_chain" value="0" />

		<!-- 是否在条件运算符（? :）前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_conditional_operator" value="true" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  赋值对齐                                                    ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 赋值表达式的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_assignment" value="0" />

		<!-- 是否在赋值运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_assignment_operator" value="true" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  数组初始化器与 for 循环对齐                                 ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 数组初始化器中表达式的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_expressions_in_array_initializer" value="2" />

		<!-- for 循环头部表达式的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_expressions_in_for_loop_header" value="2" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  紧凑控制结构对齐                                            ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 紧凑 if（单行 if-else）的对齐方式。16=强制换行缩进一级 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_compact_if" value="16" />

		<!-- 紧凑循环（单行循环体）的对齐方式。16=强制换行缩进一级 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_compact_loops" value="16" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  try-with-resources / 多重捕获对齐                           ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- try-with-resources 中资源的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_resources_in_try" value="2" />

		<!-- 多重捕获（multi-catch）中联合类型的对齐方式。18=强制换行按列对齐(16+2) -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_union_type_in_multicatch" value="18" />

		<!-- 多重捕获中是否在 | 运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_or_operator_multicatch" value="false" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  断言消息对齐                                                ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- assert 语句中消息表达式的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_assertion_message" value="0" />

		<!-- 是否在 assert 消息的冒号运算符前换行 -->
		<setting id="org.eclipse.jdt.core.formatter.wrap_before_assertion_message_operator" value="false" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  泛型/类型参数对齐                                           ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 参数化类型引用（如 List<String>）的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_parameterized_type_references" value="0" />

		<!-- 类型实参（如 <String>）的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_type_arguments" value="0" />

		<!-- 类型形参（如 <T extends Foo>）的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_type_parameters" value="0" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  注解对齐                                                    ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 包声明上注解的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_annotations_on_package" value="0" />

		<!-- 类型声明上注解的对齐方式。49=强制换行首元素独占一行缩进一级(32+16+1) -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_annotations_on_type" value="49" />

		<!-- 枚举常量上注解的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_annotations_on_enum_constant" value="0" />

		<!-- 字段上注解的对齐方式。49=强制换行首元素独占一行缩进一级(32+16+1) -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_annotations_on_field" value="49" />

		<!-- 方法上注解的对齐方式。49=强制换行首元素独占一行缩进一级(32+16+1) -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_annotations_on_method" value="49" />

		<!-- 局部变量上注解的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_annotations_on_local_variable" value="0" />

		<!-- 方法参数上注解的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_annotations_on_parameter" value="0" />

		<!-- 类型注解（TYPE_USE）的对齐方式。49=强制换行首元素独占一行缩进一级(32+16+1) -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_type_annotations" value="49" />

		<!-- 注解中参数的对齐方式。2=按需换行按列对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_arguments_in_annotation" value="2" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  模块语句对齐                                                ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 模块语句的对齐方式。0=不换行 -->
		<setting id="org.eclipse.jdt.core.formatter.alignment_for_module_statements" value="0" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  注释格式化                                                  ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 行注释的最大行宽 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.line_length" value="120" />

		<!-- 行宽是否从行首位置开始计算（false=从注释起始位置计算） -->
		<setting id="org.eclipse.jdt.core.formatter.comment.count_line_length_from_starting_position" value="false" />

		<!-- 是否格式化 Javadoc 注释 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.format_javadoc_comments" value="true" />

		<!-- 是否格式化块注释 (/* ... */) -->
		<setting id="org.eclipse.jdt.core.formatter.comment.format_block_comments" value="false" />

		<!-- 是否格式化行注释 (// ...) -->
		<setting id="org.eclipse.jdt.core.formatter.comment.format_line_comments" value="false" />

		<!-- 是否格式化从第一列开始的行注释 -->
		<setting id="org.eclipse.jdt.core.formatter.format_line_comment_starting_on_first_column" value="false" />

		<!-- 是否格式化文件头注释（文件首个块/Javadoc 注释） -->
		<setting id="org.eclipse.jdt.core.formatter.comment.format_header" value="true" />

		<!-- 是否保留代码与行注释之间的空白 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.preserve_white_space_between_code_and_line_comments" value="true" />

		<!-- 是否从不缩进第一列的行注释 -->
		<setting id="org.eclipse.jdt.core.formatter.never_indent_line_comments_on_first_column" value="true" />

		<!-- 是否从不缩进第一列的块注释 -->
		<setting id="org.eclipse.jdt.core.formatter.never_indent_block_comments_on_first_column" value="false" />

		<!-- 是否合并注释中的换行（当行宽足够时） -->
		<setting id="org.eclipse.jdt.core.formatter.join_lines_in_comments" value="false" />

		<!-- 是否格式化注释中的 HTML 标签 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.format_html" value="false" />

		<!-- 是否格式化注释中的源代码（<code> 标签内容） -->
		<setting id="org.eclipse.jdt.core.formatter.comment.format_source_code" value="false" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  Javadoc 格式化                                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- Javadoc 根标签（@param, @return 等）前是否插入新行 -->
		<!-- Value: insert | do not insert -->
		<setting id="org.eclipse.jdt.core.formatter.comment.insert_new_line_before_root_tags" value="insert" />

		<!-- 不同 Javadoc 标签之间是否插入新行 -->
		<!-- Value: insert | do not insert -->
		<setting id="org.eclipse.jdt.core.formatter.comment.insert_new_line_between_different_tags" value="do not insert" />

		<!-- 是否对齐 Javadoc 标签名与描述 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.align_tags_names_descriptions" value="false" />

		<!-- 是否将同组 Javadoc 标签的描述对齐 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.align_tags_descriptions_grouped" value="true" />

		<!-- @param 标签的参数描述前是否插入新行 -->
		<!-- Value: insert | do not insert -->
		<setting id="org.eclipse.jdt.core.formatter.comment.insert_new_line_for_parameter" value="do not insert" />

		<!-- 是否缩进 Javadoc 中参数描述部分 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.indent_parameter_description" value="false" />

		<!-- 是否缩进 Javadoc 标签描述部分 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.indent_tag_description" value="false" />

		<!-- 是否缩进 Javadoc 根标签 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.indent_root_tags" value="false" />

		<!-- 是否在 Javadoc 边界处插入空行 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.new_lines_at_javadoc_boundaries" value="true" />

		<!-- 是否清除 Javadoc 中的空行 -->
		<setting id="org.eclipse.jdt.core.formatter.comment.clear_blank_lines_in_javadoc_comment" value="false" />

		<!-- ╔══════════════════════════════════════════════════════════════╗ -->
		<!-- ║  格式化开关标签                                              ║ -->
		<!-- ╚══════════════════════════════════════════════════════════════╝ -->

		<!-- 是否启用格式化开/关标签 -->
		<setting id="org.eclipse.jdt.core.formatter.use_on_off_tags" value="true" />

		<!-- 启用格式化的标签 -->
		<setting id="org.eclipse.jdt.core.formatter.enabling_tag" value="@formatter:on" />

		<!-- 禁用格式化的标签 -->
		<setting id="org.eclipse.jdt.core.formatter.disabling_tag" value="@formatter:off" />

	</profile>
</profiles>
```

# 完整配置：settings.json

下面是完整的 settings.json，Zed 版本 1.17.2，所有注释保留，直接替换你的 settings.json 即可使用。原始配置里的 macOS 路径已替换成 Linux 路径：JDK 指向 `~/.jdks/` 软链，格式化器在 `~/.config/formatter/`（路径里的 /home/zhangtianci 换成你自己的用户名）。

每一项改动的对照在「修改说明」，分组解读与环境搭建在「逐节拆解」。

```jsonc
// Zed settings
//
// For information on how to configure Zed, see the Zed
// documentation: https://zed.dev/docs/configuring-zed
//
// To see all of Zed's default settings without changing your
// custom settings, run `zed: open default settings` from the
// command palette (cmd-shift-p / ctrl-shift-p)
{
  // 用于 UI 的 Zed 主题名称。
  //
  // `mode` 可为以下值之一：
  // - "system"：使用与系统外观对应的主题
  // - "light"：使用 "light" 字段指定的主题
  // - "dark"：使用 "dark" 字段指定的主题
  "theme": {
    "mode": "system",
    "light": "Catppuccin Latte (Blur)",
    "dark": "Catppuccin Espresso (Blur)",
  },
  "icon_theme": "Catppuccin Latte",
  // 要使用的基础键位绑定集的名称。
  // 此设置可接受以下值：
  //
  // 1. "Zed"
  // 2. "VSCode"
  // 3. "Atom"
  // 4. "JetBrains"
  // 5. "SublimeText"
  // 6. "TextMate"
  // 7. "Emacs"
  // 8. "Cursor"
  // 9. "None"
  "base_keymap": "Zed",
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
  // 如何显示补全菜单中每个条目的 LSP 项类型（函数、方法、变量等）。
  //
  // 1. 不显示项类型：
  //   "off"（默认值）
  // 2. 显示单字母标记，并根据当前语法主题着色：
  //   "symbol"
  "completion_menu_item_kind": "symbol",
  // 编辑器中显示差异的方式。
  //
  // 默认：split
  "diff_view_style": "unified",
  // 当光标在括号内时，是否在编辑器中显示方法签名。
  "auto_signature_help": true,
  // 是否在补全或插入括号对后显示签名帮助。
  // 如果启用了 `auto_signature_help`，此设置也将被视为启用。
  "show_signature_help_after_edits": true,
  // 是否以及如何显示来自语言服务器的代码镜头（Code Lens）。
  //
  // 可选值：
  //
  // 1. 不显示代码镜头。
  //      "code_lens": "off",
  // 2. 在代码元素上方显示语言服务器提供的代码镜头。
  //      "code_lens": "on",
  // 3. 在代码操作菜单中显示代码镜头。
  //      "code_lens": "menu",
  "code_lens": "on",
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
    "group_by": "none",
    "file_icons": true,
    /// 在面板中是以树状视图还是平铺视图显示条目。
    ///
    /// 默认值：false
    "tree_view": true,
  },
  "agent": {
    "sandbox_permissions": {
      "allow_unsandboxed": false,
      "write_paths": ["~/.local/state/fnm_multishells", "~/.m2/repository"],
    },
    "tool_permissions": {
      "tools": {
        "terminal": {
          "always_allow": [
            {
              "pattern": "^rg\\b",
            },
            {
              "pattern": "^echo\\b",
            },
            {
              "pattern": "^head_lines\\s+10(\\s|$)",
            },
            {
              "pattern": "^tail_lines\\s+8(\\s|$)",
            },
            {
              "pattern": "^fd\\b",
            },
            {
              "pattern": "^rg\\s+FusionBankPointRangeDTO(\\s|$)",
            },
            {
              "pattern": "^head_lines\\s+120(\\s|$)",
            },
            {
              "pattern": "^git\\b",
            },
            {
              "pattern": "^tail\\b",
            },
            {
              "pattern": "^head\\b",
            },
            {
              "pattern": "^ls\\b",
            },
            {
              "pattern": "^find\\s+\\.\\.(\\s|$)",
            },
            {
              "pattern": "^sed\\b",
            },
            {
              "pattern": "^ls\\s+~/.jdks/(\\s|$)",
            },
            {
              "pattern": "^mvn\\b",
            },
            {
              "pattern": "^env\\b",
            },
            {
              "pattern": "^ls\\s+target/surefire-reports/(\\s|$)",
            },
            {
              "pattern": "^head_lines\\s+5(\\s|$)",
            },
            {
              "pattern": "^head_lines\\s+3(\\s|$)",
            },
            {
              "pattern": "^head_lines\\s+20(\\s|$)",
            },
            {
              "pattern": "^JAVA_HOME=/home/zhangtianci/.jdks/21\\s+PATH=/home/zhangtianci/.jdks/21/bin:/usr/bin:/bin\\s+mvn\\s+test(\\s|$)",
            },
            {
              "pattern": "^grep\\b",
            },
            {
              "pattern": "^unzip\\b",
            },
            {
              "pattern": "^rg\\s+FusionModeValueEnum(\\s|$)",
            },
            {
              "pattern": "^which\\s+mvn(\\s|$)",
            },
            {
              "pattern": "^java\\b",
            },
          ],
        },
      },
    },
    // 是否在生成 git 提交消息时将项目规则文件
    // （AGENTS.md、CLAUDE.md、.rules 等）包含在提示中。
    "commit_message_include_project_rules": false,
    "default_profile": "write",
    // 创建新线程时使用的默认模型。
    "default_model": {
      "effort": "max",
      // 要使用的提供商。
      "provider": "deepseek",
      // 要使用的模型。
      "model": "deepseek-v4-pro",
      // 是否启用思考。
      "enable_thinking": true,
    },
    "favorite_models": [],
    "model_parameters": [],
    "commit_message_model": {
      // 要使用的提供商。
      "provider": "deepseek",
      // 要使用的模型。
      "model": "deepseek-v4-flash",
    },
    "commit_message_instructions": "当用户要求生成提交信息时，写出简洁清晰的提交信息，准确概括变更内容。\n\n## 基本原则\n\n- 只返回提交信息本身，不附带说明、评论、原始 diff 或 Markdown 代码块。\n- 使用简体中文编写提交信息。\n- 仅用标题就能清晰表达变更内容时，无需正文。\n- 正文每行不超过 72 个字符。\n- 标题尽量控制在 50 个字符以内，末尾不加标点。\n- 标题使用祈使语气，直接描述变更行为，不重复 `<类型>` 已传达的信息。\n- 不在正文中重复标题已表达的内容。\n- 标题与正文之间必须空一行。\n\n## 约定式提交格式\n\n<类型>[可选的作用域]: <简短描述>\n\n[可选的正文]\n\n[可选的脚注]\n\n## 类型\n\n| 类型 | 说明 |\n| --- | --- |\n| feat | 新增功能，对应语义化版本的 MINOR |\n| fix | 修复 Bug，对应语义化版本的 PATCH |\n| docs | 仅文档变更，例如 README、注释等 |\n| style | 代码风格调整，不影响逻辑 |\n| refactor | 代码重构，既非新增功能，也非修复 Bug |\n| perf | 性能优化 |\n| test | 测试相关，例如新增或修改测试用例 |\n| build | 构建系统或外部依赖变更 |\n| ci | CI/CD 配置变更 |\n| chore | 其他不影响源码或测试的日常任务 |\n| revert | 回滚某次提交 |\n\n## 作用域\n\n作用域可选，紧跟在类型之后并用括号包裹，用于指明影响范围。\n\n示例：\nfeat(parser): 新增数组解析能力\nfix(auth): 修复登录超时未正确处理的问题\n\n## 破坏性变更\n\n存在破坏性变更时，在脚注中添加 `BREAKING CHANGE:`。\n\n示例：\nfeat: 允许外部配置文件覆盖默认值\n\nBREAKING CHANGE: 配置文件中的 `extends` 字段现用于扩展其他配置，行为与旧版不兼容。\n\n## 脚注格式\n\n- 脚注位于正文之后，与正文间隔一个空行。\n- 每行脚注格式为 `<令牌>: <值>`。\n- 令牌使用 `-` 作为连字符（如 `Reviewed-by`、`Refs`），以区别于多行正文。\n- `BREAKING CHANGE` 作为令牌时必须大写，`BREAKING-CHANGE` 为其同义词。\n\n示例：\nfix: 防止请求竞态\n\n引入请求 ID 并引用最新请求，丢弃非最新请求的响应。\n\nReviewed-by: Z\nRefs: #123",
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
  // 是否在保存前执行缓冲区格式化：
  //   "on" — 格式化整个缓冲区
  //   "off" — 不格式化
  //   "modifications" — 仅格式化有未暂存更改的行；当没有 Git 差异或语言服务器不支持范围格式化时跳过格式化
  //   "modifications_if_available" — 同上，但在无法使用范围格式化时回退为格式化整个缓冲区
  // 请注意，如果启用了带延迟的自动保存，则 format_on_save 将被忽略
  "format_on_save": "modifications",
  // 如何软换行长文本。
  // 可能的值：
  //
  // 1. 通常首选单行，除非遇到非常长的行。
  //      "soft_wrap": "none",
  //      "soft_wrap": "prefer_line", // （已弃用，与 "none" 相同）
  // 2. 软换行超出编辑器宽度的行。
  //      "soft_wrap": "editor_width",
  // 3. 在首选行长度处软换行。
  //      "soft_wrap": "preferred_line_length",
  // 4. 在首选行长度或编辑器宽度中较小者处软换行。
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
  "edit_predictions": {
    // 控制 Zed 在使用编辑预测（Edit Predictions）功能时是否可能收集训练数据。
    // 仅当项目被检测为开源项目时才会捕获数据。
    // 可选值：
    //   - "default": 使用之前通过状态栏开关设置的首选项，
    //     若未存储过首选项则为 false（不允许）。
    //   - "yes": 允许收集开源项目中的文件数据。
    //   - "no": 永远不允许收集数据。
    "allow_data_collection": "no",
  },
  // 终端特定设置。
  "terminal": {
    // 打开终端时使用什么 shell。可取 3 个值：
    // 1. 使用 /etc/passwd 中系统的默认终端配置
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
    "shell": {
      "program": "fish",
    },
    // 在终端的工作目录中（由 working_directory 设置决定），
    // 如果找到 Python 虚拟环境，则激活它。
    // 将其设置为 "off" 以禁用此行为。
    "detect_venv": {
      "on": {
        // 搜索虚拟环境时要使用的默认目录，相对于当前工作目录。
        // 我们建议在项目的设置中覆盖此项，而非全局设置。
        "directories": [".env", "env", ".venv", "venv"],
        // 也可以是 `csh`、`fish`、`nushell` 和 `power_shell`
        "activate_script": "fish",
      },
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
  // LSP 特定设置。
  "lsp": {
    // 在此处以 LSP 名称作为键指定设置。
    "jdtls": {
      "settings": {
        "java_home": "/home/zhangtianci/.jdks/25",
        "lombok_support": true,
        "jdk_auto_download": false,

        // 控制何时检查 JDTLS、Lombok 和调试器的更新
        // - "always"（默认）：始终检查并下载最新版本
        // - "once"：仅当本地没有安装时才检查更新
        // - "never"：从不检查更新，仅使用已有的本地安装（若缺失则报错）
        //
        // 注意：无效值将默认使用 "always"
        // 如果提供了自定义路径（见下文），则对该组件忽略 check_updates 设置
        "check_updates": "always",
      },
      "initialization_options": {
        "settings": {
          "java": {
            /// JDK 安装根目录的绝对路径。
            /// 类型: string | null
            /// 默认值: null
            "home": "/home/zhangtianci/.jdks/25",

            "edit": {
              /// 智能分号检测。
              /// 启用后，输入分号时编辑器会智能地将其置于合适的位置（例如
              /// 自动跳过已存在的分号），而不是简单地追加。
              /// 类型: boolean
              /// 默认值: false
              "smartSemicolonDetection": {
                "enabled": true,
              },
            },

            "format": {
              /// 是否启用 JDTLS 内置格式化器。
              /// 类型: boolean
              /// 默认值: true
              "enabled": true,

              /// 键入时自动格式化（例如输入分号或闭合大括号时触发）。
              "onType": {
                /// 类型: boolean
                /// 默认值: false
                "enabled": true,
              },

              /// 按 Tab 键时插入空格而非制表符（\t）。
              /// 类型: boolean
              /// 默认值: true
              "insertSpaces": true,

              /// 一个制表符等价于多少个空格（即缩进宽度）。
              /// 类型: integer
              /// 默认值: 4
              "tabSize": 4,

              /// 格式化时是否包含注释区域的格式化。
              "comments": {
                /// 类型: boolean
                /// 默认值: true
                "enabled": true,
              },

              /// 外部格式化器配置文件（Eclipse Formatter XML 格式）。
              /// 该文件需由 Eclipse IDE 导出（Window → Preferences → Java → Code Style → Formatter）。
              "settings": {
                /// URL 或本地文件路径。
                /// 类型: string | null
                /// 默认值: null
                "url": "/home/zhangtianci/.config/formatter/eclipse-java-formatter.xml",
              },
            },

            "configuration": {
              /// 当 `pom.xml` 或 `build.gradle` 等构建配置文件发生变化时，
              /// 如何处理项目自动更新。
              /// 可选值:
              ///   - "disabled"    : 禁用自动更新，需手动触发（通过命令或按钮）
              ///   - "interactive" : 检测到变化时弹出提示，由用户决定是否更新
              ///   - "automatic"   : 自动静默更新
              /// 类型: string
              /// 默认值: "interactive"
              "updateBuildConfiguration": "automatic",

              /// Java 执行环境（JDK 运行时）列表。
              /// 用于多版本项目或不同模块指定不同 JDK。
              /// 类型: array of object
              /// 默认值: []
              "runtimes": [
                {
                  /// 执行环境名称，如 "JavaSE-11" / "JavaSE-17"
                  "name": "JavaSE-25",
                  /// JDK 安装路径
                  "path": "/home/zhangtianci/.jdks/25",
                  /// 是否设为默认 JDK（可选）
                  "default": true,
                  /// 源代码附件的路径（可选）
                  // "sources": "/path/to/src.zip",
                  /// Javadoc 附件的路径（可选）
                  // "javadoc": "https://docs.oracle.com/en/java/javase/11/docs/api",
                },
                {
                  "name": "JavaSE-21",
                  "path": "/home/zhangtianci/.jdks/21",
                },
                {
                  "name": "JavaSE-17",
                  "path": "/home/zhangtianci/.jdks/17",
                },
                {
                  "name": "JavaSE-1.8",
                  "path": "/home/zhangtianci/.jdks/8",
                },
              ],
            },

            /// Maven 依赖管理相关配置（独立于项目导入）
            "maven": {
              /// 是否自动下载 Maven 依赖的源代码（sources.jar）。
              /// 类型: boolean
              /// 默认值: false
              "downloadSources": true,

              /// 是否每次同步时强制更新 SNAPSHOT 和 RELEASE 版本依赖。
              /// 类型: boolean
              /// 默认值: false
              "updateSnapshots": true,
            },

            /// 接口 / 抽象类方法的实现显示策略。
            /// 可选值:
            ///   - "none"       : 不显示
            ///   - "all"        : 显示该接口/抽象方法的所有实现
            ///   - "overridden" : 仅显示被重写（有具体实现）的方法
            /// 类型: string
            /// 默认值: "none"
            "implementationCodeLens": "all",

            "saveActions": {
              /// 保存文件时是否自动组织（排序并移除未使用的）导入语句。
              /// 类型: boolean
              /// 默认值: false
              "organizeImports": true,
            },

            /// 方法签名帮助（在输入方法参数时显示参数列表与文档）。
            "signatureHelp": {
              /// 类型: boolean
              /// 默认值: false
              "enabled": true,

              /// 是否在签名帮助中显示方法的 Javadoc 概要描述。
              "description": {
                /// 类型: boolean
                /// 默认值: false
                "enabled": true,
              },
            },

            /// 最大并发构建任务数。
            /// 类型: integer (>= 1)
            /// 默认值: 1（由 Eclipse 平台决定）
            "maxConcurrentBuilds": 5,

            "project": {
              /// 当项目没有显式指定编码时，如何处理编码问题。
              /// 可选值:
              ///   - "ignore"  : 忽略，使用系统默认编码
              ///   - "replace" : 强制使用工作区编码覆盖
              ///   - "prompt"  : 弹窗提示用户选择编码
              /// 类型: string
              /// 默认值: "ignore"
              "encoding": "ignore",
            },

            "completion": {
              /// 是否启用补全项的延迟解析（Lazy Resolve）。
              /// 启用后补全列表会更快返回，但单项的详细信息（如 Javadoc）
              /// 需要被选中时才加载，可显著提升弱网或低性能环境下的体验。
              "lazyResolveTextEdit": {
                /// 类型: boolean
                /// 默认值: false
                "enabled": true,
              },

              /// 从补全列表中选择方法后，如何处理方法参数。
              /// 可选值:
              ///   - "insertParameterNames"           : 插入参数名作为占位符
              ///   - "insertBestGuessedArguments"     : 插入 JDTLS 最佳猜测的实际参数
              ///   - "insertBestGuessedArgumentsWithType": 插入带类型的最佳猜测参数
              ///   - true  (已弃用) : 等同于 "insertBestGuessedArguments"
              ///   - false (已弃用) : 等同于 "insertParameterNames"
              /// 类型: boolean | string
              /// 默认值: "insertParameterNames"
              "guessMethodArguments": "insertBestGuessedArguments",

              /// 收藏的静态成员 —— 这些成员在补全列表中会总是优先显示。
              /// 格式: `全限定类名.*` 或 `全限定类名.方法名`
              /// 类型: array of string
              /// 默认值:
              ///   ["org.junit.Assert.*", "org.junit.Assume.*",
              ///    "org.junit.jupiter.api.Assertions.*", "...Assumptions.*",
              ///    "...DynamicContainer.*", "...DynamicTest.*"]
              "favoriteStaticMembers": [
                "org.junit.Assert.*",
                "org.junit.Assume.*",
                "org.junit.jupiter.api.Assertions.*",
                "org.junit.jupiter.api.Assumptions.*",
                "org.junit.jupiter.api.DynamicContainer.*",
                "org.junit.jupiter.api.DynamicTest.*",
                "java.util.Objects.*",
                "java.util.Collections.*",
              ],

              /// 组织导入时各包（package）的排序顺序。
              /// 排在越前的包越先出现在 import 区块中。
              /// 类型: array of string
              /// 默认值: ["java", "javax", "org", "com"]
              "importOrder": ["java", "javax", "org", "com", "io"],

              /// 补全列表中要过滤（隐藏）的类型列表。
              /// 支持通配符 `*`，如 `com.sun.*`。
              /// 类型: array of string
              /// 默认值:
              ///   ["com.sun.*", "io.micrometer.shaded.*", "java.awt.*",
              ///    "jdk.*", "org.graalvm.*", "sun.*"]
              "filteredTypes": ["com.sun.*", "io.micrometer.shaded.*", "java.awt.*", "jdk.*", "org.graalvm.*", "sun.*"],

              /// 是否启用链式补全。
              /// 启用后，在输入 `a.b().` 时会自动探测 `.b()` 返回类型的可用方法
              /// 并纳入补全列表，无需手动逐级展开。
              "chain": {
                /// 类型: boolean
                /// 默认值: false
                "enabled": true,
              },
            },

            "sources": {
              "organizeImports": {
                /// 当来自同一包的普通导入数量超过此阈值时，压缩为星号导入
                /// （如 `import java.util.*`）。
                /// 类型: integer
                /// 默认值: 99
                "starThreshold": 50,

                /// 当来自同一包的静态导入数量超过此阈值时，压缩为静态星号导入
                /// （如 `import static org.mockito.Mockito.*`）。
                /// 类型: integer
                /// 默认值: 99
                "staticStarThreshold": 10,
              },
            },

            "codeGeneration": {
              /// hashCode / equals 方法生成策略
              "hashCodeEquals": {
                /// 是否使用 Java 7 引入的 `Objects.hash(...)` 和 `Objects.equals(...)`。
                /// 关掉则生成传统的自实现 hash 算法。
                /// 类型: boolean
                /// 默认值: false
                "useJava7Objects": true,

                /// 在 `equals` 方法中是否使用 `instanceof` 来比较类型
                /// （而非 `getClass()`）。
                /// 使用 `instanceof` 允许子类相等性判断，更符合 Liskov 替换原则。
                /// 类型: boolean
                /// 默认值: false
                "useInstanceof": true,
              },

              /// 生成方法体时是否为 `if` / `for` 等控制流语句使用块（`{ }`）。
              /// 类型: boolean
              /// 默认值: false
              "useBlocks": true,

              /// 是否在生成的方法上生成 Javadoc 注释。
              /// 类型: boolean
              /// 默认值: false
              "generateComments": true,

              /// 生成代码的插入位置。
              /// 可选值:
              ///   - "lastMember"   : 类/接口的最后一个成员之后
              ///   - "afterCursor"  : 当前光标位置之后
              ///   - "beforeCursor" : 当前光标位置之前
              /// 类型: string | null
              /// 默认值: null（由 Eclipse 内部策略决定）
              "insertionLocation": "lastMember",

              /// 是否为代码生成所创建的新字段添加 `final` 修饰符。
              /// 可选值:
              ///   - "none"    : 不添加 final
              ///   - "all"     : 所有新字段都声明为 final
              ///   - "private" : 仅 private 字段声明为 final
              ///   - "package" : package 可见性及更窄可见性的字段声明为 final
              /// 类型: string
              /// 默认值: "none"
              "addFinalForNewDeclaration": "all",

              /// toString 方法生成配置
              "toString": {
                /// 生成的 toString 使用的模板。
                /// 可选值:
                ///   - "STRING_BUILDER"            : 使用 StringBuilder 拼接
                ///   - "STRING_BUILDER_CHAINED"    : 使用链式 StringBuilder
                ///   - "STRING_FORMAT"             : 使用 String.format()
                ///   - "TO_STRING_BUILDER"         : Apache Commons Lang3 ToStringBuilder
                ///   - "SPRING_TO_STRING_CREATOR"  : Spring ToStringCreator
                ///   - "GUAVA_TO_STRING_HELPER"    : Guava MoreObjects.toStringHelper
                ///   - "OBJECTS_TO_STRING"         : Java 7 Objects.toString()
                /// 类型: string | null
                /// 默认值: null
                "template": "STRING_BUILDER_CHAINED",

                /// 代码风格（决定生成逻辑的实现方式）。
                /// 可选值:
                ///   - "Eclipse" : Eclipse 内置风格（StringBuilder）
                ///   - "Apache"  : Apache Commons Lang
                ///   - "Guava"   : Google Guava
                ///   - "Spring"  : Spring Framework
                ///   - "Custom"  : 使用自定义模板（需同时指定 template）
                /// 类型: string | null
                /// 默认值: null
                "codeStyle": "Eclipse",

                /// toString 输出中是否跳过值为 null 的字段。
                /// 类型: boolean
                /// 默认值: false
                "skipNullValues": true,

                /// 是否列出数组 / 集合的内容（而非仅显示其引用地址）。
                /// 类型: boolean
                /// 默认值: true
                "listArrayContents": true,

                /// toString 输出中数组 / 集合的最大元素数量。
                /// 0 表示无限制。
                /// 类型: integer
                /// 默认值: 0
                "limitElements": 10,
              },
            },

            /// 是否允许 JDTLS 收集匿名遥测数据。
            /// 类型: boolean
            /// 默认值: false
            "telemetry": {
              "enabled": false,
            },

            "inlayHints": {
              /// 参数名称内嵌提示。
              "parameterNames": {
                /// 显示模式:
                ///   - "none"     : 不显示
                ///   - "literals" : 仅当实参为字面量（如 `true`、`42`、`"hello"`）时显示
                ///   - "all"      : 始终显示所有参数的名称
                /// 类型: string
                /// 默认值: "literals"
                "enabled": "all",

                /// 当参数名与传入的变量名完全一致且仅带数字后缀时，
                /// 是否抑制提示。
                /// 例: `foo(index1)` 传入 `int index1` → 抑制提示。
                /// 类型: boolean
                /// 默认值: false
                "suppressWhenSameNameNumbered": true,
              },

              /// 是否显示局部变量及字段的类型内嵌提示。
              /// 例: `var foo = bar();` → 显示 `<String> foo = bar();`
              "variableTypes": {
                /// 类型: boolean
                /// 默认值: false
                "enabled": true,
              },

              /// 是否在方法调用处显示参数类型提示。
              /// 例: `process(null, null)` → `process(String name, Integer value)`
              "parameterTypes": {
                /// 类型: boolean
                /// 默认值: false
                "enabled": true,
              },

              /// 是否在格式化方法（`String#format`, `Console#printf` 等）的参数
              /// 旁显示其对应的格式化占位符（`%s`, `%d` 等）。
              "formatParameters": {
                /// 类型: boolean
                /// 默认值: false
                "enabled": true,
              },
            },

            "jdt": {
              "ls": {
                /// 是否启用 Protobuf（Protocol Buffers）文件支持。
                /// 类型: boolean
                /// 默认值: false
                "protobufSupport": {
                  "enabled": true,
                },

                /// 是否启用 javac 编译器（实验性）。
                /// 开启后使用 javac 而非 Eclipse JDT 内置编译器进行编译，
                /// 可获得与标准 javac 完全一致的编译行为。
                /// 类型: boolean
                /// 默认值: false
                "javac": {
                  "enabled": true,
                },
              },
            },

            "compile": {
              "nullAnalysis": {
                /// 用户自定义的 `@Nonnull` 注解类型（全限定名）。
                /// 用于空值分析，检测可能为 null 的赋值/传递。
                /// 类型: array of string
                /// 默认值: []
                "nonnull": ["javax.annotation.Nonnull", "org.eclipse.jdt.annotation.NonNull"],

                /// 用户自定义的 `@Nullable` 注解类型（全限定名）。
                /// 标记可为 null 的类型。
                /// 类型: array of string
                /// 默认值: []
                "nullable": ["javax.annotation.Nullable", "org.eclipse.jdt.annotation.Nullable"],

                /// 用户自定义的 `@NonNullByDefault` 注解类型（全限定名）。
                /// 标记包或类级别默认为非空。
                /// 类型: array of string
                /// 默认值: []
                "nonnullbydefault": [
                  "javax.annotation.ParametersAreNonnullByDefault",
                  "org.eclipse.jdt.annotation.NonNullByDefault",
                ],

                /// 空值分析模式。
                /// 可选值:
                ///   - "disabled"    : 禁用空值分析
                ///   - "interactive" : 检测到注解时提示用户是否启用分析
                ///   - "automatic"   : 检测到注解后自动启用分析
                /// 类型: string
                /// 默认值: "disabled"
                "mode": "automatic",
              },
            },
          },
        },
      },
    },

    "rust-analyzer": {
      // 通过 LSP 查询文件相关的任务。
      "enable_lsp_tasks": true,

      // 如果您同时处理多个不在同一个工作区中的 Cargo 项目，您可以告诉 rust-analyzer 这些项目：
      // "linkedProjects": ["./project-a/Cargo.toml", "./project-b/Cargo.toml"],

      "initialization_options": {
        // 内联提示。
        "inlayHints": {
          // 内联提示的最大长度。设为 null 表示无限制。默认值: 25
          "maxLength": null,

          // 生命周期省略提示的配置。
          "lifetimeElisionHints": {
            // 生命周期省略提示的启用时机。
            // 可选值:
            //   "always"       - 总是显示
            //   "never"        - 从不显示（默认）
            //   "skip_trivial" - 仅当返回类型涉及生命周期时显示
            "enable": "skip_trivial",

            // 是否优先使用参数名作为提示名称。默认值: false
            "useParameterNames": true,
          },

          // 闭包返回类型提示的启用时机。
          // 可选值:
          //   "always"     - 总是显示
          //   "never"      - 从不显示（默认）
          //   "with_block" - 仅当闭包包含代码块时显示
          "closureReturnTypeHints": {
            "enable": "always",
          },
        },

        // 诊断。
        "diagnostics": {
          // 是否启用实验性诊断（可能误报较多）。默认值: false
          "experimental": {
            "enable": true,
          },

          // 要禁用的诊断 ID 列表（例如 ["unresolved-proc-macro"]）。默认值: []
          "disabled": [],

          // 将某些警告提升为“信息”级别（蓝色波浪线）。默认值: []
          "warningsAsInfo": [],

          // 将某些警告提升为“提示”级别（淡化，不出现在问题面板）。默认值: []
          "warningsAsHint": [],
        },

        // 保存时检查。
        "check": {
          // 保存时执行的命令（例如改为 "clippy" 以运行更多 lint）。默认值: "check"
          "command": "clippy",

          // 特性设置：可设为字符串 "all"（--all-features）或字符串数组（如 ["serde"]）。
          // 默认值: null（继承自 cargo.features）
          "features": null,

          // 要忽略的检查诊断列表（例如 ["dead_code", "unused_imports"]）。默认值: []
          "ignore": [],
        },

        // Cargo。
        "cargo": {
          // 特性配置："all" 或具体数组（如 ["serde", "tokio"]）。默认值: 空数组（即只启用默认特性）
          "features": "all",

          // rust-analyzer 专用的 target 目录，避免与 cargo build 锁竞争。
          // 设为 true 表示使用 target/rust-analyzer 子目录，也可指定路径（如 "./target/ra"）。默认值: null（不隔离）
          "targetDir": true,
        },
      },
    },
  },
  // 配置代理面板中可用的代理服务器。
  "agent_servers": {
    "pi-acp": {
      "type": "registry",
    }
  },
}
```

# 结语

这套配置就三条取舍：语义能力交给语言服务器（高亮、折叠、大纲、格式化），界面噪音能关就关（协作按钮、遥测、确认弹窗），AI 权限按只读前缀放行、其余确认。每条都可以独立替换：不想用 LSP 语义就把 semantic_tokens、document_folding_ranges、document_symbols 三项改回 off，习惯全文件格式化就把 format_on_save 改成 on，想要更激进的 AI 权限就删掉 tool_permissions 直接 default: allow。

Zed 升级比较勤，配置项偶尔会改名或加默认值。隔一阵子拿自己的 settings.json 和默认配置对照一遍（全文就在上面），把失效的项清掉，配置才不会越积越旧。

配置写到顺手之后应该沉到水面以下。编辑器最理想的状态是你不记得它有什么配置，只记得代码写完没有报错。工具理应用来忘记，而不是用来折腾。如果这套配置能帮你少折腾两天，把省下的时间还给代码，这篇东西就没白写。
