---
title: COSMIC 桌面系统语言中文化排障记录
tags:
  - 生活指南
  - Linux
  - Fedora
  - Atomic
  - COSMIC
  - 博客
  - 207）
生活场景: Linux
share: true
date: 2026-08-23T16:40:00
lastmod: 2026-08-31T01:05:00
cover:
    image: ../../images/COSMIC 桌面系统语言中文化排障记录.webp
categories: 生活指南
---

# 前言

我的电脑装的是 Fedora COSMIC Atomic（基于 rpm-ostree 的原子化桌面发行版）。装系统时图省事选了英文，用了大半个月想切回中文，原以为一条 `localectl set-locale LANG=zh_CN.UTF-8` 就能搞定。结果前前后后踩了四个坑：桌面还是英文、environment.d 里早写好的配置不生效、应用列表是英文名、文件管理器的文件夹也是英文名。每一个坑的成因都不同，还牵扯到两个上游项目的已知 bug。

这个问题没有看起来那么简单：**从系统配置到最终显示要经过五层传递，每一层都可能改写结果**。文章按“问题 → 排查 → 根因 → 修复”的顺序记录，重点是排查思路和读源码的方法。第五层（文件管理器文件夹名）只有根因、没有可用修复，正文会说明。

文章分两种读法：

- **只想修好问题**：直接看“修复速查”，照抄即可
- **想看懂为什么**：从“背景知识”开始，每一节先给结论，再展开分析过程

另外因为是原子化系统，所有修复都刻意放在 `~/.config` 和 `/etc`（原子化系统上依然可写）里，没有一处需要改动系统镜像，`rpm-ostree` 用户可以直接照抄。

# 修复速查

| 症状 | 根因 | 修复 |
|------|------|------|
| 桌面界面还是英文 | 登录时 Fedora 的 lang.sh 触发 VT 控制台回退，把 LANG 改回 en_US | 用户 shell 配置里重设 `LANG`（见第二层） |
| 面板应用列表是英文名 | 应用列表的 locale 匹配逻辑不处理 `.UTF-8` 后缀，且读取了拼写错误的变量 | 设置 `LANGUAGES=zh_CN`（见第四层） |
| 文件管理器文件夹是英文 | COSMIC Files 未实现 xdg-user-dirs 显示名翻译 | 暂无可用修复，等上游（见第五层） |

# 背景知识

看懂这篇文章只需要两个概念。

**概念一：locale 是“语言 + 地区习惯 + 编码”的组合。** `zh_CN.UTF-8` 拆开看：`zh` 是语言（中文），`CN` 是地区（中国大陆，决定日期写成“2026 年 8 月 23 日”还是“Aug 23, 2026”），`UTF-8` 是字符编码（保证中文不乱码）。系统通过一组环境变量（`LANG`、`LC_TIME`、`LC_MESSAGES` 等）把 locale 传递给程序：每个程序启动时读取这些变量，决定界面语言、日期格式、数字格式等行为。

**概念二：环境变量由父进程传给子进程，中途可以被改写。** 开机时 systemd（所有进程的祖先）从 `/etc/locale.conf` 读取初始值，然后沿着“登录管理器 → 登录 shell → 桌面会话 → 各个应用”逐层传递。**任何一环改了值，后面所有进程拿到的都是改过的值。** 因此排查思路很直接：找到值在哪一环被改写。

一个实用的细节：`locale` 命令的输出里，**不带引号的变量是“被显式设置过”的，带引号的是未设置、跟随 `LANG` 的派生值**（`LANG` 行本身永远不带引号）。后文排查会用到这个细节。

# 第一层：系统 locale 本身

这一步最简单，没有任何坑：

```bash
localectl set-locale LANG=zh_CN.UTF-8
```

命令写入 `/etc/locale.conf`，重启后 systemd 的环境变量即为 `LANG=zh_CN.UTF-8`，用 `systemctl show-environment` 可以确认系统层面的 locale 已是中文。（本机 locale.conf 里另外还有 9 行 `LC_*` 变量，是早前配置语言时写入的，不是这条命令加的；第二层定位正好用到它们。）另外需确认中文 locale 已生成（`locale -a` 里有 `zh_CN.utf8`），没有的话先运行 `localedef` 生成。

# 第二层：登录会话为什么还是英文

