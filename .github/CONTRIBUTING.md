# 为 RemoteDispatch 做贡献

感谢你有意参与贡献。本文档介绍如何搭建环境、代码库的组织结构，以及提交变更时需要留意的事项。

---

## 项目概览

三个主要组成部分：

- **游戏监视器**（`Engine/`）挂接 Unity 事件与 Harmony 补丁，用于检测状态变化。
- **会话系统**（`Server/Session.cs`、`Server/AsyncSet.cs`）跟踪每个已连接客户端（浏览器会话）尚需接收的数据，采用基于标签的长轮询机制，而非持续轮询。
- **HTTP 服务器**（`Server/HttpServer.cs`）把游戏状态序列化为 JSON 推送给客户端，并将传入的控制命令在主线程路由回游戏。

另有一个可选的 **Signals 子项目**（`RemoteDispatch.Signals/`）——一个独立程序集，在安装了 [DVSignals](https://github.com/Fuggschen/dv-signals-test) mod 时与其集成。主项目由 `SignalsShim.cs` 在运行时通过反射加载它，因此核心 mod 没有针对它的编译期依赖。

---

## 环境搭建

### 前置条件

- 已安装 [UnityModManager](https://www.nexusmods.com/site/mods/21) 的 [Derail Valley](https://www.derailvalley.com/)
- [dotnet SDK](https://dotnet.microsoft.com/download) 10 版（推荐——可解决大部分构建问题）
  （用 `dotnet --version` 确认它在你的 PATH 中）

#### 可选，但推荐

- **[仅 Windows]** [Visual Studio Community](https://visualstudio.microsoft.com/vs/community/) 对开源使用免费。
  安装时选择 **.NET 桌面开发**工作负载——它包含 dotnet SDK 以及构建本 mod 所需的全部内容。
- **[全平台]** [Visual Studio Code](https://code.visualstudio.com/) 加 [C# Dev Kit 扩展](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit)，指向 `.slnx` 文件。

### 首次构建

1. 克隆仓库。
2. 参照下面的示例，在仓库根目录创建 `Directory.Build.props` 文件。这是个人文件，不应提交回仓库：
    ```xml
    <Project>
        <PropertyGroup>
            <!-- 替换为你机器上 Derail Valley 的安装路径 -->
            <DvInstallDir>C:\Program Files\Steam\steamapps\common\Derail Valley</DvInstallDir>
        </PropertyGroup>
    </Project>
    ```
3. 在仓库根目录（与 `RemoteDispatch.slnx` 同目录）的终端中运行：
    ```
    dotnet build -v detailed
    ```
4. 你应看到类似输出：
    ```
    Restore complete (0.3s)
        Determining projects to restore...
        All projects are up-to-date for restore.
      RemoteDispatch.Signals netstandard2.0 succeeded (0.1s) → RemoteDispatch.Signals\bin\Debug\netstandard2.0\RemoteDispatch.Signals.dll
      RemoteDispatch netstandard2.0 succeeded (0.8s) → RemoteDispatch\bin\Debug\netstandard2.0\RemoteDispatch.dll
        Deployed to: C:\Program Files\Steam\steamapps\common\Derail Valley\Mods\RemoteDispatch

    Build succeeded in 1.4s
    ```
5. 进入你的 RemoteDispatch mod 文件夹（`<DerailValley>/Mods/RemoteDispatch`），确认其中没有 `.cache` 文件——若有则删除，它属于旧版本 mod 的残留。
6. 启动游戏，确认 RemoteDispatch 在 UMM 中已启用。

#### Linux 用户

构建流程包含一个打包脚本，它在 Linux 上不会自动运行。你需要手动把 `.dll` 文件拷贝到 mods 文件夹。

确保你的 `Directory.Build.props` 指向 Derail Valley 安装目录中的 `Managed` 文件夹以便解析依赖——路径与上面的 Windows 示例不同。也可以安装 PowerShell Core 后手动运行打包脚本。如果你愿意提交相应改动、让 `RemoteDispatch.csproj` 在 Linux 上自动处理这件事，我们非常欢迎。

#### Release 构建

运行 `dotnet build -c Release -v detailed`，打包脚本会在 `dist` 文件夹中生成 zip 文件。

---

## 代码组织

```
RemoteDispatch/
├── Main.cs            # Mod 入口点，生命周期（enable/disable/reload）
├── Settings.cs        # 端口、密码、按用户的权限、游戏内 GUI
├── Engine/
│   ├── CarUpdater.cs       # 对车辆生成/销毁与操控变更的 Harmony 钩子
│   ├── JunctionPatches.cs  # 对 Junction.Switch 的 Harmony 钩子
│   ├── LocoControl.cs      # 向 RemoteControllerModule 发送油门/制动/换向命令
│   └── Updater.cs          # Unity 协程；供异步代码使用的 RunOnMainThread 桥
├── Data/
│   ├── CarData.cs          # 把车辆与机车状态快照为可 JSON 序列化的对象
│   ├── JobData.cs          # 任务与阶段数据；对任务状态变更的 Harmony 钩子
│   ├── PlayerData.cs       # 玩家位置与 Steam 昵称
│   └── RailTracks.cs       # 轨道几何与道岔位置，烘焙为经纬度
├── Server/
│   ├── AsyncSet.cs         # 线程安全集合，带异步 TakeAsync，用于挂起的标签
│   ├── HttpServer.cs       # HttpListener；路由请求、处理鉴权与 gzip
│   └── Session.cs          # 按客户端划分的会话；基于标签的变更通知
├── Shims/
│   └── SignalsShim.cs      # 与 DVSignals mod 的可选运行时集成
└── frontend/
    └── *                   # 前端源码（见下）

RemoteDispatch.Signals/     # 独立程序集，仅在安装了 DVSignals mod 时加载
├── Bootstrap.cs            # 由 SignalsShim 反射调用的公共入口点
└── SignalsBridge.cs        # 读写信号显示的桩实现
```

### 前端

前端是一个位于 `frontend/` 的 JavaScript 应用。它**在构建时直接打包进主 DLL**——没有独立的前端构建步骤，也没有开发服务器。你对前端文件的任何改动都会在下次运行 `dotnet build` 时自动生效。

---

## 更新系统的工作原理

理解这一点会让新增数据类型容易得多。

当游戏中发生变更时，相应的监视器调用 `Sessions.AddTag("some-tag")`。这会把该标签排入每个活动客户端会话的 `AsyncSet<string>`。客户端对 `GET /updates/<sessionId>` 发起长轮询——若有挂起的标签会立即收到；否则请求会挂起最多一分钟等待。

当服务器组装响应时，会调用 `GetUpdateForTag(tag)`，把每个挂起标签的当前状态序列化为一个 JSON 对象，以标签名为键。

新增一种数据类型：

1. 在 `Data/` 中添加一个返回 `JObject` 或 `JToken` 的序列化器。
2. 在 `Session.cs` 的 `GetUpdateForTag` 中添加一个 case。
3. 添加游戏侧的监视器，在数据变化时调用 `Sessions.AddTag("your-tag")`——可以是 `Engine/` 中的 Harmony 补丁、`CarUpdater.cs` 中的事件订阅，或 `Updater.cs` 中的协程。
4. 如果还希望客户端能按需拉取数据（更新流之外），在 `HttpServer.cs` 中添加一个 HTTP 端点。

---

## Harmony 补丁

所有补丁在 mod 启用时应用，在禁用或关闭时移除。

若新增补丁类，把它放在与所补丁对象最相关的文件中（车辆/机车补丁放 `CarUpdater.cs`，道岔补丁放 `JunctionPatches.cs`，任务补丁放 `JobData.cs`）。

---

## 可选的 Signals 集成

`SignalsShim.cs` 在 DVSignals mod 存在时，于运行时通过反射加载 `RemoteDispatch.Signals.dll`。主项目对 Signals 没有编译期依赖。如果你的改动涉及信号相关行为，它属于独立的 Signals 集成程序集，而不是核心项目。

---

## 调试与日志

- `Main.Log(...)` —— 始终写入 UMM 日志。
- `Main.DebugLog(...)` —— 仅当 UMM mod 设置中启用日志时才写入；适用于你希望用户_能够_按需使用、但不该在用户调试其他 mod 时刷屏的冗长或诊断输出。

你也可以用预处理指令 `#if DEBUG` 给日志"加闸"：闸内的代码只会被包含在以 DEBUG 配置构建的产物中。若给 `dotnet build` 命令不传 `-c Release`，这就是默认行为。

---

## 代码风格

仓库根目录有 `.editorconfig`——请让你的编辑器遵守它。需要留意的要点：

- **缩进：**制表符（tab），不是空格。这样每位贡献者都能把编辑器设成自己习惯的缩进宽度，也避免 PR 中出现大量仅空白差异的改动。
- **大括号风格：**C# 用 Allman（左大括号独占一行），JavaScript 用 K&R One True Brace Style（左大括号与语句同行）。多数编辑器默认如此并会自动应用。

遵循这些约定能让 PR diff 保持干净，让每个人的审阅都更容易。

---

## 测试

所有测试都是手工的——没有自动化测试（附带一句：如果你想添加自动化测试，请务必加）。提交 PR 前，请至少连接一个浏览器客户端，在游戏中测试你的改动。值得验证的点：

- 你的新功能/修复是否有效
- 更新循环（游戏内数据变化时能否送达客户端）是否仍然正常

---

## 提交变更

- **保持 PR 聚焦。**每个 PR 只做一个功能或修复，审阅会轻松得多。
- **多个小改动优于少数大改动。**如果你有一个大改动的想法，考虑把它拆成更小的、增量式的 PR。这更易于审阅与合并，也有助于避免合并冲突。如果想在功能完全打磨前就合并，随时可以先把它放在调试设置或功能开关后面。

---

## 功能开关

如果你要添加尚未完成、或希望能开关以便测试的功能，可以把它放在 mod 设置菜单中的某个设置后面。这既让其他贡献者在你工作期间测试并基于你的成果继续开发，也让有兴趣尝鲜的用户可以自行启用未完成功能。

这与常规功能设置不要混淆。
功能开关用于把仍在开发中的未完成功能挡在后面，而常规设置针对的是已完成、但用户可能想启用或禁用的功能。功能完成并打磨后，把功能开关转成常规设置，是完全可能且现实的。

---

## AI

用 AI 帮你写代码没问题，用 AI 解释代码没问题，用 AI 写全部代码也没问题。

用 AI 写出你没有读过、也不理解的代码，不可以。

经验法则：*只用 AI 帮你写出你本可以独立写出的代码。*也就是说，你可以用它来提速或学习，但请不要试图以 vibe-code 的方式即兴堆出整个功能。


