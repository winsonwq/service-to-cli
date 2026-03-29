---
name: service-to-cli
description: |
  把现有系统（GUI、CLI、REST API 等）改造为 Agent 可用的 Skill + CLI 接口。
  当用户要求"把 xxx 系统改造成 agent 可用的工具"、"封装现有系统为 CLI"、"设计 agent 化接口"时触发。
  输入现有系统的文档、API 规范或业务逻辑描述，输出 Skill.md + CLI 代码骨架 + 技术栈建议。
---

# service-to-cli

把现有系统改造成 Agent Native 接口的核心方法论。

## 核心结论

**Skill + CLI 是改造现有系统的最优解。**

| 方案 | 本地安装 | 分发成熟度 | 云端成本 | 调用延迟 | 适合场景 |
|------|---------|-----------|---------|---------|---------|
| 纯 Skill | ✅ | ⚠️ 早期 | 低 | 快 | 知识整理、SOP |
| **Skill + CLI** | ✅ | ✅ pip/npm/brew | **低** | **快** | **现有系统改造** |
| MCP | ⚠️ 常驻 server | ✅ 发展中 | 高 | 中 | 开放平台 |
| Skill + mcporter + MCP | ✅ | ✅ OpenClaw 特供 | 中 | 中 | 大型工具集管理 |

**MCP 的核心问题**：工具数量膨胀后 agent 陷入"选择困难"，且 server 常驻有运维开销。

## 两个层次

改造有两层，关注点不同：

1. **分发层**：改造后的工具怎么交付给 agent？（Skill 说明书 + CLI 执行体）
2. **接口层**：工具的输入输出、参数设计要怎么变？

## 适用条件

- 现有系统有相对清晰的 API 或操作逻辑
- 系统有一定的业务语义（而非纯工具函数集合）
- 如果系统过于底层（全是通用 CRUD），可能不适合 agent 化改造

## 改造流程

### 第一步：分析现有系统，梳理业务原子

从文档中提取核心业务动作，按"谁在什么场景下做什么"来组织。

**关键问题**：
- 这个系统的用户最终想完成什么目标？（而非系统有哪些按钮）
- 哪些动作经常一起出现，可以合并成一个任务？
- 哪些动作是独立的，可以单独调用？

**输出**：业务动作表

| 业务目标 | 涉及操作 | 设计后的 CLI |
|---------|---------|-------------|
| 新销售入职 | 创建账号+建档+发工牌+入群 | `onboard-sale` |
| 学员完成报名 | 创建学员+录入学时+发放证书 | `enroll-student` |

### 第二步：设计 CLI 粒度，遵循五大原则

**核心原则（必须遵守）**：

1. **幂等优先**：重复执行不破坏状态。给了 agent 试错的勇气，是最重要的一条。
2. **输出结构化**：stdout JSON，stderr 人类可读，exit code 精确。
3. **粒度重设计**：一个 CLI 完成一个业务目标，而非一个技术操作。
4. **参数自解释**：`--user-id` 好过 `--uid`，`--expires-at` 好过 `--exp`。
5. **文档即 prompt**：`--help` 的内容就是给 agent 的使用说明，要包含场景、参数说明、示例、常见错误。

**输出示例**（直接复制修改使用）：

```bash
$ onboard-sale --help

用法：onboard-sale --name <用户名> [--expires-at <过期时间>]

新销售入职任务。幂等：重复执行返回已有数据。

参数：
  --name        用户名，自解释参数，必填
  --expires-at  过期时间，ISO 格式，默认永不过期

示例：
  onboard-sale --name 张三
  onboard-sale --name 张三 --expires-at 2026-12-31

常见错误：
  - name 已存在：返回已有数据，不创建新记录
  - name 格式错误：返回错误信息，exit code 2
```

**exit code 约定**：
- `0` — 成功
- `1` — 一般错误（参数错误等）
- `2` — 业务逻辑错误（资源不存在等）

**输出 JSON schema**：
```json
// 成功
{"status": "ok", "data": {"id": "...", "name": "..."}}

// 失败（exit code 非 0）
{"status": "error", "message": "具体错误描述"}
```

**幂等处理**：检查是否已存在，存在则返回已有数据，不报错

### 第三步：生成 Skill.md

每个 CLI 对应一个 `Skill.md`，文件路径为 `skills/<cli-name>-skill/SKILL.md`。

**内容结构**：

- **触发场景**：描述哪些用户语言会触发这个 skill，用列表列举典型说法
- **调用方式**：写清楚 CLI 命令和参数的格式（不加 code fence，因为这是给 agent 读的说明）
- **参数说明**：列表形式，说明每个参数的含义、格式要求、是否必填、默认值
- **输出**：说明成功和失败时的返回格式。成功返回 JSON（`status: ok`），失败 exit code 非 0
- **注意事项**：重点说明幂等逻辑、边界情况、agent 需要注意的坑

### 第四步：输出完整交付物

- `skills/<name>-skill/SKILL.md` — 每个业务动作一个
- `cli/<name>/` — 每个 CLI 的代码骨架（bash / Python / Node.js 任选）
- `README.md` — 整体说明
- 技术栈建议（用哪个 CLI 框架：click / typer / oclif 等）
- 分发方式建议（pip / npm / brew）

## CLI 部署场景

| 场景 | 说明 |
|------|------|
| **本地** | 私人 agent 的工具箱，CLI 装到系统 PATH |
| **服务端** | 批量 agent 共享能力层，不同实例复用同一套工具 |

**对 CLI 的一致要求**：可无头调用（不需要 TTY）、输出可解析（JSON）、幂等安全。

## 接口设计三变化

| 变化 | 传统方式 | Agent 化方式 |
|------|---------|-------------|
| **粒度** | 功能视角（"创建用户"按钮） | 任务视角（`onboard-new-sale` 完成整套入职） |
| **参数** | 缩写、可读性差 | 自解释，默认值安全 |
| **输出** | 人类可读文本 | 结构化 JSON + 精确 exit code |

## 参考资料

- [Agent-Native 概念](https://vercel.com/blog/agents)
- [MCP 协议](https://modelcontextprotocol.io/)
- [OpenClaw](https://github.com/openclaw/openclaw)
- [mcporter](https://github.com/steipete/mcporter)

CLI 框架选择建议：Python 用 Click，Node.js 用 oclif，Go 用 Cobra。
