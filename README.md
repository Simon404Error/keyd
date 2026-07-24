[![Kofi](https://badgen.net/badge/icon/kofi?icon=kofi&label)](https://ko-fi.com/rvaiya)

# 项目动机

[![Packaging status](https://repology.org/badge/tiny-repos/keyd.svg)](https://repology.org/project/keyd/versions)

Linux 缺乏一个好的按键重映射方案。为了实现满意的效果，通常需要组合使用多种工具（例如 xcape、xmodmap），最终结果往往被绑定在特定的显示环境（X11）中。keyd 试图通过提供一个灵活的系统级守护进程来解决这个问题，它使用内核级输入原语（evdev、uinput）来重映射按键。

# v2 说明

自第一个版本发布以来，配置格式经历了几次迭代。对于从 v1 迁移配置的用户，建议重新阅读[手册页](https://raw.githubusercontent.com/rvaiya/keyd/refs/heads/master/docs/keyd.scdoc)（`man keyd`）。

另请参阅：[更新日志](docs/CHANGELOG.md)。

# 目标

  - 速度       （用 C 编写的手动调优输入循环，耗时远小于 1ms）
  - 简洁性     （[配置格式](#示例配置)直观易懂）
  - 一致性     （修饰键默认[与层配合良好](https://github.com/rvaiya/keyd/blob/6dc2d5c4ea76802fd192b143bdd53b1787fd6deb/docs/keyd.scdoc#L128)）
  - 模块化     （UNIX 风格的核心，通过 [IPC](https://github.com/rvaiya/keyd/blob/90973686723522c2e44d8e90bb3508a6da625a20/docs/keyd.scdoc#L391) 机制可扩展）

# 功能特性

keyd 具有多项独特功能，其中许多通常只在自定义键盘固件（如 [QMK](https://github.com/qmk/qmk_firmware)）中才能找到，还有一些是 keyd 独有的。

一些比较有趣的功能包括：

- 层（支持[混合修饰键](https://github.com/rvaiya/keyd/blob/6dc2d5c4ea76802fd192b143bdd53b1787fd6deb/docs/keyd.scdoc#L128)）。
- 按键重载（点按/长按触发不同行为）。
- 针对特定键盘的配置。
- 即时重映射（无需刷写固件 :)）。
- 客户端-服务器模型，支持脚本编写和与显示服务器无关的应用程序重映射。（目前支持 X、sway 和 GNOME（Wayland））。
- 系统级配置（在虚拟终端中也能工作）。
- 一流的修饰键重载支持。
- Unicode 支持。

### keyd 适合以下人群：

 - 想要尝试自定义[层](https://docs.qmk.fm/feature_layers)（即自定义 shift 键）和单次修饰键。
 - 想在同一台机器上为多个键盘设置不同的布局。
 - 想要重新映射 `C-1` 而不破坏修饰键语义。
 - 想要一个易于理解的键盘配置文件格式。
 - 喜欢遵循 Unix 哲学的小型守护进程。
 - 想把 Control 和 Escape 键放在它们该在的位置。
 - 希望能够切换到虚拟终端调试问题而不破坏按键映射。

### keyd 不是：

 - 一个用于编程单个按键按下/释放事件的工具。

# 依赖

 - 你喜欢的 C 编译器
 - Linux 内核头文件（大多数系统已预装）

## 可选依赖

 - python（用于应用程序特定重映射）
 - python-xlib（仅 X 支持需要）
 - dbus-python（仅 KDE 支持需要）

# 安装

*注意：* master 分支是开发分支，版本之间可能会偶尔出现问题。发布版本有[标签](https://github.com/rvaiya/keyd/tags)标记，应视为稳定版本。

## 从源码安装

    git clone https://github.com/rvaiya/keyd
    cd keyd
    make && sudo make install
    sudo systemctl enable --now keyd

# 快速开始

1. 安装并启动 keyd（例如 `sudo systemctl enable keyd --now`）

2. 将以下内容放入 `/etc/keyd/default.conf`：

```
[ids]

*

[main]

# 将 Caps Lock 映射为：点按触发 Escape，按住触发 Control。
capslock = overload(control, esc)

# 将 Escape 键重映射为 Caps Lock
esc = capslock
```

可以使用 `keyd monitor` 命令获取按键名称。请注意，当 keyd 正在运行时，此命令的输出将对应 keyd 的输出。要查看原始输入事件，可以先停止 keyd，然后再运行该命令。更多细节请参阅手册页。

3. 运行 `sudo keyd reload` 重新加载配置集。

4. 更全面的说明请参阅手册页（`man keyd`）。

配置错误会出现在日志输出中，可以通过系统服务管理器以常规方式查看（例如 `sudo journalctl -eu keyd`）。

*注意：* 错误的配置文件可能导致机器无法使用。如果遇到这种情况，特殊按键序列 `backspace+escape+enter` 应该会使 keyd 终止。

某些鼠标（例如 Logitech MX Master）能够发送按键事件，因此会被通配符 id 匹配到。可能需要显式地将这些设备加入黑名单。

## 应用程序特定重映射（实验性功能）

- 将自己添加到 keyd 用户组：

	`usermod -aG keyd <用户名>`

- 编辑 `~/.config/keyd/app.conf`：

例如：

	[alacritty]

	alt.] = macro(C-g n)
	alt.[ = macro(C-g p)

	[chromium]

	alt.[ = C-S-tab
	alt.] = macro(C-tab)

- 运行：

	`keyd-application-mapper`

除非你正在使用 GNOME，否则你可能需要将 `keyd-application-mapper -d` 放在显示服务器初始化逻辑中的某个位置（例如 `~/.xinitrc`）。

更多细节请参阅手册页。

## 单板计算机支持

感谢 Giorgi Chavchanidze 的贡献，通过 usb-gadget 添加了对单板计算机（SBC）的实验性支持。

详情请参阅 [usb-gadget.md](src/vkbd/usb-gadget.md)。

## 软件包

部分发行版有第三方维护的软件包。如果你想将自己的软件包添加到列表中，请提交 PR。这些软件包由社区成员热心维护，作者不对其承担个人责任。

### Alpine Linux

[keyd](https://pkgs.alpinelinux.org/packages?name=keyd) 软件包由 [@jirutka](https://github.com/jirutka) 维护。

### Arch

[Arch Linux](https://archlinux.org/packages/extra/x86_64/keyd/) 软件包由 Arch 打包者维护。

### Debian

keyd 软件包在 Debian 13（"trixie"）及更高版本中可用。安装方法：

```shell
sudo apt install keyd
```

### Fedora

[COPR](https://copr.fedorainfracloud.org/coprs/alternateved/keyd/) 软件包由 [@alternateved](https://github.com/alternateved) 维护。

### Gentoo

[GURU](https://gitweb.gentoo.org/repo/proj/guru.git/tree/app-misc/keyd) 软件包由 [jack@pngu.org](mailto:jack@pngu.org) 维护。

### openSUSE
[openSUSE](https://software.opensuse.org//download.html?project=hardware&package=keyd) 软件包由 [@bubbleguuum](https://github.com/bubbleguuum) 维护。

使用 `sudo zypper in keyd` 即可轻松安装。

### Ubuntu

keyd 软件包在 Ubuntu 25.04（"plucky"）及更高版本中可用。安装方法：

```shell
sudo apt install keyd
```

此外，在 [`ppa:keyd-team/ppa` 存档](https://launchpad.net/~keyd-team/+archive/ubuntu/ppa)中还可以找到针对各 Ubuntu 版本反向移植的最新 Debian 软件包。

### Void Linux

[xbps](https://github.com/void-linux/void-packages/tree/master/srcpkgs/keyd) 软件包由 [@Barbaross](https://gitlab.com/Barbaross) 维护。

使用 `sudo xbps-install -Su keyd` 即可轻松安装。

# 示例 1

	[ids]

	*

	[main]

	leftshift = oneshot(shift)
	capslock = overload(symbols, esc)

	[symbols]

	d = ~
	f = /
	...

# 示例 2

以下配置覆盖了 macOS 用户可能更熟悉的特定 Alt 组合键，同时保持其余键不变。

	[ids]
	*

	[alt]

	x = C-x
	c = C-c
	v = C-v

	a = C-a
	f = C-f
	r = C-r
	z = C-z

# 推荐配置

许多用户可能不需要充分利用 keyd 的全部功能。对于那些只寻求简单生活品质提升的用户，我推荐以下配置：

	[ids]

	*

	[main]

	shift = oneshot(shift)
	meta = oneshot(meta)
	control = oneshot(control)

	leftalt = oneshot(alt)
	rightalt = oneshot(altgr)

	capslock = overload(control, esc)
	insert = S-insert

此配置将 Caps Lock 键重载为既可作为 Escape（点按时）也可作为 Control（按住时）使用，并将所有修饰键重映射为「单次」键。因此，要输入字母 A，现在只需按一下 Shift 再按 A，而不需要一直按住 Shift。最后，它将 Insert 重映射为 S-insert（在 X11 上粘贴）。

# 常见问题

## 为什么启用 keyd 后我的触控板会干扰输入？

libinput 是大多数 Wayland 和 X11 设置中使用的高级输入组件，它包含一个名为「打字时禁用」的功能，可在打字时禁用触控板。

为了实现这一点，它需要区分内置键盘和外接键盘，这是通过为特定硬件硬编码规则（「quirks」）来实现的。由于 keyd 创建了一个包含外接和内置键盘的虚拟设备，你需要指示 libinput 将 keyd 虚拟设备视为内置设备。

可以通过在 `/etc/libinput/local-overrides.quirks`（可能需要创建此文件）中添加以下内容来实现：

```
[Serial Keyboards]

MatchUdevType=keyboard
MatchName=keyd*keyboard
AttrKeyboardIntegration=internal
```

感谢 @mark-herbert42 和 @canadaduane 提供了原始解决方案。

## xmodmap/setxkbmap/* 怎么样？

xmodmap 及其同类是显示服务器级别的工具，功能有限。keyd 是一个系统级解决方案，实现了高级功能，如分层和[单次修饰键](https://docs.qmk.fm/#/one_shot_keys)。虽然一些 X 工具也提供类似的功能，但据我所知，没有哪个工具能像 keyd 一样灵活。

## [kmonad](https://github.com/kmonad/kmonad) 呢？

keyd 是几年前编写的，目的是让我能够在我日益庞大的键盘收藏上轻松尝试不同的布局。当时 kmonad 还不存在，只有像 [QMK](https://github.com/qmk/qmk_firmware)（keyd 的灵感来源）这样的自定义键盘固件才能获得类似的功能。我在发布 keyd 之后才了解到 kmonad。虽然 kmonad 是一个优秀且目标相似的项目，但它采用了不同的方法和设计理念。

值得注意的是，keyd 完全用 C 编写，以性能和简洁性为核心考量，并且很可能永远不会像 kmonad（可用 Haskell 扩展）那样可配置。尽管如此，keyd 用不到 2000 行 C 代码提供了（在作者看来）最有价值的功能，同时提供了简单且语言无关的配置格式。

## 为什么 keyd 不实现 X 功能？

如果你觉得缺少某些功能或发现了 bug，欢迎在 GitHub 上提交 issue。keyd 秉承极简（但合理）的设计哲学，有意省略某些功能（例如以 root 身份执行任意可执行文件）。QMK 等自定义键盘固件中已存在的功能是实现的好候选。

# 贡献

请参阅 [CONTRIBUTING](CONTRIBUTING.md)。
IRC 频道：oftc 上的 #keyd
