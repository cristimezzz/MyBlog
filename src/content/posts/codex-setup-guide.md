---
title: Codex 设置方式梳理
published: 2026-06-08
description: 从 config.toml、AGENTS.md、MCP、权限、Hooks 到 Skills 和 Plugins，整理 Codex 的常用设置层级
tags: [Codex, AI, Tools]
category: 'AI'
draft: false
---

## 零、为什么要认真设置 Codex

Codex 并不是一个“打开就只会聊天”的编辑器插件。更准确地说，它像一个可以读仓库、改代码、跑命令、调用外部工具的协作代理。既然它能做的事情比普通补全工具多得多，那么设置方式也不应该只停留在“换个模型”这一层。

我认为 Codex 的设置可以先分成六类：

* `config.toml`：控制 Codex 自身行为，例如模型、权限、沙箱、MCP、hooks 等。
* `AGENTS.md`：告诉 Codex 这个仓库应该怎么工作，例如包管理器、测试命令、代码风格。
* MCP：把外部工具或数据源接进 Codex，例如文档搜索、浏览器、Figma、Sentry。
* Permissions / Sandbox：限制 Codex 能读写哪些文件、能否访问网络。
* Hooks：在工具调用、会话开始、任务结束等节点执行脚本。
* Skills / Plugins：把可复用工作流沉淀成能力，必要时再打包分发。

