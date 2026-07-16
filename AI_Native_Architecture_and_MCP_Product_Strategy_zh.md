# AI-Native Architecture Skill 与 MCP Control Plane 产品构想

> **设计状态（2026-07-16）：** 本文保留为原始研究企划与产品背景。Part I 与 Part II 的获批规范性设计分别见 [RFC 0001 — Portable AI Native Architect Skill](./docs/rfcs/0001-portable-ai-native-architect-skill.md) 和 [RFC 0002 — Architecture-Driven Capability Control Plane](./docs/rfcs/0002-architecture-driven-capability-control-plane.md)。如果本文与 RFC 的产品边界冲突，以 RFC 为准；尤其是 Part II 已从“统一网关优先”收窄为架构驱动的能力契约、策略编译与独立保障层，MVP 不进入运行时数据路径。

**版本：** v0.1  
**日期：** 2026-07-13  
**工作名称：** AI Native Architect / MCP Control Plane  
**作者：** Edward Wang（概念草案）

---

## 0. 执行摘要

很多公司知道自己“应该变得 AI-native”，但真正困难的不是调用一个模型，而是回答下面这些问题：

- 哪些业务流程适合由 AI 参与，哪些仍应保持确定性？
- 应该做 Copilot、RAG、Agent、Workflow，还是 Multi-Agent？
- AI 可以做哪些判断，哪些动作必须由规则、权限系统或人类控制？
- Agent 如何访问公司的数据、软件和服务？
- 如何设计状态、记忆、评估、观测、失败恢复和上线流程？
- 如何避免每个团队重复搭建连接器、权限、日志和评测系统？

本构想将产品拆成三个相互促进的层次：

1. **AI Native Architect Skill**  
   在代码库或项目环境中，通过结构化访谈、证据收集、方案比较和决策记录，把模糊的 AI 想法整理成可评审的系统架构。

2. **MCP Control Plane**  
   为企业统一管理 Agent 可以发现、调用和组合的工具，覆盖私有 Registry、身份与权限、策略、动态工具暴露、健康状态、观测和评估。

3. **AI Architecture Handbook / Whitepaper Website**  
   提供厂商中立的架构模式、成熟度模型、案例、模板和交互式评估，承担教育、品牌与获客作用。

推荐顺序不是立即建设一个庞大的“MCP 版 OpenRouter”，而是：

> **白皮书建立认知 → 开源 Skill 验证需求 → 垂直案例证明价值 → MCP Control Plane 产品化**

核心定位可以概括为：

> **From an ambiguous AI idea to a verifiable, governable AI-native architecture.**

---

## 1. 为什么这个问题值得做

### 1.1 企业真正缺少的是架构决策能力

多数团队并不缺少模型 API 或 Demo，而是缺少一套可重复的方法，把业务目标转换为：

- 明确的工作流边界；
- 可验证的 Agent 职责；
- 确定性服务与概率性推理之间的分工；
- 工具与数据访问模型；
- 权限和人工审批机制；
- 评估指标与上线标准；
- 可持续维护的技术路线。

当前常见问题包括：

- 从技术名词出发，而不是从业务过程出发；
- 把能够用普通软件解决的问题强行做成 Agent；
- 过早采用 Multi-Agent；
- 把 MCP 当成完整架构，而不是连接协议；
- 先做 Tool Calling，后补权限、审计和评估；
- 架构存在于少数人的脑中，没有形成可复用的决策记录；
- Demo 可以运行，但无法解释何时失败、为什么失败、如何恢复。

### 1.2 最有价值的产品不是“替企业自动决定”

架构设计包含大量组织约束和风险判断。产品不应声称自动输出唯一正确答案，而应：

- 把隐含假设显式化；
- 区分已确认事实、合理推断和未知信息；
- 提出 2–3 个可比较方案；
- 解释每个取舍；
- 推荐满足目标所需的**最低自主性**；
- 形成可审查、可修改、可追踪的决策包。

因此，它更像一位严谨的 AI 系统架构师，而不是自动生成 Mermaid 图的聊天机器人。

---

## 2. 三种产品路径

## 路径 A：Skill-first（推荐）

先发布一个可移植的 `AI Native Architect` Agent Skill，并配套模板、架构模式库和案例。

### 优点

