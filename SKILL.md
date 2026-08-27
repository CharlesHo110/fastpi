---
name: fastpi
description: Charles 的 pi 全局要求配置:语言与 git commit 规范;codegraph 优先的代码查询约束;pi 推荐安装列表(通用基础设施 + 扩展补齐 Claude Code 内置能力)。
metadata:
  author: Charles
  version: "0.4.0"
---

# fastpi - pi 全局要求配置

工作环境的 pi 全局要求(语言/git commit 规范 + 推荐安装列表)。本 skill 是这些约束与列表的**唯一载体**,pi 在本环境工作时应遵循以下规范。

## 语言要求

所有思考过程和回答必须使用中文。

## git commit 消息

git commit 消息必须使用前缀 `[reviewed by $username]`,`$username` 为电脑当前用户名(动态占位符,运行时以 `git config user.name` 或系统用户名替换)。

示例:

```
[reviewed by Charles] 新增 xxx 功能
[reviewed by nobody] 修复 yyy
```

## 代码查询约束(codegraph 优先)

查代码(架构、调用链、符号位置、bug 定位等)时,若 codegraph(MCP)可用,**优先使用 codegraph 的结果,不要自己 grep/read 逐个翻文件**:

- 首选 `codegraph_explore`(一次调用返回相关符号的完整源码,Read 等价,通常一次就够);
- 定位用 `codegraph_search`,影响面用 `codegraph_callers` / `codegraph_callees` / `codegraph_impact`;
- 只有 codegraph 未覆盖的具体细节,才回退 Read/Grep 确认。

此约束同样写入 AGENTS.md 等约束文件,与本 skill 保持一致。

## 推荐安装列表

分两类:

- **通用**:基础工作流能力(内网业务 plugin + 公网流程 skill + 用户级基础设施)。
- **pi 专用**:pi 核心哲学是 "primitives, not features",subagent/plan mode/web 搜索/MCP 均不内置,需靠扩展对标 Claude Code。

级别约定:**项目级**(每个项目装一次,装到当前项目)与**用户级**(每台机装一次,装到全局)分开。codegraph 与 rtk 不是 skill,需用户级单独安装。**pi 专用项一律用户级**(补的是 pi 自身能力,与项目无关,不随项目重复装)。

### 一、通用

| 名称 | 类型 | 级别 | 说明 |
|---|---|---|---|
| pipeline | 内网 plugin | 项目级 | Mermaid(.mmd) flowchart 流水线定义与执行 |
| superpowers | 公网 skill | 项目级 | obra/superpowers 流程 skill 集;实际按需装子 skill(如 brainstorming),`npx skills add obra/superpowers -a pi` |
| codegraph | MCP | 用户级 | 代码图谱,语义导航/重构/诊断(非 skill) |
| rtk | CLI 工具 | 用户级 | Token 优化命令包装器(60-90%;非 skill;当前 v0.35.0 装在 C:\Program Files\rtk\,hook 待 `rtk init -g` 激活) |

#### 通用安装命令

```bash
# 项目级:skill 安装(npx skills,agent 参数用 -a pi)
npx skills add pipeline -a pi
npx skills add superpowers -a pi
```

```bash
# 用户级:codegraph(MCP),先装 codegraph CLI,再注册 stdio MCP 服务器
# pi 无原生 MCP,需经 pi-mcp-adapter 挂载:在用户级 .mcp.json 加 stdio 服务器
# command=codegraph, args=["serve","--mcp"]

# 用户级:rtk(CLI 工具,Rust Token Killer)
# - macOS/Linux:brew install rtk-ai/tap/rtk 或 cargo install rtk
# - Windows:GitHub Releases 下载 rtk-x86_64-pc-windows-msvc.zip 解压,rtk.exe 加入 PATH(当前装在 C:\Program Files\rtk\)
# 装完用 `rtk init -g` 激活 hook(用户级,一次性;当前状态:CLI v0.35.0 已装但 hook 未激活,每条命令尾部提示 "No hook installed - run `rtk init -g`")
```

### 二、pi 专用(扩展补齐 Claude Code 内置能力)