官方文档中，Codex 的配置入口主要集中在 [Config basics](https://developers.openai.com/codex/config-basic)、[Config reference](https://developers.openai.com/codex/config-reference)、[AGENTS.md](https://developers.openai.com/codex/guides/agents-md)、[MCP](https://developers.openai.com/codex/mcp)、[Permissions](https://developers.openai.com/codex/permissions)、[Hooks](https://developers.openai.com/codex/hooks)、[Skills](https://developers.openai.com/codex/skills) 和 [Plugins](https://developers.openai.com/codex/plugins) 这几处。

## 一、先分清“全局”和“项目”

Codex 的配置不是只读一个文件。官方文档说明，个人默认配置放在：

```text
~/.codex/config.toml
```

而某个项目自己的配置可以放在仓库内：

```text
.codex/config.toml
```

这两个文件的职责不同。

全局配置适合放“我个人到哪里都一样”的偏好，例如默认模型、默认权限配置、常用 MCP 服务器、个人通知命令。项目配置则适合放“这个仓库才需要”的设置，例如当前项目允许 Codex 写哪些目录、是否打开某个项目专用 hook、是否设置项目级 rules。

需要注意的是，项目级 `.codex/config.toml` 只有在你信任该项目时才会被加载。这个设计很合理：仓库里的配置毕竟可以影响 Codex 的执行环境，如果随便从陌生项目读取，等于把钥匙挂在门外。

官方给出的配置优先级可以简单理解为：

```text
命令行参数 > 项目配置 > profile 配置 > 用户配置 > 系统配置 > 内置默认值
```

因此，如果某个设置“不知道为什么没生效”，第一件事不是改十遍文件，而是先判断它是不是被更高优先级覆盖了。

## 二、最小可用的 config.toml

下面是一份偏保守的用户级配置示例，适合日常在自己仓库里使用：

```toml
# ~/.codex/config.toml
model = "gpt-5.5"
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = false
```

这里有三个核心点。

`model` 指定默认模型。官方配置参考里把它描述为 Codex 要使用的模型名称，例如 `gpt-5.5`。模型名称会更新，所以我更建议把它写在全局配置里，方便以后统一调整。

`approval_policy = "on-request"` 表示 Codex 可以在需要更高权限时向你请求确认。日常交互中，这比 `never` 更适合新手，因为你能看到哪些操作越过了普通沙箱。

`sandbox_mode = "workspace-write"` 表示 Codex 可以在工作区内读写，但不会直接拥有全盘权限。配合 `network_access = false`，可以避免普通命令随意访问网络。

如果你只想让 Codex 看代码、不改文件，可以换成：

```toml
sandbox_mode = "read-only"
```

如果你非常清楚自己在做什么，也可以临时提高权限，但我不建议把 `danger-full-access` 当成长期默认值。代理越强，边界越重要。

## 三、用 AGENTS.md 写“项目说明书”

`config.toml` 管 Codex 怎么运行，`AGENTS.md` 管 Codex 怎么理解项目。

一个博客仓库可以这样写：

```markdown
# AGENTS.md

## Commands

- Use pnpm only.
- Run `pnpm check` after changing Astro components.
- Run `pnpm build` before publishing content changes.

## Content

- Blog posts live in `src/content/posts/`.
- Keep frontmatter fields valid for the Astro content schema.
- Do not edit unrelated posts unless the user asks.
```

官方文档中，Codex 会在开始工作前读取 `AGENTS.md`。它还支持全局和项目级层叠：全局文件可以放在 Codex home 目录，项目根目录和子目录也可以各自放一份。越靠近当前目录的说明越晚出现，因此更适合写更具体的覆盖规则。

我的经验是，`AGENTS.md` 最适合写三类内容：

* 固定命令：安装、开发、测试、构建、格式化。
* 固定约定：目录含义、命名方式、提交前检查。
* 固定禁区：不要改生成物、不要碰密钥、不要替用户回滚未确认改动。

不要把所有个人偏好都塞进仓库的 `AGENTS.md`。例如“回复我时用中文”更像个人偏好，适合放全局；“这个仓库必须用 pnpm”才是项目约定。

## 四、用 profile 区分不同工作模式

如果你经常在“只读审查”和“可写开发”之间切换，可以用 profile 文件：

```text
~/.codex/review.config.toml
~/.codex/dev.config.toml
```

例如：

```toml
# ~/.codex/review.config.toml
sandbox_mode = "read-only"
approval_policy = "on-request"
```

```toml
# ~/.codex/dev.config.toml
sandbox_mode = "workspace-write"
approval_policy = "on-request"

[sandbox_workspace_write]
network_access = false
```

运行时通过参数选择：

```bash
codex --profile review
codex --profile dev
```

这样做的好处是，不需要每次临时修改同一个配置文件。审查 PR 时保持只读，真正实现功能时再切到开发模式，心里会清楚很多。

## 五、权限配置：让 Codex 有边界地工作

旧式配置里常见的是 `sandbox_mode` 和 `sandbox_workspace_write`。新的权限配置还可以用 `default_permissions` 和 `[permissions.<name>]` 定义更细的文件系统与网络策略。

例如，允许 Codex 写当前工作区，但拒绝读取 `.env`：

```toml
default_permissions = "project-edit"

[permissions.project-edit]
extends = ":workspace"

[permissions.project-edit.filesystem.":workspace_roots"]
"**/*.env" = "deny"

[permissions.project-edit.network]
enabled = false
```

如果确实需要访问某些网络域名，也可以用 allowlist：

```toml
[permissions.project-edit.network]
enabled = true

[permissions.project-edit.network.domains]
"api.openai.com" = "allow"
"*.github.com" = "allow"
```

这里的原则很朴素：能只读就不要可写，能只给工作区就不要给全盘，能只放行一个域名就不要放行整个互联网。

## 六、接入 MCP：给 Codex 新工具

MCP（Model Context Protocol）可以让 Codex 调用外部工具。官方 MCP 文档给出的配置形态大致如下：

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
env_vars = ["LOCAL_TOKEN"]
```

远程 HTTP MCP 也可以这样配置：

```toml
[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
bearer_token_env_var = "FIGMA_OAUTH_TOKEN"
```

我建议先从“只解决一个明确痛点”的 MCP 开始。例如：

* 写前端时接浏览器或 Playwright MCP，用来截图和点页面。
* 写设计稿还原时接 Figma MCP，用来读取设计上下文。
* 查官方文档时接 OpenAI Docs MCP 或 Context7，减少凭记忆写过时 API 的概率。

不要一口气装十个 MCP。工具越多，上下文越嘈杂，权限面也越大。

## 七、Hooks：把流程约束写成脚本

Hooks 适合处理“每次都应该自动发生”的事情。例如在 `PreToolUse` 里拦截危险命令，在 `PostToolUse` 里记录命令输出，在 `SessionStart` 里加载团队状态。

官方 hooks 配置按三层组织：

```text
事件 -> matcher 组 -> hook handlers
```

内联到 `config.toml` 时可以写成：

```toml
[[hooks.PreToolUse]]
matcher = "^Bash$"

[[hooks.PreToolUse.hooks]]
type = "command"
command = 'python3 "$(git rev-parse --show-toplevel)/.codex/hooks/pre_tool_use_policy.py"'
timeout = 30
statusMessage = "Checking Bash command"
```

Hooks 很强，但也容易把系统变复杂。我的建议是：只有当一条规则必须机械执行时，才写 hook。比如“禁止删除生产数据库备份”适合 hook；“写代码尽量优雅”不适合 hook，放在 `AGENTS.md` 里就够了。

## 八、Skills 和 Plugins：沉淀可复用能力

Skill 是一套可复用工作流。官方文档里，skill 通常是一个目录，里面有 `SKILL.md`，还可以带 `scripts/`、`references/`、`assets/` 等资源。

一个 skill 的结构可以很简单：

```text
my-skill/
  SKILL.md
  scripts/
  references/
  assets/
```

`SKILL.md` 里至少要有 `name` 和 `description`。Codex 会先看到技能名称、描述和路径；只有当任务匹配这个技能时，才读取完整说明。这种“渐进披露”能节省上下文。

Plugin 则更像分发单元。一个 plugin 可以包含 skills、apps、MCP servers。也就是说：

```text
Skill  = 可复用工作流
Plugin = 可安装、可分享的一组能力
```

如果只是自己复用一个流程，先写 skill 就够了。如果要给团队安装、启用、禁用、共享，再考虑做成 plugin。

## 九、我的推荐设置顺序

第一次配置 Codex 时，不要从最复杂的 hooks 和插件开始。可以按这个顺序来：

1. 先写全局 `~/.codex/config.toml`，确认模型、审批策略、沙箱。
2. 给常用仓库加 `AGENTS.md`，写清楚安装、测试、构建命令。
3. 根据任务需要加一两个 MCP，不要贪多。
4. 如果权限需求变复杂，再从 `sandbox_mode` 迁移到 permissions profile。
5. 只有当流程需要自动强制执行时，再写 hooks。
6. 重复出现三次以上的工作流，沉淀成 skill。
7. 需要共享给别人时，再打包成 plugin。

这样配置出来的 Codex 不会像“另一个陌生开发者”一样乱跑，而更像一个知道项目规矩、懂得请示权限、能调用合适工具的协作者。

## 十、结语

Codex 设置的核心不是“把所有能力都打开”，而是把能力放到正确的层级里。

个人偏好放全局配置，项目约定放 `AGENTS.md`，外部工具走 MCP，安全边界交给 permissions，机械约束再写 hooks，可复用流程沉淀为 skills，需要分发时再做 plugins。

这个顺序一旦理清，Codex 就不再只是一个会写代码的模型，而是一个可以被认真管理的工程工具。