**结论**：系统配置是中文，但登录后会话环境是英文——`LANG` 在登录过程中被 `/etc/profile.d/lang.sh` 里的 VT 控制台回退逻辑改写了。该逻辑本意是防止纯文字终端显示 CJK 乱码，但 greetd 启动图形会话时错误地满足了它的触发条件。

## 第一现场：locale 输出里的引号

重启登录后运行 `locale`：

```bash
$ locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"      # 带引号 → 未被显式设置，跟随 LANG 派生
LC_NUMERIC=zh_CN.utf8       # 不带引号 → 被显式设置过（来自 locale.conf）
LC_TIME=zh_CN.utf8
...
```

这个组合很不寻常：`LC_TIME` 等 9 个变量是 zh_CN（明显来自 `/etc/locale.conf`），唯独 `LANG` 是 en_US。这说明**有人读取了 locale.conf，却唯独把 LANG 换掉了**。后来定位根因，主要依据就是这个特征。

## 顺着进程树逐个检查 /proc/PID/environ

`/proc/<pid>/environ` 记录进程启动那一刻的完整环境变量，不含任何后续修改和继承干扰，是排查环境变量问题的第一手证据。顺着进程树逐个检查：

```bash
loginctl list-sessions
pstree -p <session-leader-pid>
```

我的环境里链条是：

```
greetd(登录管理器)
 └─ /bin/sh -c "exec fish -c start-cosmic ..."   ← greetd 启动的登录 shell
     └─ fish(用户登录 shell)
         └─ cosmic-session(桌面会话)
```

逐个读取 environ 发现：`/bin/sh` 启动时环境里就已经是 `LANG=en_US.UTF-8`。**改写发生在 greetd 到 /bin/sh 之间**。嫌疑范围收窄到三处：greetd 本身、greetd 调用的登录界面程序（cosmic-greeter）、PAM（负责登录认证的系统组件）。

## 排除法：读源码逐一排除嫌疑

**嫌疑一：cosmic-greeter。** 它在用户登录时负责启动会话，有机会注入环境变量。拉取源码检查会话构建逻辑：

```bash
git clone https://github.com/pop-os/cosmic-greeter
grep -n "env" src/greeter.rs
```

结果它构造会话环境时只传入 `XDG_SESSION_TYPE`、`XDG_CURRENT_DESKTOP` 这类桌面相关变量，完全没有触碰 LANG。排除。

**嫌疑二：PAM。** 检查 greetd 的 PAM 配置（`/etc/pam.d/greetd`）：`pam_env.so` 读取的 `/etc/environment` 是空文件，`pam_env.conf` 里也没有相关设置。另外通读了 systemd 259 的 `pam_systemd.c` 源码——它会应用用户记录（user record）里的 `preferredLanguage` 字段，但 `userdbctl user --json=pretty` 显示我的用户记录里没有语言字段。排除。

**嫌疑三：fish。** 登录 shell 是 fish，怀疑它启动时自行设置 LANG。用最小实验验证——给它一个完全干净的环境，观察它自己的行为：

```bash
env -i HOME=$HOME PATH=/usr/bin:/bin fish -c 'env | grep -E "^LANG|^LC_"'
```

结果：环境里没有 LANG 时，fish 会主动读取 `/etc/locale.conf` 并正确导出 `LANG=zh_CN.UTF-8`。fish 的行为是正确的。但这个实验制造了一个矛盾：三处嫌疑都有不在场证明，`en_US.UTF-8` 到底从哪来？

## 突破：读 greetd 源码，发现漏掉的一环

矛盾促使我回头通读 greetd 的 session worker 源码（上游 kennylevinsen/greetd 0.10.3）：

```bash
git clone https://github.com/kennylevinsen/greetd
sed -n '140,290p' greetd/src/session/worker.rs
```

两个发现。第一，会话的最终环境 = greeter 传入的变量 + PAM 环境（`execve` 时只使用了 `pam.getenvlist()` 的结果）。第二，启动命令的构造逻辑里有一个 `source_profile` 开关：

```rust
let command = if source_profile {
    format!(
        "[ -f /etc/profile ] && . /etc/profile; [ -f $HOME/.profile ] && . $HOME/.profile; exec {}",
        cmd.join(" ")
    )
} else {
    format!("exec {}", cmd.join(" "))
};
```

