# 方案总览：把 aevatar 改造成 FDE 实施平台

> status: draft v1 · owner: eanzhao · date: 2026-08-25
> 依据：6 路概念/代码库调研 + 3 路部署审计（见 [01-context.md](01-context.md)）

## 愿景

aevatar 成为 **FDE（Forward Deployed Engineer）交付企业级 agent 方案的标准平台**：FDE 在客户现场用 aevatar 建模业务、编排 agent、接通渠道与凭证，交付物生来就带权限边界、审计锚点与灰度能力；每一单 bespoke 交付都能被收割成可复售模板——"gravel road → paved highway" 的回流循环成为平台机制，而不是团队自觉。

## 核心判断（改造的出发点）

1. **aevatar 的能力本体（动词侧）已是一流**：typed 准入、风险分级、sealed profile、tool catalog 交集、saga 补偿、灰度发布——企业试点最常死掉的权限与审计问题，aevatar 天然带着。
2. **数据本体（名词侧）缺位**：客户业务对象（订单/工单/设备）在平台内没有建模位置，agent 每轮靠 connector 拉瞬时 payload。这是与 Palantir Foundry 的核心不对称，也是 FDE 建模客户业务时最缺的表面。
3. **FDE 交付面已有骨架**：Workflow Delivery Center、agent 化 authoring skills（对标 Palantir "AI FDE"）、hybrid 执行机制（NyxID node 隧道、private_ssh、HMAC webhook）。
4. **回流循环已断**：交付包是 ConfigMap 运维变更、客户端集成靠手写 REST+轮询、交付知识碎在四处。这三条是"收割侧机制缺失"的直接证据。
5. **私有化不是先决条件**：Palantir FDE 也极少让客户自装 Foundry。近期 hosted+hybrid 即可开展交付；single-tenant VPC 托管是中期工程题（端口齐、适配器单一，缺的是部署工程件）。

## 七个工作流（Workstreams）

| # | 工作流 | 一句话 | 优先级 |
|---|---|---|---|
| WS-A | Scope Ontology（数据本体层） | 让 FDE 为客户建模业务对象，agent 站在治理过的对象上读写 | P0 |
| WS-B | FDE 交付工作台 | package registry、capability manifest/diff、engagement playbook、runbook 收敛 | P0 |
| WS-C | 评测与信任 | shadow replay 评测、发布回归门禁、统一变更审计账本 | P1 |
| WS-D | 身份与权限完备化 | channel 未绑定发送者降权收尾、org-scope 连接、admission 复制税工具化 | P1 |
| WS-E | 部署形态梯度 | Tier1 hybrid 硬化 → Tier2 single-tenant VPC kit → Tier3 数据驻留 | P1/P2 |
| WS-F | 客户端集成 SDK | OSDK 对标：从 readmodel 契约生成 typed SDK（202+poll 内建） | P2 |
| WS-G | 经济回路 | template harvesting 制度、复用率/边际成本度量、AI FDE 产品化 | P0（制度）/P2（工具） |

详见 [02-workstreams.md](02-workstreams.md)。

## 阶段划分

- **Phase 0（第 0–1 月）**：不写平台代码就能做的事——runbook 收敛、用现有 hybrid 机制打样 1–2 个真实交付、建立基线指标。
- **Phase 1（1–3 月）**：Scope Ontology MVP、channel 权限收尾验证、package registry、delivery acceptance 出路。
- **Phase 2（3–6 月）**：评测门禁、capability manifest、SDK v1。
- **Phase 3（6–12 月）**：VPC 部署 kit、ontology 完整化（link 遍历 + action 写回）、度量闭环。

详见 [03-roadmap.md](03-roadmap.md)。

## 成败判据

FDE 平台成立的唯一硬指标：**现场经验回流平台的速度快过交付人头的增长**。具体化为四个可测量代理指标（见 [04-operating-model.md](04-operating-model.md)）：交付周期中位数、模板复用率、每单边际交付成本、harvest review 产出率。

## 文档结构

- [01-context.md](01-context.md) — 背景与差距分析（调研结论浓缩）
- [02-workstreams.md](02-workstreams.md) — 七个工作流的详细设计
- [03-roadmap.md](03-roadmap.md) — 阶段、里程碑与验收标准
- [04-operating-model.md](04-operating-model.md) — FDE 组织与交付流程
- [05-risks.md](05-risks.md) — 风险与对策
