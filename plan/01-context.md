# 背景与差距分析

> 本文是 2026-08-25 两轮调研（6 路概念/代码库 + 3 路部署审计）的浓缩。仓库事实核对至 aevatar `feature/integrate`（约 64 GAgent kind、109 proto 契约、63–64 query port）。

## 1. FDE：概念与判据

- **起源**：Palantir 2010s。Delta（与核心产品工程师同一标准、写生产代码）+ Echo（行业专家、找最值钱的问题）成对驻场，= "客户环境里的迷你创业公司"。
- **区别于咨询的本质**：现场工作不是计费产出，是**在生产环境做的产品研发**。"gravel road → paved highway"：bespoke 方案里重复出现的模式被核心团队收割为平台功能（Foundry 本身就是多年 Delta 工作的产品化）。
- **AI 时代复兴（2024–2026）**：OpenAI Deployment Company、Anthropic 系 Ode 等；根因是部署鸿沟而非能力鸿沟（MIT NANDA：约 95% 企业 GenAI 试点无损益影响，主要死于集成/权限/审计——方法论有争议但方向成立）。LLM 产品格外需要 FDE：last-mile 集成、context engineering、workflow discovery、信任建设。
- **失败模式**：服务陷阱（收入随人头线性）、雪花部署（无可复用抽象）、角色稀释。**成立判据只有一条：回流速度 > 人头增长。**

## 2. Ontology：语义层 + 动能层

Palantir Foundry Ontology = 运营语义层：

- **名词**：object type / property / link type，数据被"映射进"本体（pipeline backs each object type），附带权限、审计、写回。
- **动词**：Action（类型化、校验、有权限的原子变更 + 副作用写回，是**唯一合法写路径**）+ Function（注册在本体上的逻辑）。
- **治理**：对象级权限在交互时刻跨人与 agent 合成；本体变更走 branch → proposal（≈ PR）；OSDK 从本体子集生成 typed SDK，token 只 scope 到所需实体。
- **对 AI 的价值（AIP 五层 OAG 栈）**：retrieval context → object query（link 遍历）→ logic/functions → action tools → 全程治理。防幻觉是**架构性的**：看不见 = 查不到 = 算不了 = 做不成；agent 与人进同一审计日志。

## 3. aevatar 对位：动词强、名词缺

| Foundry 概念 | aevatar 现状 | 评估 |
|---|---|---|
| Object type 词汇 | `AgentKindToken` 受控词汇（约 64 kind）——只覆盖平台自身实体，目录 code-resident | 强·平台实体 / **缺·客户实体** |
| Properties | 约 109 个 proto 契约，字段决策树禁语义进 bag | 强 |
| Link types | Neo4j owner graph 机械层优秀；edge 词汇是各 projector 私有字符串，仅 workflow/scripting 两域 | 强·机械 / 弱·语义 |
| Object views | ES readmodel（显式 mapping 治理）+ 64 个 QueryPort，带 StateVersion 水位 | 强 |
| Actions | typed capability admission（read_only/write/destructive、binder 授权、saga 补偿、审批 continuation） | 强·授权 / 中·闭环（原子性、impact preview、统一审计待深化，ADR-0039 有基础） |
| Functions | workflow YAML 原语 + Ornn skills；step-type 词汇 stringly，YAML 不可查询，skill 语义=prose | 中 |
| 权限继承 | tool catalog 交集代数 + sealed proof + fail-closed；channel 侧已有发送者显式绑定（ADR-0018：`/init` external-subject binding + per-sender token exchange，`ExternalIdentityBindingGAgent`）；剩余缺口：未绑定发送者回落到 bot owner 环境配置、restricted catalog 语义未定义、无 org-scope 连接 | 强·机制 / 中·channel 收尾 |
| 本体版本化 | draft→validate→publish + sha256 封印 + 灰度 + pin 快照——per-artifact 强；无跨 artifact branch/proposal/diff | 中 |
| OSDK | 无。REST + 202 轮询，每个客户端手写 poll loop | **缺** |
| 数据摄取 | 无。connector 现场取数不落地 | **缺** |

**结论**：Foundry 是数据先行的本体 + 动词；aevatar 是动词先行的本体、名词缺位。AIP 五层栈里 aevatar 的 Layer 3–5 有坚实对应物，Layer 1–2 只对平台自身实体成立。

## 4. 交付面现状

已有资产：

- **Workflow Delivery Center（`/delivery`）**：显式 FDE 交付面——typed variables、connection slots、acceptance policy、trigger intent。
- **Agent 化交付动作**：`aevatar-workflow-authoring` 一族 skills 让 agent 完成"发现能力 → draft-run → 验证终态 → 持久化"闭环，对标 Palantir 刚产品化的 "AI FDE"。
- **治理先行**：demo 即 production skeleton（权限/审计/准入不是事后补的）。

交付摩擦（= 回流循环已断的证据）：

1. Delivery package 是部署配置不是数据：每个新客户包 = ConfigMap 运维变更，语义微调即 revoke/reinstall。
2. 调试 authored workflow 依赖部落知识 REST 编排（direct bearer vs proxy、full-actorId resume、`serviceId == workflowId` 不变量），活在私人 memory 里。
3. Channel 接线半手工且故障不透明（Lark 控制台手工激活、502 包裹 NyxID 409、镜像遗漏静默丢消息、Agent Key 泄漏 #2812）。
4. 202-accepted API 把轮询逻辑推给每个客户端。
5. 知识碎在四处：docs/canon、nyxid plugin skills、Ornn 已发布 skill（有已知错误版本）、私人 memory。
6. Admission 复制税：exact digest / `user_service_id` 人工拷贝，fail-closed 错误的补救在 skills 里而不在错误载荷里。
7. manual/none acceptance 的包永远到不了 `ready`。

## 5. 私有化部署审计结论

- **可自托管**：后端单镜像 Dockerfile；Garnet/Kafka/ES/Neo4j 全可自建；三档 boot profile（local / persistent-local / distributed）。
- **卡死**：NyxID 是四合一硬依赖（IdP + 凭证库 + 出口代理 + 服务目录/消息中继）；Ornn、sandbox、storage、search 全经它到达；鉴权仅 Development 可关。没有它：channels、connected services、Agent Key 自动化、代码执行、远程 skill 全灭。
- **缺工程件**：无 helm/k8s manifests（生产部署管道在仓库外）、appsettings 烤死运营商域名、无安装器/license/租户机制、密钥手工 bootstrap。
- **架构判词**：端口齐、适配器单一（auth 是通用 OIDC；LLM 完全可插拔，direct key 一等公民；`ICodeExecutionPort`/`IRemoteSkillFetcher` 有 port 无第二实现）。
- **hybrid 机制已存在**：NyxID node 出站隧道（客户内网 HTTP API 注册为服务，零入站端口，aevatar 零改动）；`private_ssh` codex_exec（代码在客户机器执行，强制审批）；HMAC webhook ingress（客户系统触发托管 workflow）。
- **hybrid 缺口**：connector 只能从托管集群出网；scope 是逻辑隔离非数据驻留；private_ssh 是逐用户手工作坊。
