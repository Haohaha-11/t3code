# Windows 与 macOS 跨设备使用 T3 Code

这份记录基于 T3 Code `0.1.11`，覆盖 Windows 客户端、本机 agent 和两台经 Tailscale 可达的 Mac。示例中的机器名、路径和用户均为占位符。

## 多台设备继续同一条会话

T3 会话归创建它的环境。Windows、手机或浏览器接到同一个 T3 后端，可查看并继续同一条会话。项目路径、当前分支、provider 进程和登录状态都由后端所在的机器及其 OS 用户决定。客户端同时连上多个环境时，可以在侧边栏看到来自不同后端的会话。打开会话的环境标识，便可以核对这条会话实际在哪台机器上运行。

用桌面端经 SSH 添加环境时，确认选对实际使用 agent 的远端用户。同一台 Mac 上的两个 macOS 用户可以各有自己的 T3 和 provider 状态。保持该环境和连接路线可用，Windows 或手机才能继续原来的会话。参见[远程环境说明](https://github.com/pingdotgg/t3code/blob/v0.1.11/docs/user/remote-access.md)。

## Provider 接入与验证

先在 T3 的 **Settings → Providers** 选择要执行任务的环境，再检查 provider 路径和登录状态。一次最小验证可以在该环境新建会话，要求 agent 只返回一个约定字符串，随后核对会话的环境标识、provider 和回复。先验证路线，再运行会改文件的任务。

| 执行环境 | Provider | 实践结果 |
| --- | --- | --- |
| Windows 本机 | Codex、带百炼模型配置的 OpenCode | 已从 Windows 的命名项目验证调用 |
| Mac A 经 SSH | Antigravity CLI（`agy`） | Windows 客户端能看到后端已有会话并获得模型回复 |
| Mac B 经 SSH | Claude Code | 在目标 macOS 用户下完成授权，随后从 Windows 客户端获得 Claude 回复 |
| Windows 本机 | Grok Build | T3 已识别本机 CLI，首次登录和模型调用待验证 |

Grok Build 在 Windows 上使用原生 CLI。在实际执行的 Windows 用户下运行 `grok login`，用有相应订阅权益的账号授权，然后回 T3 刷新 provider 状态。Windows 客户端可以通过远程 T3 使用 Mac 上已有的 provider 登录。

如果无项目草稿的设备选择器无法选中 Windows，本机已登记的普通项目仍可作为入口。从该项目新建一条会话并核对环境标识，再选择本机 provider。参见[安装与 provider 文档](https://github.com/pingdotgg/t3code/blob/v0.1.11/docs/user/install.md#providers)。

## 手机接入

在 [T3 Code 官方安装说明](https://github.com/pingdotgg/t3code/blob/main/docs/user/install.md#mobile-app)打开 App Store 或 Google Play 的下载入口。在后端机器的 **Settings → Sharing & access** 检查可用路线。手机可通过 T3 Connect 或后端生成的配对链接添加环境。使用直接配对时，手机也需要能访问配对链接里的服务地址。

桌面端保存的一条 SSH 连接只在该桌面端提供隧道。手机要连接同一个环境，仍需要由主机提供手机可达的 T3 路线。配对后可以在两台客户端打开同一条测试会话，检查标题、回复及环境是否一致。

## 跨设备协作

桌面客户端连接远端环境，解决的是人从哪个设备操作。跨设备分派还需要让 T3 **后端之间**建立 Linked environments。`0.1.11` 将其列为 Beta。两端分别在 **Settings → Labs** 启用，在发起端的 **Settings → Environments → Linked environments** 配对并选择访问范围。这条链接有方向，接收端能撤销。发起端后端还需要能到达接收端的服务地址。已有 Tailscale 和 SSH 可以提供网络路线，agent 权限仍由 T3 配对授予。参见[链接环境文档](https://github.com/pingdotgg/t3code/blob/v0.1.11/docs/user/remote-access.md#link-another-environment)。

已获得 T3 编排 MCP 工具的 agent 可以通过 `delegate_task` 创建子会话并收回结果，用 `t3_thread_launch` 创建独立会话，也可以按权限给现有会话发消息、读结果或等待。工具会检查当前会话的权限和目标 provider。手工连接三台设备之后，仍需实际验证这些机器的 peer link 和权限。参见[编排 MCP 文档](https://github.com/pingdotgg/t3code/blob/v0.1.11/docs/orchestration-v2/orchestrator-mcp-server.md)。

可以用同一个小 Git 仓库来验证协作。三台机器各自 checkout，先确认使用同一提交。让主会话把一个只读检查交给已链接的环境，并要求报告提交哈希、一项发现和子会话结果。确认结果回到主会话以后，再尝试改代码的任务。多机修改时，用 commit 和独立分支交换变更，避免让多台服务器同时写同一个工作目录或会话数据库。T3 会把任务与结果转交给对方，未提交的文件改动仍留在原工作区。
