# service-to-cli

**把现有系统改造成 Agent Native 接口的方法论和工具。**

基于《从 GUI 到 CLI：现有软件的 Agent 化改造思路》— 探讨如何把现有软件（GUI、CLI、REST API）改造成 AI agent 可直接调用的 Skill + CLI 形态。

## 核心结论

**Skill + CLI 是改造现有系统的最优解。**

| 方案 | 本地安装 | 分发成熟度 | 云端成本 | 调用延迟 |
|------|---------|-----------|---------|---------|
| 纯 Skill | ✅ | ⚠️ 早期 | 低 | 快 |
| **Skill + CLI** | ✅ | ✅ pip/npm/brew | **低** | **快** |
| MCP | ⚠️ 常驻 server | ✅ | 高 | 中 |

## 核心五原则

1. **幂等优先** — 重复执行不破坏状态，给 agent 试错的勇气
2. **输出结构化** — stdout JSON，stderr 人类可读，exit code 精确
3. **粒度重设计** — 一个 CLI 完成一个业务目标，而非一个技术操作
4. **参数自解释** — `--user-id` 好过 `--uid`
5. **文档即 prompt** — `--help` 的内容就是给 agent 的使用说明

## 目录结构

```
service-to-cli/
├── SKILL.md     # OpenClaw 技能文件
└── README.md    # 整体说明
```

## 使用方法

当需要把某个现有系统改造成 agent 可用的工具时，运行此 skill，它会：

1. 分析现有系统，梳理业务原子
2. 按五大原则设计 CLI 粒度
3. 生成 Skill.md + CLI 代码骨架
4. 输出完整交付物

## 相关链接

- [Agent-Native 概念](https://vercel.com/blog/agents)
- [MCP 协议](https://modelcontextprotocol.io/)
- [OpenClaw](https://github.com/openclaw/openclaw)
- [mcporter](https://github.com/nicepkg/mcporter)

## License

MIT