- 开发成本最低；
- 可以快速进入真实项目环境；
- 能直接读取代码、文档和现有架构；
- 容易开源和传播；
- 可以观察用户反复遇到的真实问题；
- 为后续 SaaS 和 MCP 平台积累需求、模板与评测数据。

### 缺点

- 初期收入有限；
- 输出质量依赖项目上下文；
- 单纯的 Markdown Skill 容易被复制；
- 必须通过工作流、模板、案例和数据建立壁垒。

### 适合验证的问题

- 用户是否愿意按照结构化流程完成架构发现？
- 哪些架构决策最容易反复出现？
- 用户最重视哪些输出物？
- 用户是否愿意将 Skill 引入真实代码库和内部文档？
- 哪些环节需要团队协作、版本管理和权限控制？

---

## 路径 B：MCP Control Plane-first

直接建设一个“MCP 领域的 OpenRouter”，统一发现、认证、筛选、调用和观测 MCP Server。

### 优点

- 产品价值容易与企业基础设施预算关联；
- 有机会成为 Agent 工具层的长期控制平面；
- 可以形成使用数据、信任评分和连接器生态；
- 与企业私有工具、权限和治理高度绑定。

### 缺点

- 范围非常大；
- 安全责任高；
- 工具不像模型 endpoint 那样天然可互换；
- OAuth、密钥、租户隔离和审计都属于高复杂度基础设施；
- 官方 Registry 已经覆盖基础元数据，单纯目录价值有限；
- 过早建设可能无法确定客户真正需要“路由”的是什么。

### 结论

不建议作为第一步，但可以作为 Skill 验证后的核心商业产品。

---

## 路径 C：Vertical-first

先做实验室、机器人或运营场景中的具体 Agent 产品：

- Laboratory Workflow Preflight Agent；
- ROS Incident Copilot；
- Agent Trace-to-Eval；
- Scientific Operations Agent。

### 优点

- 用户和问题非常具体；
- 更容易衡量价值；
- 可以积累真实工作流、失败模式和评测数据；
- 有助于形成强案例。

### 缺点

- 市场较窄；
- 容易被理解为单一行业工具；
- 不一定自然扩展到企业级架构平台。

### 结论

适合作为参考实现和早期 design partner 项目，而不是唯一定位。

---

## 3. 推荐的统一产品路线

三个方向并不冲突，可以形成一个完整飞轮：

```text
AI Architecture Handbook
        ↓ 教育与获客
AI Native Architect Skill
        ↓ 生成架构、能力与治理需求
Vertical Reference Implementations
        ↓ 验证真实效果并积累失败数据
MCP Control Plane
        ↓ 执行连接、权限、策略、观测与评估
Architecture Pattern & Evaluation Data
        └──────────────→ 反哺 Skill 和白皮书
```

产品之间的关系：

- **白皮书/网站**告诉团队“应该如何思考”；
- **Skill**帮助团队“针对自己的项目做出决策”；
- **Control Plane**帮助团队“将决策安全地运行起来”；
- **垂直产品**证明这套方法在真实环境中有效。

---

# Part I — AI Native Architect Skill

## 4. 产品定位

### 4.1 一句话

> A project-aware architecture skill that turns an ambiguous AI initiative into a reviewable, evidence-backed AI-native system design.

### 4.2 核心用户

首个 ICP 建议聚焦：

- 50–500 人的软件、运营或研发密集型公司；
- 已经尝试过 LLM Demo，但没有专门 AI Platform 团队；
- 存在多个 SaaS、内部 API、人工审批和跨团队工作流；
- CTO、Head of Engineering、Staff Engineer、Product Lead 或 AI Transformation Lead 需要共同做决策。

次级用户：

- AI 咨询公司；
- 企业创新团队；
- Agent 创业公司；
- 希望为客户提供架构评审的独立工程师。

### 4.3 用户得到的不是一张图

最终交付应该是一个 **Architecture Decision Pack**：

```text
ai-architecture/
├── 00-executive-brief.md
├── 01-current-state.md
├── 02-workflow-map.md
├── 03-ai-suitability-assessment.md
├── 04-architecture-options.md
├── 05-target-architecture.md
├── 06-agent-contracts.md
├── 07-mcp-capability-map.md
├── 08-trust-permissions-and-data.md
├── 09-failure-modes.md
├── 10-evaluation-plan.md
├── 11-rollout-roadmap.md
└── adr/
    ├── 001-autonomy-level.md
    ├── 002-state-and-memory.md
    └── 003-tool-access-model.md
```