而 `source_profile` 是 greetd 的 `[general]` 配置项，默认 `true`（本机 `/etc/greetd/config.toml` 没有设置它，走默认值）。**也就是说，登录 shell 启动前会先 source `/etc/profile` 下的所有脚本**。这就是之前漏掉的一环，环境变量在这里多经过了一次处理。

## 真相：lang.sh 的 VT 控制台 CJK 回退

重新审视 Fedora 的 `/etc/profile.d/lang.sh`，这次不再一扫而过：

```sh
# 仅在虚拟终端（/dev/tty*）上生效：内核 VT 字体显示不了 CJK，强制用英文
# （节选：实际脚本还有非 UTF-8 分支和更多语言；脚本末尾注明此处刻意不 export LANG，
#   但 LANG 在此前已处于导出状态，赋值照样会传给子进程）
if [ -n "${LANG}" ] && [ "${TERM}" = 'linux' ] && /usr/bin/tty | /usr/bin/grep --quiet -e '/dev/tty'; then
    case ${LANG} in
        zh*)  LANG=en_US.UTF-8 ;;
        ja*)  LANG=en_US.UTF-8 ;;
        ko*)  LANG=en_US.UTF-8 ;;
        ...
    esac
fi
```

所有线索对上了：

- `source_profile=true` → `/etc/profile` 被 source → lang.sh 执行。那 9 个 LC_* 变量来自 `/etc/locale.conf`（systemd 开机读取并传给所有进程，lang.sh 执行时也会再导入一遍），唯独 LANG 被后面的回退逻辑换掉
- greetd 给会话传入的 `TERM` 默认是 `linux`，且会话运行在 tty1 上 → 触发条件全部满足
- `zh*)` 分支把 `LANG` 改回 `en_US.UTF-8` → 与现场观察到的特征完全吻合

这段代码的设计意图是合理的：纯文字控制台（`/dev/tty*`）的内核字体渲染不了中日韩文字，强制回退英文以避免乱码。但 greetd 启动的是**图形会话**，图形环境完全可以正常显示中文，脚本却无法区分两者，误判了。

