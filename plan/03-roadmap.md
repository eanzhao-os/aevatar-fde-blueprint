# 阶段、里程碑与验收标准

> 时间为相对区间，按一个小型平台团队 + 1–2 个 FDE pod 的带宽估算；与 aevatar 既有 milestone 冲突时，本路线让位于生产稳定性工作（让位顺序见各阶段注记）。

## Phase 0 · 立即可做（第 0–1 月）——零平台功能开发

> "零平台功能开发"指不启动新功能工作流；打样中暴露的缺陷修复照常走 aevatar 主仓库流程，不计入违约。

| 里程碑 | 内容 | 验收 |
|---|---|---|
| P0-M1 FDE runbook v1 | 收敛四处知识为单一可执行手册（authoring 调试环、channel 接线、admission 流程、故障对照表、**manual/none acceptance 的人工验收 workaround**）；废弃 Ornn 上的已知错误版本 skill | 一名未参与过交付的团队成员（非作者）仅凭 runbook 完成一次端到端交付演练 |
| P0-M2 Hybrid 打样 | 用现有机制（NyxID node 隧道 / private_ssh / webhook）完成 1–2 个真实客户交付 | 客户内网能力被 agent 调用，零入站端口；交付全程计时 |
| P0-M3 Harvest review 制度 | 制度落地 + 收割 owner 指定 + 度量口径定义；**registry 落地前，模板类产物先入临时暂存（本 repo 旁的 template 暂存目录），P1-M3 后迁入 registry** | 打样交付各产出一次 harvest review 记录 |
| P0-M4 基线指标 | 记录现状：交付周期、人工步骤数、每单成本；**冻结后续阶段的对比口径与达标阈值** | 基线报告一份（P3-M5 以它为对照） |

## Phase 1 · P0 建设（第 1–3 月）

> 让位顺序（生产稳定性冲突时）：先滑 P1-M3 registry，再滑 P1-M1 ontology MVP；P1-M2 验证类工作最后放弃。

| 里程碑 | 内容 | 验收 |
|---|---|---|
| P1-M1 Scope Ontology MVP | object type catalog actor + typed property + admitted-operation sourcing + **object-ingestion actor（写侧触发物化）** + `object_query` 工具 + ES 物化（link/action 留 Phase 3） | 一个真实客户域用 ontology 重建**读路径**；人工评测集（10–30 条，即 bootcamp 验收基线协议）对照 connector 直拉出数据 |
| P1-M2 Channel 权限收尾 | 先实测 per-sender catalog 现状（ADR-0018 绑定已存在）；补未绑定发送者 restricted catalog + 绑定引导 | 绑定用户按本人 authority（实测记录）；未绑定用户落入降权目录而非 owner 全权 |
| P1-M3 Package registry | 交付包 upload API + draft→validate→publish + 版本化 | 新客户包全程不触 ConfigMap；一次语义修订走版本发布 |
| P1-M4 Acceptance 出路 | manual/none acceptance 的人工验收凭证上传（WS-B4 前半；P0-M2 打样已用 runbook workaround 顶住） | 一个 manual-acceptance 包能到 `ready` |

## Phase 2 · 评测与集成（第 3–6 月）

| 里程碑 | 内容 | 验收 |
|---|---|---|
| P2-M1 评测语料 + 回归门禁 | opt-in 评测语料捕获（脱敏/保留策略 ADR）+ 候选版本重执行评测 + publish 前 eval gate + eval-based acceptance | 一次 profile 灰度全程有 eval 对比；一次 delivery 用回归集验收 |
| P2-M2 Capability manifest | manifest aggregate actor + 当前态 readmodel + diff 导出 artifact + proposal 流转 | 一次真实交付的全部 artifact 出一张 manifest diff 并走评审 |
| P2-M3 SDK v1 | readmodel 契约生成 TS/C# 客户端（202+poll 内建）+ workflow typed 客户端 | debug 脚本族被 SDK 重写；外部系统用 SDK 完成触发→观察→读结果闭环 |
| P2-M4 Admission 工具化 | 错误载荷内嵌补救 + digest 自动拷贝 | 一次全新集成 admission 零手工拷贝 |
| P2-M5 Template catalog v1 | P0-M3 暂存的模板迁入 registry，变量化/脱敏规范成文（WS-G2） | 至少一个模板从暂存迁入并被引用 |

## Phase 3 · 部署梯度与本体完整化（第 6–12 月）

> 阶段内顺序：P3-M4 审计账本先于 P3-M3（action 写回依赖账本溯源）。

| 里程碑 | 内容 | 验收 |
|---|---|---|
| P3-M1 Tier1 fleet 化 | node/private_ssh fleet 管理（批量注册、健康监控）+ webhook outbound callback | 一个 scope 下 ≥3 节点统一管理；掉线可告警 |
| P3-M2 Tier2 VPC kit | helm/manifests + 配置模板化（config + code-default 全量清扫 + guard）+ secret bootstrap 工具 + NyxID 随附部署手册 + 镜像发布管道 | 干净 k8s 集群按手册从零拉起全栈，通过 `docs/canon/agent-turn-tool-catalog.md` 定义的 production verification matrix 全项 |
| P3-M4 统一审计账本 | ADR-0039 深化：跨入口 mutation ledger + impact preview | 任意写操作可溯源到 admission proof |
| P3-M3 Ontology 完整化 | link taxonomy + `link_traverse` + `object_action` 写回（接 admission/审批/**P3-M4 账本**）+ retrieval context | 五层 OAG 栈端到端演示：检索→查询→遍历→action 写回→账本溯源 |
| P3-M5 度量闭环 | WS-G 度量看板 + 连续三单趋势 | 对照 P0-M4 冻结的基线：交付周期中位数下降 ≥30%，或模板承担步骤占比 ≥40%（阈值随基线报告一次性冻结，不事后调整） |
| P3-M6 延伸项 | org-scope 连接（WS-D2）；AI FDE 从副驾驶到客户自助的第一步（WS-G3） | org 级连接完成一次真实交付；一次小改动由客户侧在人审批下自助完成 |

## 依赖关系

```
P0-M1 runbook ─────────────┐
P0-M2 hybrid 打样 ──┐      ├─→ P1-M3 registry ─→ P2-M5 template catalog ─→ P3-M5 度量闭环
P0-M3 harvest 制度 ─┴─(P0-M4 基线冻结)┘
P0-M1 workaround ─→ P1-M4 acceptance 出路 ─→ P2-M1 eval-based acceptance
P1-M1 ontology MVP ─→ P3-M3 ontology 完整化 ←─ P3-M4 审计账本（先行）
P1-M2 channel 收尾 ──(独立)
P2-M1 eval ─→ P3-M5 度量闭环
P2-M3 SDK ─→（P3-M3 后升级为真 OSDK）
P2-M2 manifest ─→ P3-M2 VPC kit（交付物清单依赖 manifest）
```

## 不做清单（显式）

- 不做客户自运维私有化安装器/license（Tier3 之外），除非出现足够大的订单；
- 不做第二套投影/消息主链；ontology 物化必须走既有 projection pipeline，且只由写侧触发；
- 不改 NyxID / chrono-* 外部仓库（随附部署≠修改）；
- 不在 Phase 1–2 做 per-tenant 数据驻留；
- 不在本路线内重做 channel 发送者绑定（ADR-0018 已存在，只做收尾硬化）。