---

## 5. Skill 的核心原则

### 5.1 从业务过程出发，而不是从技术名词出发

首先理解：

- 谁在完成工作？
- 目标是什么？
- 输入和输出是什么？
- 哪些步骤是确定性的？
- 哪些步骤需要判断？
- 哪些错误最昂贵？
- 哪些动作可撤销？
- 谁拥有最终责任？

在此之前，不讨论 LangGraph、MCP、RAG 或 Multi-Agent。

### 5.2 推荐最低足够自主性

默认选择顺序：

1. 普通软件或确定性自动化；
2. AI 辅助 Copilot；
3. 有边界的 Workflow Agent；
4. 事件驱动 Agent；
5. Multi-Agent；
6. 高自主、长期运行系统。

只有前一级无法满足目标时，才升级自主性。

### 5.3 将推理与执行控制分离

建议架构通常分为：

```text
Business Intent
      ↓
AI Reasoning / Planning
      ↓
Structured Plan or Proposed Action
      ↓
Policy + Deterministic Validation
      ↓
Human Approval when required
      ↓
Execution Services / MCP Tools
      ↓
State, Trace, Evaluation and Feedback
```

### 5.4 所有建议都必须有证据状态

Skill 应给每项关键结论加标签：

- **Confirmed**：由代码、文档或用户明确确认；
- **Inferred**：根据现有证据推断；
- **Unknown**：信息不足，需要验证；
- **Decision**：团队已经做出的选择；
- **Risk**：可能导致失败或返工的假设。

### 5.5 多方案比较，而不是单方案包装

至少提出两种，通常三种：

- **Conservative**：低自主、低风险、快速上线；
- **Balanced**：有限自主、明确边界、可逐步扩展；
- **Ambitious**：更高自主性、更强潜力、更高治理成本。

---

## 6. 七阶段工作流

## Stage 0 — Context Intake

目标：建立项目事实基础。

可读取：

- 代码库；
- README 和架构文档；
- API schema；
- 产品需求；
- 运行手册；
- 已有数据模型；
- 现有工作流；
- 用户提供的组织约束。

输出：

- 已知事实；
- 缺失上下文；
- 初步利益相关者；
- 不应访问的敏感范围。

硬性要求：

- 默认只读；
- 不读取秘密或生产凭证；
- 不在上下文不足时假装理解公司流程。

---

## Stage 1 — Business Outcome

回答：

- 需要改善的业务结果是什么？
- 当前成本、时间、错误率或体验问题是什么？
- 成功标准是什么？
- 哪些事情明确不在范围内？
- 谁承担最终决策责任？

输出：

- 问题陈述；
- 非目标；
- 成功指标；
- 约束；
- 负责人。

---

## Stage 2 — Workflow Decomposition

将工作流拆成：

- Actors；
- Inputs；
- States；
- Decisions；
- Actions；
- Systems of Record；
- Exceptions；
- Approvals；
- Outputs。

输出：

- 当前流程图；
- 手工交接点；
- 信息断点；
- 高摩擦步骤；
- 高风险步骤。

---

## Stage 3 — AI Suitability Assessment

对每个步骤评估：

| 维度 | 判断问题 |
|---|---|
| 模糊性 | 是否需要自然语言、视觉或不完整信息判断？ |
| 确定性 | 是否可以用普通规则或代码可靠解决？ |
| 可验证性 | 是否有客观方式判断结果正确？ |
| 可撤销性 | 错误动作能否轻易恢复？ |
| 风险 | 错误是否涉及资金、隐私、安全或合规？ |
| 频率 | 是否高频到值得自动化？ |
| 数据条件 | 是否有足够上下文和权限？ |
| 反馈 | 是否能获得执行结果并形成学习闭环？ |

简单决策规则：

```text
稳定、确定、规则清晰
→ 普通软件或 Workflow Automation

需要模糊判断，但不直接执行高风险动作
→ Copilot

需要多步判断，动作受限且可验证
→ Bounded Workflow Agent

需要长期状态、事件响应和恢复
→ Event-driven Agentic Workflow

角色之间确有独立职责、上下文和并行价值
→ 考虑 Multi-Agent
```

---

## Stage 4 — Architecture Options

每个方案应包含：

