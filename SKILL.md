---
name: fastpi
description: Charles 的 pi 全局要求配置:语言与 git commit 规范;codegraph 优先的代码查询约束;pi 推荐安装列表(通用基础设施 + 扩展补齐 Claude Code 内置能力)。
metadata:
  author: Charles
  version: "0.7.0"
---

# fastpi - pi 全局要求配置

工作环境的 pi 全局要求(语言/git commit 规范 + 推荐安装列表)。本 skill 是这些约束与列表的**唯一载体**,pi 在本环境工作时应遵循以下规范。

## 语言要求

所有思考过程和回答必须使用中文。

## git commit 消息

commit 消息用中文,简明扼要说明本次改动的目的或内容。**是否加 `[reviewed by <user>]` 前缀,取决于仓库远程地址**:

- **仅当** `git remote -v` 的 origin 包含 `gitlab.bj.tkoffice` 时,commit message 才以 `[reviewed by <user>]` 开头;`<user>` 为电脑当前用户名(`whoami`,如 `hecan`)。
- **其他仓库**(`codeup.aliyun.com`、`github.com`、`gitee` 等)**不加**此前缀,直接用中文描述即可。

示例:

```
[reviewed by hecan] 修复登录态过期问题     # 仅 origin 含 gitlab.bj.tkoffice 时
修复登录态过期问题                         # github / codeup / gitee 等一律不加
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
| rtk | CLI 工具 | 用户级 | Token 优化命令包装器(60-90%;非 skill;当前 v0.35.0 装在 C:\Program Files\rtk\;pi 侧由 pi-rtk-optimizer 扩展自动改写命令加 rtk 前缀,rtk 自身 hook 无需激活) |

#### 通用安装命令

```bash
# 项目级:skill 安装(npx skills,agent 参数用 -a pi)
npx skills add pipeline -a pi
npx skills add superpowers -a pi
```

```bash
# 用户级:codegraph(MCP),先装 codegraph CLI,再注册 stdio MCP 服务器
# pi 无原生 MCP,需经 pi-mcp-adapter 挂载:在用户级 .mcp.json 加 stdio 服务器
# command=codegraph, args=["serve","--mcp"], lifecycle="eager"(启动即自动连接,2026-09-14 起本机启用)

