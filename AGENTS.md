# AGENTS.md

## 语言要求 / Language Requirements

本项目（dv-remote-dispatch-chinese）要求以简体中文翻译呈现：
- 项目文档（README.md、CHANGELOG.md、docs/ 下说明）使用简体中文。
- 代码标识符（类名、方法名、变量名、命名空间）保持英文，符合 C# 惯例；代码注释可用中文。
- 新增代码中的用户可见字符串默认使用简体中文；确需保留英文专有名词（如游戏术语、API 名称）时保留原文。
- 提交信息（commit message）与 issue 标题/描述建议使用中文。

## 仓库布局 / Repository Layout

本项目是一个针对 Derail Valley 的 C# UnityModManager 模组，提供远程调度（remote dispatch）能力。代码结构如下：
- `RemoteDispatch/` - 主模组源代码
- `RemoteDispatch.Signals/` - Signals 集成模块
- `RemoteDispatch.Tests/` - 单元测试
- `RemoteDispatch.Signals.Tests/` - Signals 测试
- `build/` - 构建输出目录
- `dist/` - 发行打包目录
- `frontend/` - 内嵌的 Web 静态资源（HTML、JS、CSS）

## 文件说明 / File Descriptions

### 主模组文件
- `Main.cs` - 入口点与模组生命周期管理
- `Settings.cs` - 配置与权限处理
- `HttpServer.cs` - HTTP 端点实现与请求处理器
- `CarData.cs` - 车辆/机车数据处理
- `PlayerData.cs` - 玩家光点（blip）数据处理
- `RailTracks.cs` - 轨道与道岔数据处理

### Signals 模块
- `Bootstrap.cs` - Signals 集成初始化
- `SignalsBridge.cs` - 与 Signals API 的通信桥接
- `LoggingReturn.cs` - Signals 模块的日志回调

### 测试文件
- `UnitTest1.cs` - 基础单元测试占位
- `Bootstrap.cs` - Signals 测试（当前结构中缺失）

## 代码流程 / Code Flow

模组经由 Main.Load() 初始化，随后通过 Harmony 对游戏系统打补丁。启用时启动 HTTP 服务器与数据更新器。数据通过以下 HTTP 端点提供：
- `/car` - 车辆/机车数据
- `/player` - 玩家光点数据
- `/track` - 轨道道岔数据
- `/signals` - 信号数据（启用时）
- `/updates` - 实时数据更新

## 构建/检查/测试命令 / Build/Lint/Test Commands

```bash
# 构建解决方案
dotnet build RemoteDispatch.slnx

# 运行测试
dotnet test RemoteDispatch.Tests/RemoteDispatch.Tests.csproj

# 执行单个测试
dotnet test RemoteDispatch.Tests/RemoteDispatch.Tests.csproj --filter "Test1"

# 打包发布
.\package.ps1 -Configuration Release

# 开发部署
.\package.ps1 -Configuration Debug -DVPath "C:\path\to\derailvalley"
```

## 代码风格指南 / Code Style Guidelines

### 导入 / Imports
- using 语句按分组排列：先标准库，再第三方，最后项目内
- 使用完整命名空间以保持清晰（除非明确需要，否则不使用 'using static'）
- 各分组内按字母顺序排列

### 格式化 / Formatting
- 使用 4 空格缩进（不用制表符）
- 遵循 C# 命名约定（方法与属性用 PascalCase，参数用 camelCase）
- 控制语句的左大括号与语句置于同一行
- 逻辑段落之间使用空行分隔

### 类型 / Types
- 尽可能使用 readonly 字段而非常量
- 局部变量使用 var 并带显式类型
- 当清晰性优先时，使用显式 null 检查而非 null 条件运算符
- 使用可空引用类型（项目中已启用）

### 命名约定 / Naming Conventions
- 类：PascalCase
- 方法：PascalCase
- 变量：camelCase
- 常量：PascalCase
- 私有字段：_camelCase

### 错误处理 / Error Handling
- 对可能失败的操作使用 try/catch 块
- 记录错误时附带描述性信息，适当时包含堆栈跟踪
- 优先使用具体的异常处理而非笼统的 catch-all
- 优雅处理 null 值，使用显式检查

### 文档 / Documentation
- 公共方法使用 XML 注释记录
- 未完成功能使用 TODO 注释
- 行内注释保持简短且聚焦

### 语法安全清单 / Syntax Safety Checklist
- **绝不要**在多行跨度上使用 replaceAll 而不做精确边界匹配
- 编辑后始终确认没有产生重复代码块
- 替换文本时，包含足够的上下文使其唯一
- 每次编辑后，读取受影响的文件以确认语法完整性

## AGENTS.md 内容规范 / AGENTS.md Content Policy

- 本文件只存放面向 agent 的工作指引：语言要求、仓库布局、文件说明、构建与测试命令、代码风格、协作流程。
- 不在 AGENTS.md 中写用户故事（User Stories）、验收标准清单这类需求叙述内容；需求与验收细节应留在 issue 或专门的规格文档里。
- 其他 AGENTS.md（子目录或兄弟上下文文件）同样不要写入类似内容，若发现请将内容迁出到上述位置。

## Agent skills

### Issue tracker

Issue（工作项）通过 `gh` CLI 跟踪在 GitHub Issues 中。详见 `docs/agents/issue-tracker.md`。

### Triage labels

使用默认的五个规范 triage 标签（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`）。详见 `docs/agents/triage-labels.md`。

### Domain docs

单上下文：仓库根目录的 `CONTEXT.md` + `docs/adr/`。详见 `docs/agents/domain.md`。