- 系统边界；
- Agent 数量和职责；
- 状态与记忆；
- 数据来源；
- 工具能力；
- MCP 是否必要；
- 权限和审批；
- 失败恢复；
- 评估方式；
- 预计复杂度；
- 主要风险。

评分维度：

| 维度 | 建议权重 |
|---|---:|
| 业务价值 | 20% |
| 可靠性 | 20% |
| 可验证性 | 15% |
| 安全与治理 | 15% |
| 实施复杂度 | 10% |
| 维护成本 | 10% |
| 可扩展性 | 5% |
| 上线速度 | 5% |

评分不是替团队自动决策，而是暴露取舍。

---

## Stage 5 — Target Architecture

目标架构至少覆盖七层：

1. **Experience Layer**  
   用户界面、聊天、嵌入式操作入口或后台任务。

2. **Orchestration Layer**  
   工作流、状态机、事件、任务队列和重试。

3. **Reasoning Layer**  
   模型、Planner、Classifier、Retriever 或专用 Agent。

4. **Capability Layer**  
   APIs、MCP Servers、内部服务和确定性工具。

5. **Identity & Policy Layer**  
   用户身份、服务身份、RBAC/ABAC、审批和数据边界。

6. **State & Data Layer**  
   System of Record、短期状态、长期记忆、缓存和 provenance。

7. **Evaluation & Operations Layer**  
   Trace、日志、指标、评测、回归测试、成本和事件处理。

---

## Stage 6 — Trust, Failure and Evaluation

必须回答：

- 哪些动作是 read-only？
- 哪些动作会改变外部状态？
- 哪些动作需要审批？
- 失败后如何重试、补偿或回滚？
- 工具返回内容是否可信？
- 模型看到哪些数据？
- 如何检测 Prompt Injection 或 Tool Poisoning？
- 谁可以查看 Trace？
- 哪些数据可以用于后续评测？
- 什么条件下系统必须停止而不是继续尝试？

评测分层：

```text
Component Evals
- 分类、抽取、检索、工具参数

Workflow Evals
- 多步完成率、状态转换、恢复能力

Policy Evals
- 权限、审批、数据边界、危险动作拦截

Business Evals
- 时间、成本、质量、转化、人工负担

Production Monitoring
- 延迟、错误率、工具健康、成本、漂移
```

---

## Stage 7 — Rollout Plan

建议采用：

```text
Offline replay
→ Shadow mode
→ Human-in-the-loop pilot
→ Limited production
→ Broader rollout
```

每个阶段必须定义：

- 进入条件；
- 退出条件；
- 失败阈值；
- 回滚方案；
- 负责人；
- 需要收集的数据。

---

## 7. Skill 的推荐目录结构

```text
ai-native-architect/
├── SKILL.md
├── references/
│   ├── architecture-patterns.md
│   ├── ai-suitability-rubric.md
│   ├── autonomy-levels.md
│   ├── mcp-decision-guide.md
│   ├── state-and-memory.md
│   ├── trust-and-permissions.md
│   ├── evaluation-framework.md
│   └── rollout-patterns.md
├── assets/
│   ├── executive-brief-template.md
│   ├── workflow-map-template.md
│   ├── option-comparison-template.md
│   ├── adr-template.md
│   └── architecture-pack-template/
└── scripts/
    ├── initialize_pack.py
    ├── validate_pack.py
    └── render_diagrams.py
```

第一版不必实现脚本；先验证纯 Skill + 模板是否足够。

---

## 8. Skill 的差异化

官方已经存在帮助开发者构建 MCP Server 的 Skill。因此，本 Skill 不应重复：

- 选择 MCP transport；
- 生成 Server scaffold；
- 包装某一个 API；
- 教用户写基础 Tool schema。

它应处于更高一层：

> **先判断公司是否需要 Agent、需要什么程度的自主性、MCP 在哪里有用，以及整个系统如何被验证和治理。**

当目标架构确定后，才可以将具体 MCP Server 交给专门构建 Skill。

---

# Part II — MCP Control Plane

## 9. “MCP 里的 OpenRouter”应该如何定义

这个类比有启发性，但不能照搬。

模型 Provider 通常暴露相近的推理接口，可以按价格、延迟、吞吐、地域和可用性路由。MCP Tool 则可能：