# 用户级:rtk(CLI 工具,Rust Token Killer)
# - macOS/Linux:brew install rtk-ai/tap/rtk 或 cargo install rtk
# - Windows:GitHub Releases 下载 rtk-x86_64-pc-windows-msvc.zip 解压,rtk.exe 加入 PATH(当前装在 C:\Program Files\rtk\)
# `rtk init -g` 仅 Claude Code 等宿主需要;pi 由 pi-rtk-optimizer 扩展改写命令加 rtk 前缀,rtk hook 本身无需激活
```

### 二、pi 专用(扩展补齐 Claude Code 内置能力)

pi 很多 Claude Code 内置能力需靠**扩展(npm 包,`pi install` 安装,非 skill)**补齐。下表是对标清单,**全部用户级安装**(`pi install -g` / `npx skills add -g`,每台机一次):

| Claude Code 内置能力 | pi 补齐方案 | 类型 | 安装(用户级) |
|---|---|---|---|
| Task/subagent(并行/链式子代理) | pi-subagents(180K 下载/月;另可选 pi-sub-agent 带 9 个内置代理) | 扩展 | `pi install -g npm:pi-subagents` |
| WebSearch / WebFetch | pi-web-access(web_search/fetch_content,GitHub URL 自动本地克隆,YouTube/PDF 提取) | 扩展 | `pi install -g npm:pi-web-access` |
| MCP servers | pi-mcp-adapter(单代理工具省上下文,可直接读 .mcp.json,从 Claude Code/Cursor 导入) | 扩展 | `pi install -g npm:pi-mcp-adapter` |
| codegraph(代码图谱 MCP) | codegraph CLI + MCP(语义导航/重构/诊断;pi 无原生 MCP,需经 pi-mcp-adapter 挂载,配置同通用列表的 stdio 服务器) | MCP | 先装 codegraph CLI,再在用户级 .mcp.json 加 stdio 服务器 command=codegraph, args=["serve","--mcp"], lifecycle="eager"(启动自动连接) |
| prompt/KV 缓存命中率优化 | pi-cache-optimizer(重排系统提示提升缓存命中、prompt_cache_key 回退、缓存统计页脚、`/cache-optimizer fix` 自动修复;Pi 0.82+) | 扩展 | `pi install -g npm:pi-cache-optimizer`(装后 `/reload`) |
| Plan mode | badlogic/pi-mono 官方示例扩展 `plan-mode`(examples/extensions/,MIT,需自行拷贝;本机已拷贝但 index.ts.disabled,实际改用下方 npm 包) | 扩展(示例) | 拷贝到用户级扩展目录(~/.pi/agent/extensions/) |
| Todo/任务跟踪 | pi-subagents 自带;否则用计划文件/TODO.md(superpowers 约定) | — | — |
| 代码流程能力(TDD/调试/评审/计划) | superpowers skill(见通用列表,pi 已官方适配) | skill | `npx skills add superpowers -a pi -g` |
| 文档编辑(docx/xlsx/pptx/pdf) | anthropics/skills 官方文档四件套(注意:source-available 参考快照,非开源许可) | skill | `npx skills add anthropics/skills@docx -a pi -g`(xlsx/pptx/pdf 同理),见下方说明 |
| 目标/任务跟踪(goal 契约) | pi-goal(create_goal 活动目标契约 + pi-goal-writer skill) | 扩展 | `pi install -g npm:pi-goal` |
| 行锚精确编辑 | pi-hashline-edit(read 返回 LINE#HASH 锚点,edit 按锚点改;pi 0.82+ 内置 hashline-edit 工具的基础) | 扩展 | `pi install -g npm:pi-hashline-edit` |
| 上下文管理/知识库 | context-mode(ctx_index/ctx_search/ctx_stats,BM25 知识库 + 自动捕获决策/错误/计划;大输出索引化省上下文) | 扩展 | `pi install -g npm:context-mode` |
| 交互式表单 | pi-interview(interview 工具,多维度决策/需求采集,优于来回聊天) | 扩展 | `pi install -g npm:pi-interview` |
| Plan mode(包实现) | @narumitw/pi-plan-mode(当前实际安装,Codex 式只读 /plan 模式) | 扩展 | `pi install -g npm:@narumitw/pi-plan-mode` |
| rtk 优化(pi 侧) | pi-rtk-optimizer(配合 rtk CLI,pi 会话级 token 优化器) | 扩展 | `pi install -g npm:pi-rtk-optimizer` |
| Chrome 控制 | pi-chrome(桥接已登录 Chrome:tab 管理/page.evaluate/截图,POST 127.0.0.1:17318;抓登录态页面/微信公众号等) | 扩展 | `pi install -g npm:pi-chrome` |
| 跨会话记忆 | pi-memory(MEMORY.md/daily log/scratchpad 纯 markdown + qmd 语义搜索;KV cache-stable 注入,零依赖;不做自动事实提取) | 扩展 | `pi install -g npm:pi-memory` |
| 精确上下文投喂 | @narumitw/pi-file-context(Ctrl+Shift+X 选行范围/changed hunks/git 溯源挂载到下一 prompt) | 扩展 | `pi install -g npm:@narumitw/pi-file-context` |
| 权限门禁(执行前风险裁决) | pi-verdict(三态 allow/ask/deny:内置 deny floor(bash 危险正则 + 路径敏感度 S0-S5)→ 你的 allow/deny 规则 → 模型分类器;fail-closed;零依赖单文件 ~1k 行;不可配置关闭的自保护层;替代已从 npm 下架的 pi-safety-gate) | 扩展 | `pi install -g npm:pi-verdict` |

codegraph 的 MCP 服务器注册文件示例见本仓库 `config/mcp.json`(stdio 服务器,含 lifecycle="eager" 启动自动连接);复制或合并到用户级 `~/.pi/agent/mcp.json` 即可。其余个人配置(settings/models/AGENTS/extensions)为机器本地内容,不入库。

文档编辑 skill 说明:Anthropic 官方仓库 `github.com/anthropics/skills` 的 `docx` / `xlsx` / `pptx` / `pdf` 四个 skill 是 Claude 文档能力的生产级参考实现(创建/编辑/分析,保留格式、公式、修订),pi 侧一律 `-g` 装到用户级。注意其许可为 **source-available 参考快照**(非 Apache 2.0),商用前需确认;skills.sh 上另有社区替代品可回退。

## 当前实际安装快照(2026-09-20 更新)

本机(hecan)pi 0.85.1 实际启用 **14 个 npm 扩展**,均为用户级安装(`pi install npm:<pkg>`,记录于 `~/.pi/agent/settings.json` 的 `packages`,实装于 `~/.pi/agent/npm/node_modules/`):

| 扩展 | 版本 | 用途 |
|---|---|---|
| pi-mcp-adapter | 2.34.0 | MCP 挂载 |
| pi-web-access | 0.30.0 | 联网搜索/抓取 |
| pi-subagents | 0.70.0 | 子代理编排 |
| pi-cache-optimizer | 2.8.10 | KV 缓存命中率优化 |
| pi-goal | 0.1.7 | 目标契约 |
| pi-hashline-edit | 0.8.3 | 行锚精确编辑 |
| context-mode | 1.0.169 | 上下文管理/知识库 |
| pi-interview | 0.12.0 | 交互式表单 |
| pi-rtk-optimizer | 0.9.0 | rtk 命令改写(pi 侧) |
| pi-chrome | 0.15.51 | 已登录 Chrome 桥接 |
| @narumitw/pi-plan-mode | 0.58.1 | 只读 plan 模式 |
| pi-memory | 0.4.2 | 跨会话记忆 |
| @narumitw/pi-file-context | 0.54.2 | 精确上下文投喂 |
| pi-verdict | 0.9.1 | 执行前权限门禁(三态裁决) |

一键复现(用户级,全部装到 npm 扩展目录):

```bash
for p in pi-mcp-adapter pi-web-access pi-subagents pi-cache-optimizer pi-goal \
         pi-hashline-edit context-mode pi-interview pi-rtk-optimizer pi-chrome \
         @narumitw/pi-plan-mode pi-memory @narumitw/pi-file-context pi-verdict; do
  pi install npm:$p
