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

## 用户故事 / User Stories

> 本节原为 issue #1 规格说明中的 `## User Stories` 章节，现迁移至本文件作为需求与验收上下文；issue 正文保留问题陈述、方案与实现/测试决策。

1. 作为一名中文玩家，我想让调度网页显示中文，这样我不必在游戏和网页之间翻译铁路术语。
2. 作为一名中文玩家，我想让网页上的道岔名与游戏里用通讯电台指着道岔时看到的名称一致，这样我能在网页上指的道岔一定能在游戏里找到。
3. 作为一名中文玩家，我想让「全列制动」「独立制动」「换向器」「油门」这些词与游戏内既有译名一致，这样我能直接对应到驾驶室里的控件。
4. 作为一名中文玩家，我想让信号弹窗里的「通行」「预告减速」「减速慢行」「停车」一眼可辨，这样我不会把减速信号误当成通行信号。
5. 作为一名中文玩家，我想让搜索框、表格表头、空态提示等零散文案也是中文，这样界面不会中英混杂。
6. 作为一名中文玩家，我想让语言选项存在于网页设置面板里，这样我不必去动模组的配置文件。
7. 作为一名中文玩家，我想让网页默认就是中文，这样我打开就能用，不需要先找设置。
8. 作为一名调度员，我想能在网页上随时切回英文，这样当我要拿某个界面词去搜上游文档或提 issue 时，能对上原文。
9. 作为一名英文玩家或用英文排障的开发者，我想让语言切换不影响功能，这样切到英文后所有控件仍然正常工作。
10. 作为一名排障者，我想让浏览器控制台的日志消息保持英文，这样我拿报错信息去搜上游 issue 时能命中。
11. 作为一名排障者，我想让 UMM 日志保持英文原文，这样报错文本可直接与上游对照。
12. 作为一名排障者，我想让 UMM 模组设置面板保持英文，这样当 IMGUI 中文字体渲染不可靠时不会引入新的显示问题。
13. 作为一名开发者，我想让中文译文集中放在一个独立的词典文件里而不是散落在代码中，这样上游改文案时冲突最小。
14. 作为一名开发者，我想让 HTML 里保留英文原文，这样即便词典漏了某个 key，界面也会退回显示英文而不是留下空白。
15. 作为一名开发者，我想让词典以 key 索引，这样能靠一条命令扫出漏译的条目。
16. 作为一名开发者，我想让中文版有独立的版本号，这样 UMM 里不会把中文版与上游版本混为一谈。
17. 作为一名开发者，我想让中文版不参与上游的自动更新链路，这样已装中文版的玩家不会被自动更新意外换回英文版。
18. 作为一名开发者，我想让仓库里有一份术语表，这样后续新增界面文案时译法不会漂移。
19. 作为一名开发者，我想让本次「用运行时词典而非就地替换」的取舍留下书面记录，这样日后有人问「为什么不直接改字符串」时有据可查。
20. 作为一名维护者，我想让 README 也是中文，这样中文使用者能自己看懂安装与排障说明。
21. 作为一名维护者，我想让 CHANGELOG 的历史条目保持英文原样、只让中文版的新改动记中文，这样与上游的变更记录仍能逐条对照。
22. 作为一名维护者，我想让 UMM 设置面板里的 15 条英文标签作为已知待办记录下来，这样以后字体验证通过时可以补做，而不是彻底遗忘。
23. 作为一名维护者，我想让几个模组自造的兜底字符串（玩家名兜底、信号错误伪 ID、默认权限档案名）作为已知待办记录下来，这样它们不会被误当成「游戏数据」而永久漏掉。

## Agent skills

### Issue tracker

Issue（工作项）通过 `gh` CLI 跟踪在 GitHub Issues 中。详见 `docs/agents/issue-tracker.md`。

### Triage labels

使用默认的五个规范 triage 标签（`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`）。详见 `docs/agents/triage-labels.md`。

### Domain docs

单上下文：仓库根目录的 `CONTEXT.md` + `docs/adr/`。详见 `docs/agents/domain.md`。