- 具有完全不同的语义；
- 使用不同用户账户；
- 访问不同数据；
- 产生不可逆副作用；
- 依赖状态和会话；
- 需要不同审批；
- 返回不可信内容。

因此，更准确的产品定位是：

> **A discovery, policy, routing and observability control plane for enterprise agent capabilities.**

或者：

> **Cloudflare + Okta + OpenRouter for agent tools.**

---

## 10. 五类路由

### 10.1 Discovery Routing

根据任务和上下文，只向 Agent 暴露必要的 Server 和 Tool。

价值：

- 减少 Tool schema 占用；
- 降低错误选择；
- 减少攻击面；
- 支持大型企业工具目录。

### 10.2 Policy Routing

根据：

- 用户身份；
- 团队；
- 数据区域；
- 风险等级；
- 时间；
- 环境；
- 审批状态；

决定哪些工具可以被发现或调用。

### 10.3 Endpoint Routing

当同一 MCP 服务存在多个兼容 endpoint 时，根据：

- 健康状态；
- 延迟；
- 区域；
- 租户；
- 成本；
- 版本；

选择 endpoint。

### 10.4 Compatibility Routing

只有在能力契约真正等价时，才允许跨 Server fallback。

例如：

- 多个只读 Web Search Provider；
- 同一企业服务的多区域部署；
- 同一连接器的稳定版与备用版。

对于发送邮件、转账、删除数据等 write action，不应默认自动换 Server。

### 10.5 Workflow Routing

根据任务构建最小 Tool Bundle 或调用链，但必须保留：

- 显式状态；
- 可解释选择；
- 权限检查；
- 执行前验证；
- 高风险动作审批。

---

## 11. Control Plane 核心组件

```text
Agent / MCP Host
        ↓
Unified MCP Gateway
        ↓
Identity & Tenant Resolution
        ↓
Policy Engine
        ↓
Capability Discovery / Semantic Router
        ↓
Approval & Validation
        ↓
Execution Proxy
        ↓
MCP Servers / Internal APIs / SaaS
        ↓
Trace, Metrics, Evaluation and Audit
```

### 11.1 Registry Ingestion

数据来源：

- 官方公共 MCP Registry；
- 企业私有 Registry；
- 内部 connector catalog；
- 手工审核的远程 Server。

存储：

- Server metadata；
- Tool schemas；
- capabilities；
- owner；
- version；
- status；
- trust metadata；
- policy tags。

### 11.2 Capability Normalization

建立企业内部能力分类，而不是只依赖 Tool 名称：

```text
communication.email.read
communication.email.send
crm.contact.read
crm.contact.update
engineering.issue.create
data.warehouse.query
finance.payment.initiate
```

为每项能力补充：

- read/write/destructive；
- reversible/irreversible；
- data sensitivity；
- approval level；
- idempotency；
- expected latency；
- owner；
- compatible alternatives。

### 11.3 Identity and Auth Broker

需要处理：

- 用户身份；
- Agent/service identity；
- OAuth delegation；
- token vault；
- tenant isolation；
- scoped credentials；
- revocation；
- short-lived access。

原则：

- Agent 不直接持有长期秘密；
- 每次调用可追溯到用户和工作流；
- 权限最小化；
- 写操作与读操作分离。

### 11.4 Policy Engine

策略示例：

```text
Sales agents may read CRM contacts.
Only account owners may update a deal.
Any external email to more than 10 recipients requires approval.
Production database tools are read-only for AI agents.
Scientific instruments require deterministic validation before execution.
```

### 11.5 Trust and Security Layer

包括：

- namespace/source verification；
- package和依赖扫描；
- Tool schema change detection；
- prompt/tool poisoning checks；
- sandbox；
- outbound network policy；
- response sanitization；
- risk classification；
- trust score；
- revocation。

### 11.6 Execution Proxy

职责：

- 参数 schema 校验；
- policy enforcement；
- approval；
- timeout；
- retry；
- idempotency key；
- circuit breaker；
- rate limit；
- result normalization；
- error taxonomy；
- compensation hook。

### 11.7 Observability and Evaluation

记录：

- 谁请求了什么能力；
- 为什么选择这个 Tool；
- 哪条策略允许或阻止；
- 参数和响应摘要；
- 延迟、成本和错误；
- 人工审批；
- 工作流最终结果；
- 是否应成为回归案例。

---

## 12. Control Plane MVP

第一版应严格收窄。

### 支持