done
```

配套用户级配置(机器本地内容,不入库):

- `~/.pi/agent/mcp.json` —— codegraph stdio 服务器,与本仓库 `config/mcp.json` 一致(`lifecycle="eager"`)。
- `~/.pi/agent/config/pi-verdict.json` —— pi-verdict 的用户规则:分类器模型 + `allow`/`deny`/`denyPaths`;本机 `classifierModel` 指向轻量模型 `volcengine-coding-plan/glm-5.3-flash`(默认自省 = 会话主模型亲自裁决,每条灰区命令都烧主力额度且更慢)。

备注(选型与前车之鉴):

- `@gotgenes/pi-permission-system` 仍在 npm 依赖中但**未启用**:纯确定性 allow/ask/deny(无 LLM 判定、无延迟),可作 pi-verdict 的规则型替代。
- `~/.pi/agent/extensions/` 下另有 pi 官方示例扩展 `confirm-destructive` / `dirty-repo-guard` / `git-checkpoint` / `protected-paths`(均为 session 层,不碰 bash 命令),与本扩展清单不冲突。
- **选包前置校验**:扩展一律先在**实际使用的 registry**(本机 = `repo.huaweicloud.com`)用 `npm view <pkg> version` 验证存在,再 `pi install`。包从 registry 下架会让 `pi update --extensions` **整批** ETARGET 失败(一个包拖垮全量更新);registry 上消失的包必须同时从 `~/.pi/agent/npm/package.json` 依赖里摘除。
- 替代方案:`pi-safety-gate` 下架后曾用「本地路径包」过渡(`pi install /abs/path`,磁盘副本留在 `~/.pi/agent/local-packages/`),但本地路径包不受 registry 更新、需手工维护,最终改用同为 npm 包的 pi-verdict。
