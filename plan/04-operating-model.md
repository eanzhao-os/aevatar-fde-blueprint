# FDE 组织与交付流程

## 1. Pod 结构（对标 Palantir Delta/Echo）

一个 engagement 一个 pod：

- **Delta（交付工程师）**：与平台工程师同一技术标准；在客户环境用 aevatar 建模（WS-A ontology）、编排 workflow、接 channel/凭证/执行节点；写的是跑在生产的配置与代码，不是 PPT。
- **Echo（部署策略师）**：行业/客户侧专家；找最值钱的问题、定验收口径、保采用率。小团队初期可由创始人/产品兼任，但角色要显式。
- **平台收割 owner（共享角色）**：不驻场；主持每单 harvest review，把可复用产物推进 registry / milestone。这是回流循环的执行者，缺了它 FDE 就退化为咨询。**初始由 eanzhao 兼任，Phase 1 结束前落实专人。**

每个 workstream 启动时必须指定 owner（记入 00-overview 附注）；未指定 owner 的 workstream 不启动。

## 2. Engagement 生命周期

```
Scoping（1 周内）
  Echo 选题：任务关键、可测量、4 周内可见价值；明确数据敏感级 → 选部署 Tier
    │
Bootcamp（1–5 天，对标 Palantir AIP bootcamp）
  真实数据、真实权限；产出不是 demo 而是生产骨架：
  day1 发现（连服务、列能力、ontology 草模）
  day2–3 构建（workflow draft-run 循环、profile 绑定）
  day4 接线（channel/webhook/执行节点）
  day5 验收基线（eval 集初版 + acceptance 通过）
    │
Hardening（2–6 周）
  边界用例、审批策略、灰度发布、监控告警；每周向客户演示 delta
    │
Handover
  交付物清单 = capability manifest 快照 + runbook 附录 + eval 基线
  客户侧 owner 培训（含 AI FDE 技能的使用）
    │
Harvest review（收尾必开，产出三选一）
  ① 沉淀模板 → package registry
  ② 沉淀平台 issue → milestone
  ③ 显式"不收割"及原因记录
```

**Phase 0–1 的降级口径**（生命周期引用的 Phase 2 产物落地前）：manifest 快照用人工整理的 artifact 清单代替（P2-M2 后自动化）；eval 基线用人工 10–30 条用例（P2-M1 后随语料增厚）；acceptance 按 runbook 的人工验收 workaround（P1-M4 前）。降级不豁免流程本身——每单仍必须产出这三样。

## 3. 回流协议

- **输入**：每单的 harvest review 记录 + 交付计时 + 摩擦日志（Delta 在交付中随手记的"平台让我慢下来的地方"）。
- **仲裁**：收割 owner 每月汇总，与平台 roadmap 对齐；重复出现 ≥2 次的摩擦自动升级为 P1 issue。
- **输出**：模板进 catalog；平台改进进 milestone；runbook 增量更新。
- **红线**：交付分支/私有 fork 不允许长期存在——bespoke 逻辑要么变量化进模板，要么留在客户 scope 的数据（workflow/profile/ontology）里，不进平台代码。

## 4. 度量（成败判据的代理指标）

| 指标 | 定义 | 方向 |
|---|---|---|
| 交付周期中位数 | scoping 开始 → acceptance 通过 | ↓ |
| 模板复用率 | 新单中由既有模板承担的步骤占比 | ↑ |
| 每单边际成本 | Delta 人日 + 运维变更次数 | ↓ |
| Harvest 产出率 | 每单沉淀的模板/issue 数 | 稳定 >0 |
| 回流判据 | 按单看：连续三单的每单边际成本趋势 ↓ 且模板复用率趋势 ↑（单均口径，与人头数无关，避免"人头不变即达标"的空判据） | 成立 = FDE 是产品战略；不成立 = 在做咨询 |

## 5. AI FDE：平台自己的交付员

aevatar 已有的 agent 化 authoring 技能（workflow authoring、profile 管理、channel 交付一族）就是 "AI FDE" 的雏形——Palantir 已把同名产品化（agent 操作 Foundry 完成建管道、改本体、提 proposal）。

演进路径：内部工具（现状）→ Delta 的副驾驶（bootcamp 中 agent 执行发现/draft-run/验证，人审批关键步骤）→ 面向客户成功团队的产品能力（小改动客户自助，FDE 只接新域）。每一步的权限边界都由既有 sealed profile + admission 机制承载，不需要新的信任模型。