- 一个组织；
- 一个私有 Tool Catalog；
- 3–5 个远程 MCP Server；
- 统一 Gateway endpoint；
- read-only capability 为主；
- 静态 policy；
- health check；
- invocation trace；
- 人工审批后的少量 write action；
- 从官方 Registry 同步 metadata；
- 自动生成 Agent 可见的最小 Tool Bundle。

### 暂不支持

- 大型公共 Marketplace；
- 自动跨供应商替换高风险 Tool；
- 复杂计费分成；
- 任意本地 MCP Server 托管；
- 自动修改企业权限；
- 完全自主的多工具长流程；
- “安装后自动理解整个公司”。

### MVP 的核心验证

> 企业是否愿意把多个 Agent 的 Tool discovery、权限和调用记录统一放到一个控制平面中？

---

# Part III — Whitepaper / Website

## 13. 网站的作用

网站不应只是产品 Landing Page，而应成为：

> **Vendor-neutral AI-native architecture knowledge base.**

目标：

- 建立可信度；
- 教育市场；
- 为 Skill 提供 references；
- 收集真实架构问题；
- 形成搜索流量；
- 将读者转化为 Skill 用户、design partner 或 Control Plane 客户。

---

## 14. 推荐内容结构

```text
Home
├── AI-Native Architecture Manifesto
├── Interactive Architecture Assessment
├── Pattern Library
│   ├── Copilot
│   ├── RAG
│   ├── Bounded Workflow Agent
│   ├── Event-Driven Agent
│   ├── Multi-Agent
│   └── Physical / Scientific Agent
├── MCP Architecture Guide
│   ├── When to Use MCP
│   ├── Capability Taxonomy
│   ├── Auth and Permissions
│   ├── Routing and Discovery
│   └── Security and Trust
├── Evaluation and Reliability
├── Case Studies
├── Templates
├── Skill
└── Control Plane
```

---

## 15. 白皮书建议标题

候选标题：

1. **The AI-Native Architecture Playbook**
2. **From Copilots to Governed Agents**
3. **Designing Verifiable Agentic Workflows**
4. **The Enterprise MCP Control Plane**
5. **How to Turn a Company Workflow into an AI-Native System**

推荐主白皮书：

> **The AI-Native Architecture Playbook: From Business Workflow to Verifiable Agent System**

配套技术白皮书：

> **The Enterprise MCP Control Plane: Discovery, Identity, Policy, Routing and Observability for Agent Tools**

---

## 16. 交互式 Architecture Assessment

网站可以提供一个 10–15 分钟评估：

输入：

- 业务流程；
- 当前系统；
- 风险；
- 数据；
-动作类型；
- 人工审批；
- 频率；
- 可验证性。

输出：

- 推荐自主性等级；
- 推荐架构模式；
- MCP 是否必要；
- 关键风险；
- 需要补充的信息；
- Architecture Pack 预览。

免费版本只生成摘要；完整版本交给 Skill 在项目环境内继续完成。

---

# Part IV — 垂直参考实现

## 17. Laboratory Workflow Preflight Agent

价值主张：

> Validate an experimental workflow before it touches hardware.

用于证明：

- 推理与确定性执行分离；
- 高风险 Tool approval；
- 可追踪状态；
- 物理执行前验证；
- 执行后的反馈与复盘。

---

## 18. ROS Incident Copilot

价值主张：

> Turn robot logs into a failure timeline and next-test checklist.

用于证明：

- 多模态/多源上下文；
- trace analysis；
- failure taxonomy；
- human-in-the-loop diagnosis；
- 物理系统的可验证 Agent。

---

## 19. Agent Trace-to-Eval

价值主张：

> Turn every failed agent run into a reproducible regression test.

用于证明：

- Tool trace；
- workflow eval；
- failure clustering；
- rubric generation；
- continuous improvement。

三个案例可以共享统一底层思想：

```text
Observe
→ Structure
→ Validate
→ Execute
→ Trace
→ Diagnose
→ Evaluate
→ Improve
```

---

# Part V — 商业模式与投放

## 20. 产品层级

### Community / Open Source

- Agent Skill；
- 模板；
- Pattern Library；
- 本地 Architecture Pack；
- 示例案例；
- 基础验证脚本。

目标：传播、贡献、建立行业标准感。

### Team