pi 很多 Claude Code 内置能力需靠**扩展(npm 包,`pi install` 安装,非 skill)**补齐。下表是对标清单,**全部用户级安装**(`pi install -g` / `npx skills add -g`,每台机一次):

| Claude Code 内置能力 | pi 补齐方案 | 类型 | 安装(用户级) |
|---|---|---|---|
| Task/subagent(并行/链式子代理) | pi-subagents(180K 下载/月;另可选 pi-sub-agent 带 9 个内置代理) | 扩展 | `pi install -g npm:pi-subagents` |
| WebSearch / WebFetch | pi-web-access(web_search/fetch_content,GitHub URL 自动本地克隆,YouTube/PDF 提取) | 扩展 | `pi install -g npm:pi-web-access` |
| MCP servers | pi-mcp-adapter(单代理工具省上下文,可直接读 .mcp.json,从 Claude Code/Cursor 导入) | 扩展 | `pi install -g npm:pi-mcp-adapter` |
| codegraph(代码图谱 MCP) | codegraph CLI + MCP(语义导航/重构/诊断;pi 无原生 MCP,需经 pi-mcp-adapter 挂载,配置同通用列表的 stdio 服务器) | MCP | 先装 codegraph CLI,再在用户级 .mcp.json 加 stdio 服务器 command=codegraph, args=["serve","--mcp"] |
| prompt/KV 缓存命中率优化 | pi-cache-optimizer(重排系统提示提升缓存命中、prompt_cache_key 回退、缓存统计页脚、`/cache-optimizer fix` 自动修复;Pi 0.82+) | 扩展 | `pi install -g npm:pi-cache-optimizer`(装后 `/reload`) |
| Plan mode | badlogic/pi-mono 官方示例扩展 `plan-mode`(examples/extensions/,MIT,需自行拷贝) | 扩展(示例) | 拷贝到用户级扩展目录(~/.pi/agent/extensions/) |
| Todo/任务跟踪 | pi-subagents 自带;否则用计划文件/TODO.md(superpowers 约定) | — | — |
| 代码流程能力(TDD/调试/评审/计划) | superpowers skill(见通用列表,pi 已官方适配) | skill | `npx skills add superpowers -a pi -g` |
| 文档编辑(docx/xlsx/pptx/pdf) | anthropics/skills 官方文档四件套(注意:source-available 参考快照,非开源许可) | skill | `npx skills add anthropics/skills@docx -a pi -g`(xlsx/pptx/pdf 同理),见下方说明 |
| 目标/任务跟踪(goal 契约) | pi-goal(create_goal 活动目标契约 + pi-goal-writer skill) | 扩展 | `pi install -g npm:pi-goal` |
| 行锚精确编辑 | pi-hashline-edit(read 返回 LINE#HASH 锚点,edit 按锚点改;pi 0.82+ 内置 hashline-edit 工具的基础) | 扩展 | `pi install -g npm:pi-hashline-edit` |
| 上下文管理/知识库 | context-mode(ctx_index/ctx_search/ctx_stats,BM25 知识库 + 自动捕获决策/错误/计划;大输出索引化省上下文) | 扩展 | `pi install -g npm:context-mode` |
| 交互式表单 | pi-interview(interview 工具,多维度决策/需求采集,优于来回聊天) | 扩展 | `pi install -g npm:pi-interview` |
| Plan mode(包实现) | @plannotator/pi-extension(plan mode 的 npm 包实现,与官方示例 plan-mode 目录可二选一或并存) | 扩展 | `pi install -g npm:@plannotator/pi-extension` |
| rtk 优化(pi 侧) | pi-rtk-optimizer(配合 rtk CLI,pi 会话级 token 优化器) | 扩展 | `pi install -g npm:pi-rtk-optimizer` |

文档编辑 skill 说明:Anthropic 官方仓库 `github.com/anthropics/skills` 的 `docx` / `xlsx` / `pptx` / `pdf` 四个 skill 是 Claude 文档能力的生产级参考实现(创建/编辑/分析,保留格式、公式、修订),pi 侧一律 `-g` 装到用户级。注意其许可为 **source-available 参考快照**(非 Apache 2.0),商用前需确认;skills.sh 上另有社区替代品可回退。
