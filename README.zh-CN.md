# OpenNiri-Windows

[![CI](https://github.com/AdEx-Partners-DE/OpenNiri-Windows/actions/workflows/ci.yml/badge.svg)](https://github.com/AdEx-Partners-DE/OpenNiri-Windows/actions/workflows/ci.yml)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)

OpenNiri-Windows 是一款适用于 Windows 10/11、采用 Rust 构建的可滚动平铺式窗口管理器。

它将 Niri 风格的水平工作区模型带到原生 Windows，同时不替换 DWM。

## 产品定位

大多数 Windows 平铺式窗口管理器由树形/BSP 布局驱动。OpenNiri-Windows 则以滚动浏览为核心：

- 窗口排列在一条水平带状区域中。
- 显示器充当这条区域的视口。
- 添加窗口后，导航仍保持一致的空间关系。
- 你可以在工作区上下文中移动，而不必不断重建分割树。

## 适用人群

- 每天使用键盘管理大量窗口的用户。
- 使用单显示器或多显示器配置的工程师、分析师、运营人员和创作者。
- 希望使用架构透明、基于 Rust 构建的开源 Windows 平铺式窗口管理器的团队。

## 功能一览

目前已实现：

- 支持多显示器工作区，并提供感知显示器的焦点与移动命令
- 全局热键和实时配置重载
- 浮动与全屏切换
- 宽度预设（`Alt+1/2/3`）和均分（`Alt+0`）
- 平滑滚动动画、吸附提示和触控板手势（实验性功能，默认关闭）
- 可选的鼠标跟随焦点
- 系统托盘操作（暂停/重载/打开配置/打开日志/退出）
- 工作区持久化，以及更安全的关闭与恢复行为
- 用于故障排查的安全模式（`--safe-mode`）
- 内置诊断工具（`openniri-cli doctor`）

## 产品状态

OpenNiri-Windows 目前处于 **alpha** 阶段，正在积极开发中。

实际情况如下：

- 核心行为已实现，并在 CI 中测试。
- 目前的用户体验以键盘和配置为主（尚无完整的 GUI 设置流程）。
- 某些由 Windows 管理的窗口或系统窗口可能会拒绝移动或样式操作。
- 当前发布状态（截至 2026-02-08）：尚无已标记的公开版本，因此主要通过源码构建安装。

## 快速开始

### 选项 A：下载（发布后可用）

1. 打开 [GitHub Releases](https://github.com/AdEx-Partners-DE/OpenNiri-Windows/releases)
2. 如果已有已标记的版本，请下载最新的 `.zip` 压缩包
3. 将 `openniri.exe` 和 `openniri-cli.exe` 解压到某个文件夹
4. （可选）将该文件夹添加到 `PATH`
5. 生成默认配置：
   ```
   openniri-cli init
   ```
6. 启动守护进程：
   ```
   openniri-cli run
   ```

### 选项 B：从源码构建（当前 Alpha 阶段推荐）

前置条件：[Rust](https://rustup.rs) GNU 工具链（`stable-x86_64-pc-windows-gnu`）和 MinGW 链接器

```bash
rustup toolchain install stable-x86_64-pc-windows-gnu
git clone https://github.com/AdEx-Partners-DE/OpenNiri-Windows.git
cd OpenNiri-Windows
cargo +stable-x86_64-pc-windows-gnu build --release
cargo +stable-x86_64-pc-windows-gnu run -p openniri-cli -- init
cargo +stable-x86_64-pc-windows-gnu run -p openniri-cli -- run
```

工作区已在 `.cargo/config.toml` 中配置为使用 GNU（`x86_64-pc-windows-gnu`），生成的二进制文件位于 `target/release/`。

## 安装后的第一步

| 步骤 | 命令 | 功能 |
|------|---------|--------------|
| 创建配置 | `openniri-cli init` | 写入带注释的默认 `config.toml` |
| 启动守护进程 | `openniri-cli run` | 在后台启动窗口管理器 |
| 检查状态 | `openniri-cli status` | 显示版本、显示器、窗口和运行时间 |
| 切换暂停状态 | `openniri-cli toggle-pause`（别名：`pause`） | 暂停/恢复平铺，不停止守护进程 |
| 运行诊断 | `openniri-cli doctor` | 检查二进制文件、配置、守护进程和系统状态 |
| 停止守护进程 | `openniri-cli stop` | 请求关闭并等待拆除确认；如果未能确认，则在本地执行紧急恢复 |
| 本地紧急恢复 | `openniri-cli emergency-uncloak`（别名：`restore-windows`） | 不通过守护进程 IPC，尽力在本地恢复窗口可见性 |
| 重载配置 | `openniri-cli reload` | 无需重启即可应用配置更改 |

如果桌面变得无响应或焦点被困，请先使用非破坏性恢复方式：

```
openniri-cli panic-revert
openniri-cli status   # 在重新运行前，此命令必须失败（超时或提示“Daemon is not running...”）
```

如果 `panic-revert` 无法确认守护进程是否响应，它现在也会自动执行本地可见性恢复流程。  
如果关闭确认状态不明确，`openniri-cli stop` 现在也会使用相同的本地恢复保护机制。
你也可以使用 `openniri-cli recover` 作为 `panic-revert` 的简短别名。
如果无法访问终端，请使用系统托盘中的 **Emergency: Uncloak All Windows**，然后选择 **Exit**。

只有在确认守护进程已关闭后，才启动安全模式：

```
openniri-cli run --safe-mode
```

完整操作指南请参阅 [docs/GETTING_STARTED.md](docs/GETTING_STARTED.md)。  
有关故障恢复的详情，请参阅 [docs/SUPPORT_PLAYBOOK.md](docs/SUPPORT_PLAYBOOK.md)。

## 默认热键

Windows 11 保留了大多数 `Win+字母` 组合键（`Win+L` 永远无法注册），因此默认热键与 GlazeWM 一样使用 `Alt`。现有配置文件会保留其中已有的绑定。无法注册的热键会按名称记录到日志中，其余热键仍可正常使用。

| 按键 | 操作 |
|---|---|
| `Alt+H / Alt+L` | 聚焦左侧 / 右侧 |
| `Alt+J / Alt+K` | 聚焦下方 / 上方 |
| `Alt+Shift+H / Alt+Shift+L` | 将列向左 / 向右移动 |
| `Alt+Ctrl+H / Alt+Ctrl+L` | 缩小 / 加宽列 |
| `Win+Ctrl+Escape` | 紧急恢复窗口可见性 + 守护进程 panic-revert |
| `Win+Alt+H / Win+Alt+L` | 聚焦左侧 / 右侧显示器 |
| `Win+Alt+Shift+H / Win+Alt+Shift+L` | 将窗口移至左侧 / 右侧显示器 |
| `Alt+Shift+Q` | 关闭当前聚焦窗口 |
| `Alt+F` | 切换浮动状态 |
| `Alt+Shift+F` | 切换全屏状态 |
| `Alt+1 / Alt+2 / Alt+3` | 将宽度设为 1/3、1/2、2/3 |
| `Alt+0` | 均分所有列的宽度 |
| `Alt+Shift+R` | 刷新（重新枚举窗口） |

## 随 Windows 自动启动

```bash
openniri-cli autostart enable
```

此命令会在 `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` 下写入注册表项，以便登录时启动守护进程。禁用自动启动：

```bash
openniri-cli autostart disable
```

## 配置与运行时路径

配置文件：

- `%APPDATA%\openniri\config\config.toml`

状态数据：

- `%APPDATA%\openniri\data\workspace-state.json`

守护进程日志：

- `%TEMP%\openniri-daemon.log`
- `%TEMP%\openniri-daemon.err.log`

## 架构

OpenNiri-Windows 是一个 Rust 工作区：

| Crate | 职责 |
|---|---|
| `openniri-core-layout` | 与平台无关的布局引擎 |
| `openniri-platform-win32` | Win32 集成和窗口操作 |
| `openniri-ipc` | 命名管道命令/响应协议 |
| `openniri-daemon` | 运行时事件循环和状态管理 |
| `openniri-cli` | 面向用户的命令行界面 |

技术文档：

- [入门指南](docs/GETTING_STARTED.md) - 分步操作指南
- [配置说明](docs/CONFIGURATION.md) - 完整配置参考和示例
- [故障排查](docs/TROUBLESHOOTING.md) - 常见问题和调试指南
- [规范](docs/SPEC.md) - 设计规范
- [架构](docs/ARCHITECTURE.md) - 架构概览
- [Windows 限制](docs/WINDOWS_CONSTRAINTS.md) - Win32 平台限制

## 平台限制

OpenNiri-Windows 是一个**窗口控制器**，而不是合成器。

- DWM 仍负责合成。
- 提权或受保护的窗口可能会拒绝位置或样式更改。
- 不同应用框架（Win32/WPF/Electron/UWP）的行为可能有所不同。

## 参与贡献

请参阅 `CONTRIBUTING.md`。

## 许可证

GPL-3.0。请参阅 `LICENSE`。