- 共享 Architecture Workspace；
- 团队评审和版本管理；
- 私有项目 references；
- AI capability inventory；
- Architecture Decision Records；
- 私有 MCP Catalog；
- trace 和 eval；
- 多项目复用。

目标：形成可持续 SaaS 收入。

### Enterprise

- SSO；
- RBAC/ABAC；
- VPC/私有部署；
- secrets 和 OAuth；
- policy engine；
-审计；
- 数据保留策略；
- 自定义 connectors；
- 安全与合规；
- 专业服务。

目标：基础设施和治理预算。

---

## 21. 推荐投放路径

### 21.1 GitHub-first

发布：

- `ai-native-architect` Skill；
- 示例 Architecture Pack；
- 三个端到端案例；
- 可复制模板；
- 清晰的 before/after。

README 的核心 CTA：

> Describe one workflow. Get three architecture options and a reviewable implementation roadmap.

### 21.2 技术内容

持续发布：

- “什么时候不应该使用 Agent”
- “RAG、Workflow 与 Agent 的架构边界”
- “为什么 Multi-Agent 通常不是第一选择”
- “MCP 是协议，不是完整平台”
- “如何为 Agent 定义权限和人工审批”
- “如何把一次失败变成回归测试”
- “AI-native company 的成熟度模型”

### 21.3 Design Partner

首批用户建议来自：

- AI-native startup；
- 生物技术或实验室自动化团队；
- 机器人团队；
- 运营流程复杂的 SaaS 公司；
- AI 咨询团队。

提供一次高质量的架构工作坊，换取：

- 真实问题；
- 脱敏案例；
- 输出反馈；
- 是否愿意复用；
- 哪些部分愿意付费。

### 21.4 Website Funnel

```text
文章 / 白皮书 / 架构图
        ↓
Interactive Assessment
        ↓
Open-source Skill
        ↓
Architecture Pack
        ↓
Team Workspace
        ↓
MCP Control Plane / Enterprise
```

---

# Part VI — 90 天验证计划

## Phase 1：第 1–3 周

交付：

- Skill v0.1；
- 5 个核心模板；
- 3 个架构模式；
- 一个示例项目；
- 简单 Landing Page；
- 白皮书目录。

验证：

- 找 5 位工程负责人完成一次架构流程；
- 记录每一步卡点；
- 不急于建设 SaaS。

成功门槛：

- 至少 4/5 能完成；
- 至少 3/5 认为输出可以用于团队评审；
- 至少 2/5 愿意在另一个项目再次使用。

---

## Phase 2：第 4–8 周

交付：

- 完整 Architecture Pack；
- Option scoring；
- ADR 生成；
- MCP capability map；
- trust/eval 模板；
- 两个脱敏案例；
- Architecture Assessment。

验证：

- 用户是否会修改并保留这些文件；
- 哪些输出最常被分享；
- 是否出现重复的 connector、权限和 trace 痛点。

成功门槛：

- 3 个真实团队采用；
- 2 个团队将产物进入内部设计评审；
- 至少 1 个团队愿意付费获得协作或私有功能。

---

## Phase 3：第 9–12 周

只有当需求被反复验证后，才做 Control Plane 原型。

交付：

- 私有 MCP Catalog；
- 统一 Gateway；
- 简单 policy；
- health 和 trace；
- 只读 Tool Bundle；
- 一个审批型 write action。

成功门槛：

- 2 个团队愿意将真实 MCP Server 接入；
- 用户明确认为统一治理优于各项目自行配置；
- 能证明更少的工具暴露、更清晰的审计或更快的集成。

---

# Part VII — 成功指标

## 22. Skill 指标

- 从想法到 Architecture Pack 的时间；
- 用户完成率；
- 方案被团队接受的比例；
- 输出被修改和保留的比例；
- 未知假设被识别的数量；
- 架构返工次数；
- 用户再次使用率。

## 23. Control Plane 指标

- 接入的有效 capability 数量；
- 每个项目复用的 connector 数量；
- Tool selection accuracy；
- policy block / approval rate；
- Tool invocation success rate；
- schema change detection；
- mean time to diagnose；
- 高风险操作的未经授权调用次数；
- 每个工作流的成功率和人工介入率。

## 24. 业务指标

- Design partner 到付费转换；
- Team 到 Enterprise 转换；
- 单个组织活跃项目数；
- 单个组织复用的架构模式数；
- 客户节省的集成或评审时间；
- 上线周期缩短程度。

