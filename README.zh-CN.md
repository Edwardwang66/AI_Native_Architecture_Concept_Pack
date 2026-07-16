# AI-Native Architecture Concept Pack

[English](./README.md)

这是一个面向公开评审、在许可方案获批后按开源方式协作的研究与设计企划包，目标是帮助团队：

1. 判断哪些业务流程真正适合 Agent；
2. 以证据、边界、权限、失败与评测为基础完成 AI-native 架构设计；
3. 把架构决策转化为可移植的能力契约和策略，而不是再造一个通用 MCP 网关。

## 当前状态

仓库目前包含**已经批准的设计、可公开评审的实施计划**与**尚未验证的 Skill 草案**，不包含生产实现，也没有任何宿主已经取得行为验证资格。

| 轨道 | 已批准方向 | 当前状态 |
|---|---|---|
| Part I — AI Native Architect Skill | 遵循 Agent Skills 格式的可移植核心＋独立宿主适配层；OpenClaw 为首个行为验证目标 | 设计已批准；实施计划已发布；构建和验证尚未开始 |
| Part II — Capability Control Plane | Architecture-to-Policy 编译器＋独立保障层；MVP 只读且不进入运行时数据路径 | 设计已批准；实施计划已发布；构建和市场验证尚未开始 |

## 规范性设计

英文 RFC 是规范性来源：

1. [RFC 0001 — Portable AI Native Architect Skill](./docs/rfcs/0001-portable-ai-native-architect-skill.md)
2. [RFC 0002 — Architecture-Driven Capability Control Plane](./docs/rfcs/0002-architecture-driven-capability-control-plane.md)

中文快速摘要：

- **Part I：** Skill 核心必须在只聊天、只读、没有 shell、MCP、子代理或文件写入时仍然可用。新用户应在五分钟内获得第一份可评审的 Quick Architecture Brief；完整模式通过阶段状态、证据账本、人工门禁和 Architecture Decision Pack 深入展开。宿主兼容必须逐版本实测，不能宣传“到处都能用”。
- **Part II：** 长期方向不是完整 Registry＋Gateway，而是把架构意图编译成能力契约、最小工具包、策略、审批、等价性证明、评测和配置 diff。MVP 读取现有 registry、schema、配置和 trace，在运行路径之外完成审计、模拟和漂移检测，不托管凭证、不代理调用、不自动修改生产策略。

## 可公开评审的实施计划

1. [Part I — AI Native Architect Skill 实施计划](./docs/superpowers/plans/2026-07-16-ai-native-architect-skill.md)
2. [Part II — Capability Control Plane 实施计划](./docs/superpowers/plans/2026-07-16-capability-control-plane-mvp.md)

两份计划把获批 RFC 转化为测试优先的实施任务和明确的上线门禁。计划公开不代表实现或行为声明已经通过验证。

## 研究背景与现有草案

- [原始中文产品企划](./AI_Native_Architecture_and_MCP_Product_Strategy_zh.md) — 保留最初的产品、市场、投放、风险和验证假设；其中 Part I/II 的最终边界以 RFC 为准。
- [当前 Skill 草案](./ai-native-architect/SKILL.md) — RFC 之前的设计草稿，保留用于比较，尚未通过正式契约和宿主测试。

## 建议评审顺序

1. 阅读本页，确认项目边界和当前状态。
2. 评审 RFC 0001 的用户流程、可移植性、证据、产物、安全和宿主验证契约。
3. 评审 RFC 0002 的产品边界、能力契约、策略编译、漂移、安全与 MVP。
4. 评审实施计划的需求追溯、测试覆盖、发布门禁和双轨交接契约。
5. 将两份 RFC 与原始中文企划、现有 Skill 草案对照。
6. 通过公开 issue 提交缺失证据、反例、安全边界、兼容性或实现风险。

## 声明边界

- “遵循 Agent Skills”只描述格式目标，不代表输出质量已经验证。
- “可移植核心”描述的是获批架构；只有至少两个独立宿主通过契约验证后，才可宣称已经证明跨宿主可移植性。
- “兼容”与“已验证”必须绑定宿主、版本、环境、安装方式、bundle digest 和测试集。
- Registry、Marketplace 或扫描器通过不能证明行为安全。
- Part II MVP 不执行运行时授权，也不接触凭证。
- OpenClaw/ClawHub 的信任验证与本项目的行为验证必须分开报告。

## 当前发布阻断

- Skill 尚未通过获批的契约和宿主评测。
- 当前没有任何宿主可以标记为 `Native/Verified`。
- 当前 Skill 声明 Apache-2.0，而现行 ClawHub 发布文档要求发布 Skill 使用 MIT-0；权利人批准兼容许可方案前，停止 ClawHub 发布，只进行本地或 Git 安装验证。
- 实施计划已经公开，但尚未开始执行；发布仍受计划中列明的契约、证据、许可、治理和设计伙伴门禁约束。