后续搜索确认这是已知上游 bug：[pop-os/cosmic-session#207](https://github.com/pop-os/cosmic-session/issues/207)（“start-cosmic login shell on tty1 triggers Fedora's lang.sh VT locale fallback”）。COSMIC 的 `start-cosmic` 脚本会刻意用 `exec -l $SHELL` 再启动一次用户的登录 shell（目的是收集用户 shell 的环境变量），这个登录 shell 恰好撞上了 lang.sh 的误判。

## 修复：在 lang.sh 之后重新设置

官方 issue 给出的 workaround 原理很直接：**lang.sh 先执行，用户 shell 配置后执行——在用户配置里把 LANG 重新设回中文，后执行的覆盖先执行的**。

我的 shell 是 fish，写入 `~/.config/fish/config.fish`：

```fish
set -gx LANG zh_CN.UTF-8
```

bash 用户写 `~/.bash_profile`，zsh 用户写 `~/.zprofile`：

```bash
export LANG=zh_CN.UTF-8
```

之后 `start-cosmic` 将正确的值导入整个桌面会话，所有应用受益。注销重登后，桌面界面文本终于全中文（应用列表与文件管理器的问题见第四、五层）。

# 第三层：environment.d 为什么救不了

**结论**：environment.d 中设置的变量，会在每次登录时被 start-cosmic 重新导入的会话值覆盖。登录 shell 里被污染的 `LANG` 经过 `systemctl --user import-environment` 进入用户管理器，把 environment.d 里的同名设置顶掉。

排查中我发现 `~/.config/environment.d/locale.conf` 里其实早就写过 `LANG=zh_CN.UTF-8` 和 `LANGUAGE=zh_CN`，但会话里的 LANG 依然是英文。为什么部分失效？

机制在导入时序，不在优先级：用户管理器启动时先应用 environment.d（此时的 `LANG=zh_CN.UTF-8` 本来生效），但 start-cosmic 随后用 `exec -l $SHELL` 再起一次登录 shell 收集环境变量（见第二层），并在脚本末尾把它看到的变量逐个 `systemctl --user import-environment` 重新导入用户管理器。登录 shell 里的 `LANG` 已被 lang.sh 改成英文，于是 environment.d 的值在每次登录时都被这个显式导入覆盖。environment.d 本身的优先级并不低：运行 `systemctl --user daemon-reload` 时它会重新应用，反覆盖用户管理器的当前值。只是它救不了每次登录都会被重新污染的 `LANG`。

实测两行配置命运不同：

| 变量 | 结果 |
|------|------|
| `LANGUAGE=zh_CN` | ✅ 生效——登录 shell 里没有这个变量，start-cosmic 不会重新导入它，environment.d 的值被保留。它只影响 gettext 消息翻译；第二层修复后 `LANG` 已是中文，属于冗余保险（gettext 程序优先读取它） |
| `LANG=zh_CN.UTF-8` | ❌ 正常登录时被 start-cosmic 重新导入的 `en_US.UTF-8` 覆盖；没有登录 shell 的环境（如 linger 服务）下会生效 |

结论：environment.d 适合设置登录 shell 里**没有**的变量（不会被 start-cosmic 重新导入），但无法修正每次登录都会被重新污染的 `LANG`。最终处理：三行都保留——`LANG` 作为无登录 shell 场景（linger 服务等）的兜底，`LANGUAGE` 作为 gettext 翻译保险，`LANGUAGES` 用于修复应用列表。

# 第四层：应用列表的英文名

**结论**：应用列表使用的 freedesktop-desktop-entry 库存在两个 bug——locale 匹配不处理 `.UTF-8` 编码后缀，且读取的第二个环境变量名拼写错误。两者叠加导致 `Name[zh_CN]` 本地化键被丢弃，应用名回落到英文。

## 先确认版本，再读对应的源码

读上游源码最忌讳读错版本——master 分支的行为可能和发行版打包的版本完全不同。应先确认本机安装的版本及对应的上游 commit：

```bash
rpm -q cosmic-applets               # 确认版本号（包名是 cosmic-applets，不是 cosmic-app-list）
rpm -qf /usr/bin/cosmic-app-list    # 确认属于哪个包（该二进制是指向 cosmic-applets 多合一可执行文件的符号链接）
```

再查 Fedora 打包规范里锁定的上游 commit：

```bash
curl https://src.fedoraproject.org/rpms/cosmic-applets/raw/f44/f/cosmic-applets.spec
```

spec 文件里有 `%global commit 73d7278…`，拿着这个 commit 拉取对应的精确源码，读到的代码才和本机运行的二进制一致。

## 读源码：两个 bug 叠加

COSMIC 应用列表使用 `freedesktop-desktop-entry` crate（锁定版本 0.8.1）解析 `.desktop` 文件。应用显示名来自文件里的本地化键：

```ini
Name=Files
Name[zh_CN]=文件
```

获取语言列表的函数（0.8.1 原文）：

```rust
pub fn get_languages_from_env() -> Vec<String> {
    let mut l = Vec::new();
    if let Ok(lang) = std::env::var("LANG") {
        l.push(lang);                       // 原样放入 "zh_CN.UTF-8"
    }
    if let Ok(lang) = std::env::var("LANGUAGES") {  // ← 注意：LANGUAGES 是拼写错误！
        lang.split(':').for_each(|lang| l.push(lang.to_owned()));
    }
    l
}
```

第一个 bug 在这个函数里，第二个 bug 在解析器（decoder.rs）。解析 `.desktop` 文件时，语言过滤器只保留完全匹配的键：

```rust
// add_generic_locales: 对每个 locale，若含 '_' 再追加语言部分
// 输入 ["zh_CN.UTF-8"] → 过滤器 ["zh_CN.UTF-8", "zh"]
// 解析时：locale == "zh_CN" 不在过滤器中 → Name[zh_CN] 键被丢弃！
match locales_filter {
    Some(filter) if !filter.iter().any(|l| *l == locale) => return Ok(()),
    _ => (),
}
```

两个 bug 叠在一起的效果：

1. `LANG` 的值是 `zh_CN.UTF-8`（带 `.UTF-8` 后缀），而 `.desktop` 里的键是 `Name[zh_CN]`（不带后缀）→ 解析阶段 `Name[zh_CN]` 键就被过滤器丢弃了
2. 函数读取的第二个变量是拼写错误的 `LANGUAGES`，标准变量 `LANGUAGE` 它根本读取不到——所以把 `LANGUAGE` 设对也没用
3. 查找时语言列表里只有 `zh_CN.UTF-8` 和 `zh`，都匹配不上 → 回落到默认英文 `Name`

顺带对比了同桌面另一个入口：按 Super 键打开的应用库（cosmic-app-library）没有这个问题，因为它自己会 `l.split(".").next()` 把编码后缀去掉再查。**同一桌面两个入口、两套实现、两种行为**——排查时不要想当然，必须逐个看代码。

## 修复 + 用最小实验验证

**修复**：设置该库真正读取的变量 `LANGUAGES=zh_CN`。这个变量名不标准，除了这个有 bug 的库没有程序会读它，设了没有副作用。过滤器因此包含 `zh_CN`，`Name[zh_CN]` 键被保留，精确匹配命中。

写入两处，覆盖会话直启进程和 systemd 用户服务：

```fish
# ~/.config/fish/config.fish
set -gx LANGUAGES zh_CN
```

```ini
# ~/.config/environment.d/locale.conf
LANGUAGES=zh_CN
```

**验证**：重新登录前，先用 Python 按源码的精确算法模拟一遍匹配过程（过滤器构建 → 键保留 → 查找回退），输入本机真实的 `.desktop` 文件：

```
修复前：过滤器 ['zh_CN.UTF-8', 'zh']          → 键被丢弃 → "COSMIC Files"
修复后：过滤器 ['zh_CN.UTF-8', 'zh', 'zh_CN'] → 精确命中 → "COSMIC 文件管理器"
```

模拟通过后再重新登录，一次成功。

# 第五层：文件管理器的英文文件夹名

**结论**：前四层是功能异常，这一层是功能缺失——COSMIC Files 尚未实现 xdg-user-dirs 显示名翻译，侧边栏直接显示磁盘上的真实目录名。本机版本（1.6.0）上这一层没有可行的修复，本节只记录根因。

标准的 XDG 用户目录机制中，这类“翻译名”由 `~/.config/user-dirs.locale` 控制：磁盘上的目录名就是 `Downloads`，文件管理器读取该文件决定显示哪个语言的名字。修改之前先拉源码确认 COSMIC Files 是否实现了这个机制——结果它没有。侧边栏条目的显示名逻辑（1.6.0，src/app.rs 的 `update_nav_model`）：

```rust
// 侧边栏每个收藏条目的显示名：只有"主目录"走了翻译（fl!("home")），
// 下载/文档/音乐/图片/视频等特殊目录直接取磁盘目录名
let name = if matches!(favorite, Favorite::Home) {
    fl!("home")
} else if let Favorite::Network { name, .. } = favorite {
    name.clone()
} else if let Some(file_name) = path.file_name().and_then(|x| x.to_str()) {
    file_name.to_string()
} else {
    fl!("filesystem")
};
```

特殊目录（下载、文档、音乐、图片、视频）的显示名直接取自**磁盘上的真实目录名**，完全不查询 `user-dirs.locale`，也没有调用 `xdg-user-dirs` 的翻译目录。这个翻译机制并非所有文件管理器都实现了（Nautilus、GTK 文件选择对话框会读取它，不少其他桌面环境的文件管理器也不实现）。**所以修改 `user-dirs.locale` 对 COSMIC Files 无效，不做这个设置**。

侧边栏重命名这条路也走不通：1.6.0 的侧边栏右键菜单只有打开、移除等条目，没有“重命名”项。上游在 1.6.0 发布之后（2026-08-21 合入 master）才加入侧边栏自定义显示名功能（`ChangeSidebarLabel`），本机版本拿不到。也就是说这一层目前只能定位根因、等待上游。

不推荐的方案是 `xdg-user-dirs-update --force`——它会把磁盘目录**真的**重命名为中文。虽然会同步更新 `user-dirs.dirs` 让 xdg 系应用继续找到目录，但浏览器、下载工具等保存的是字面路径（`/home/用户名/Downloads`），重命名后这些默认路径全部失效，后患较多。

要彻底解决只能等上游把 xdg-user-dirs 翻译机制补上；更新到 1.6.0 之后的版本后，也可以先用侧边栏右键“重命名”（自定义显示标签，不碰磁盘路径）作为临时手段。

# 修复清单

| 层 | 问题 | 根因 | 修复 |
|----|------|------|------|
| 系统 | locale 非中文 | 未设置 | `localectl set-locale LANG=zh_CN.UTF-8` |
| 会话 | 登录后 LANG 变回英文 | lang.sh 的 VT 控制台回退误判（COSMIC bug #207） | 用户 shell 配置里重设 `LANG` |
| 环境 | environment.d 的 LANG 每次登录被 start-cosmic 重新导入的值覆盖、LANGUAGE 冗余 | start-cosmic 登录后把登录 shell 的变量重新导入用户管理器；LANGUAGE 只影响消息翻译 | environment.d 保留三行：`LANG`（兜底）、`LANGUAGE`（保险）、`LANGUAGES`（修复） |
| 应用列表 | 应用名英文 | freedesktop-desktop-entry 的匹配 bug + `LANGUAGES` 拼写错误 | 设 `LANGUAGES=zh_CN` |
| 文件管理器 | 文件夹显示英文 | COSMIC Files 未实现 xdg-user-dirs 翻译 | 暂无（等上游实现，见第五层） |

# 排查方法论

回头看，这类“语言设置不生效”的问题，通用排查顺序是**从外到内逐层定位**：

1. **系统层**：`localectl status`、`cat /etc/locale.conf`、`locale -a`——确认系统 locale 已配置且可用
2. **会话层**：登录后 `locale` 输出与系统层对比。不一致说明会话启动链上有人改了环境变量。注意引号细节：不带引号的变量是被显式设置过的（带引号的是跟随 `LANG` 的派生值），是重要的排查线索
3. **进程层**：顺着进程树逐个看 `/proc/<pid>/environ`，找到变量在哪一环被改写。`pstree -p`、`loginctl` 是定位进程链的好工具
4. **配置层**：常见改写点依次是 PAM（`/etc/environment`、`pam_env`）、`/etc/profile.d/*`、用户 shell 配置、environment.d（会被 start-cosmic 事后重新导入的值覆盖，见第三层）
5. **应用层**：只有个别应用不跟随，多半是应用自身逻辑。直接读上游源码比对行为，比反复试配置高效得多

四个经验：

用最小实验验证假设，而不是反复重启试错。排查 fish 的行为时，一条 `env -i … fish -c '…'` 就能在干净环境下观察它自己会做什么；验证修复方案时，用 Python 模拟源码里的匹配算法，几分钟确认结果，省掉一次“改配置 → 注销重登 → 看效果”的循环。实验越隔离，结论越可靠。

读源码一定要对版本。这次 app-list 的 bug 涉及发行版打包的精确 commit（从 Fedora spec 文件的 `%global commit` 拿到），直接读 master 分支很可能读到已经修过的代码，白忙一场。先 `rpm -q` 确认版本，再拉对应 tag/commit 的源码，是排查发行版软件问题的基本功。

排除法靠“环境特征”而不是直觉。会话里 `LC_TIME=zh_CN` 但 `LANG=en_US.UTF-8` 的组合，说明改写者“读取了 locale.conf 却只改了 LANG”——这个特征直接排除了“未读配置就设置默认值”的嫌疑，把范围锁定到 lang.sh 这类脚本上。

遇到诡异行为先搜上游 issue。这次的两个坑（lang.sh 回退、LANGUAGES 拼写错误）都是别人踩过、上游有记录的，搜到 issue 能省掉大量从零推断的时间。带着已经掌握的线索去搜（比如“cosmic locale zh_CN en_US”），命中率远高于泛泛搜索。

# 后续维护

这些都是 workaround，属于绕过上游 bug。等到 COSMIC 修复 `cosmic-session#207`、`freedesktop-desktop-entry` 修正语言匹配逻辑后，可以逐步清理：

- fish 配置里的 `set -gx LANG zh_CN.UTF-8` 可以删掉
- `LANGUAGES=zh_CN` 可以删掉
- environment.d 里的 `LANG`、`LANGUAGE`、`LANGUAGES` 三行属于正常配置，永久保留
- COSMIC Files 的文件夹显示名目前没有任何本地修复，等上游实现 xdg-user-dirs 翻译即可；更新到 1.6.0 之后的版本后，也可以用侧边栏右键“重命名”临时改显示标签

如果只是日常使用，前四层修复后系统界面已是全中文，只剩文件管理器的文件夹名保持英文；彻底中文化留给上游。