---

# Part VIII — 关键风险

## 25. “泛泛而谈”的风险

架构助手很容易输出看似专业、实则通用的内容。

缓解：

- 强制引用项目证据；
- 标注 Confirmed / Inferred / Unknown；
- 每个组件必须对应明确业务需求；
- 每个建议必须包含不采用它的条件；
- 用户未确认前不得进入实现。

## 26. 范围过大的风险

“帮助所有公司 AI-native”过于宽泛。

缓解：

- 先选择 1–2 类 workflow；
- 首个 ICP 聚焦中型、API-rich 团队；
- 先做架构包，不做完整企业转型平台。

## 27. 安全责任

MCP Gateway 会接触身份、凭证和高风险动作。

缓解：

- 第一版 read-only；
- 最小权限；
- short-lived credentials；
- write action 必须审批；
- 不自动 fallback 高风险 Tool；
- 完整审计；
- 引入专业安全评审。

## 28. 官方 Registry 与现有 Marketplace

单纯复制 Server Directory 没有价值。

缓解：

- 使用官方 Registry 作为 metadata source；
- 差异化放在企业私有能力、trust、policy、runtime 和 evaluation；
- 不与基础协议和公共 metadata 层竞争。

## 29. Skill 被复制

Markdown 指令容易复制。

缓解：

- 建立高质量 pattern library；
- 积累真实架构案例和评测；
- 将团队协作、私有上下文和 Control Plane 作为商业壁垒；
- 形成社区和品牌；
- 持续维护参考架构。

---

# Part IX — 最终建议

## 30. 最值得先做的产品

优先做：

> **AI Native Architect Skill**

但定位必须高于“帮你画架构图”：

> **It guides a team through evidence gathering, workflow decomposition, autonomy decisions, architecture trade-offs, trust boundaries, evaluation and rollout.**

同时用一个垂直案例证明其价值：

> **Laboratory Workflow Preflight** 或 **ROS Incident Copilot**

等到真实团队反复出现以下痛点时，再建设 MCP Control Plane：

- 多个 Agent 重复连接相同工具；
- Tool schema 太多；
- 权限和 OAuth 混乱；
- 缺少统一 trace；
- 不知道哪个 Server 可信；
- 需要按用户、区域和风险动态暴露工具；
- 需要将失败自动沉淀为 eval。

## 31. 最清晰的长期愿景

> **Build the architecture and control layer that helps companies become AI-native without losing reliability, accountability, or control.**

产品故事：

```text
First, we help teams decide what should become agentic.
Then, we help them design it correctly.
Finally, we provide the control plane to run it safely.
```

---

# Appendix A — 可能的产品名称

仅作为工作名称，使用前需要进行商标和域名检索。

### Skill / Architecture

- AI Native Architect
- Agent Blueprint
- Verifiable AI Architect
- Agent Architecture Studio
- Workflow-to-Agent

### MCP Control Plane

- ToolMesh
- ContextRouter
- Capability Cloud
- MCP Fabric
- Agent Capability Plane

### 内容品牌

- AI-Native Architecture Handbook
- Agent Systems Playbook
- Verifiable Agents
- MCP Architecture Atlas

推荐组合：

- **AI Native Architect** — Skill
- **MCP Fabric** — Control Plane
- **The AI-Native Architecture Playbook** — Whitepaper / Website

---

# Appendix B — 研究依据

本构想参考了以下公开官方资料在 2026-07-13 时的状态：

- Model Context Protocol 官方介绍与架构文档；
- MCP Specification 2025-11-25；
- MCP 官方 Registry 与 Registry Aggregators 文档；
- Agent Skills 官方格式规范；
- MCP 官方 “Build with Agent Skills” 文档；
- OpenRouter Provider Routing 文档。

这些资料说明：

- MCP 负责标准化 Host、Client、Server 之间的上下文和能力交换；
- 官方 Registry 已承担公共 Server metadata 和 namespace verification；
- 官方设计明确允许下游 aggregator 增加评分、安全扫描和自定义 metadata；
- Agent Skills 已成为可移植的 `SKILL.md` 工作流封装形式；
- 因此，本产品更应聚焦上层架构决策、企业私有能力治理和运行控制，而不是复制协议、基础 Registry 或普通 MCP Server 教程。
