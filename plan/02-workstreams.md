# 工作流详细设计

七个工作流。每个含：目标 / 交付物 / 架构落点（对齐 aevatar CLAUDE.md 既有约束：单一主干、聚合 actor 化、projection 主链复用、外部仓库无改动权）/ 验收。

---

## WS-A · Scope Ontology（数据本体层）— P0

**目标**：让 FDE 能为每个客户 scope 建模业务对象（object type / property / link / action），agent 站在治理过的对象上读写，补齐 AIP 五层栈的 Layer 1–2。

**交付物**

1. **Object type catalog actor**（scope-owned，对齐 `connector.catalog` / `role.catalog` 既有形态）：
   - object type：稳定 token（复用 `<module>.<entity>` 词汇规则）+ typed property schema（proto 定义，走字段决策树）；
   - link taxonomy：跨 type 的关系词汇（共享 edge 词汇表，替代各 projector 私有字符串）；
   - sourcing 声明：每个 type 绑定背后的**已准入操作**（connector / NyxID operation）作为取数与写回来源——把 Foundry 的 "pipeline backs object type" 换成 "**admitted operation backs object type**"，不依赖外部仓库改动；
   - 生命周期：draft → validate → publish（复用 profile 的封印/灰度机制）。
2. **实例物化（写侧触发）**：新增 scope-owned 的 object-ingestion actor 作为实例事实拥有者——外部权威源的数据由它以 committed snapshot event 提交（携带来源坐标与拉取水位），projection 主链消费这些 committed 事实物化为 ES 文档 + Neo4j link（不建第二投影轨）。物化只由写侧触发（ingestion 提交或原生 actor 提交），查询路径禁止补物化；readmodel 单调覆盖、带权威版本水位。
3. **LLM 工具面**：`object_query`（过滤/聚合）、`link_traverse`（关系遍历）、`object_action`（只能填已声明 action 的 typed 参数，继承 admission 风险分级与审批）。全部进 tool catalog 交集体系，遵守 turn class 预算。
4. **Retrieval context**：per-profile 声明固定 object set / 语义检索作为系统注入上下文（对标 AIP Layer 1）。

**架构落点**：catalog 与 ingestion 均 actor 化拥有事实；实例的权威源可以是外部系统（此时 ingestion actor 提交的 snapshot 是 replica 事实 + 来源坐标）或 aevatar actor（原生对象）；两种都显式建模，不混用。写回只经 `object_action` → admitted operation，杜绝自由写。

**验收（MVP 只验读路径）**：选一个真实客户域（如 HR/finance 工作流套件）用 ontology 重建读路径；用人工构造的 10–30 条评测用例（即 bootcamp 验收基线协议，见 05-risks R7）对比现状 connector 直拉与 object query 的防幻觉与任务成功率；发布一份 ontology 建模指南进 runbook。link 遍历与 action 写回的验收在 Phase 3（P3-M3）。

---

## WS-B · FDE 交付工作台 — P0

**目标**：把"交付一个客户"从运维变更 + 部落知识，变成平台内的数据操作 + 可执行手册。

**交付物**

1. **Package registry**：交付包成为数据——upload API + draft→validate→publish 生命周期 + 版本化；替代 ConfigMap `Aevatar:Delivery:Packages`。语义修订走新版本发布而非 revoke/reinstall。
2. **Capability manifest**：新增 scope-owned 的 manifest aggregate actor 作为聚合事实拥有者（聚合必须 actor 化，不做 query-time 拼装），物化"当前 manifest" readmodel；历史版本与 diff 建模为该 actor 拥有的导出 artifact（readmodel 不默认保留历史视图）；变更以 proposal 流转（对标 Foundry ontology proposal ≈ PR）。
3. **Engagement playbook**：bootcamp 模板（对标 Palantir 1–5 天：day1 发现 → day5 生产骨架）+ 单一 FDE runbook，收敛 docs/canon、nyxid plugin skills、Ornn skills、私人 memory 四处的部落知识（含：draft-run/submit 调试环、channel 接线全流程与故障对照表、admission 复制流程）。这本身就是回流循环的第一次收割。
4. **Delivery Center 升级**：manual/none acceptance 的出路（人工验收凭证上传）；channel wiring 纳入 delivery journey（现在是第二条无文档旅程）。

**验收**：新客户包全程不触 ConfigMap；一次真实交付的全部 artifact 能出一张 manifest diff；新人 FDE 仅凭 runbook 完成一次端到端交付。

---

## WS-C · 评测与信任 — P1

**目标**：交付验收从"跑通一次"升级为"回归集绿"；agent 变更可审计、可预览。

**交付物**

1. **评测语料与 shadow 评测**：注意两个前提——离线评测**不是**既有 replay 例外类目（那只覆盖修复/迁移/灾备），且 ADR-0039 刻意不留存完整 prompt/工具参数，历史 turn 无法直接重放。因此先建 **opt-in 评测语料捕获**：被标记的 scope/会话在脱敏与保留策略（需新 ADR）下记录可重放的 turn 输入；候选 profile/workflow 对语料做重执行评测（fresh LLM 调用、工具走 stub/只读），配合既有 SHADOW activation 的对照运行。
2. **发布回归门禁**：publish 前必须通过 scope 绑定的 eval suite；SHADOW activation 升级为带评分的对照运行。
3. **Eval-based acceptance**：Delivery acceptance policy 增加 eval 类型——验收物是回归基线而非单次演示。
4. **统一变更审计账本**：深化 ADR-0039——跨 turn/workflow/channel 的 agent mutation ledger（谁、以何身份、经哪个 admission、动了什么对象），配合 WS-A 的 `object_action` 形成写路径全审计；高风险 action 增加 impact preview。

**验收**：一次 profile 灰度全程有 eval 对比数据；任意一笔 agent 写操作可在账本中溯源到 admission proof。

---

## WS-D · 身份与权限完备化 — P0（sender 绑定）/ P1（其余）

**目标**：把"agent 以调用者权限运行"补齐到所有入口。

**交付物**

1. **Channel sender 绑定收尾**：发送者显式绑定**已存在**（ADR-0018：`/init` external-subject binding、CAS 替换、fail-closed；`ExternalIdentityBindingGAgent` + per-sender token exchange）。剩余三件收尾：① 未绑定发送者的 fallback 语义——当前回落到 bot owner 的环境配置，改为明确的降权目录（restricted catalog）；② 验证并固化"绑定用户按本人 authority 交集"的验收口径（可能已成立，先测再改）；③ 绑定引导体验（群内 typed 提示而非静默降权）。
2. **Org-scope 连接**：delivery 支持组织级 connected-service 连接（当前仅个人 scope）。
3. **Admission 复制税工具化**：fail-closed 错误载荷内嵌补救步骤与精确 digest；authoring 工具自动从 typed listing 拷贝 `user_service_id`/digest；消除逐单手工发现成本。

**验收**：先实测"同一 Lark 群内两个不同绑定用户是否已得到不同 effective catalog"（若已成立则记录为既有能力，不重复建设）；未绑定用户落入 restricted catalog 而非 owner 全权；一次全新外部集成的 admission 全程无手工拷贝。

---

## WS-E · 部署形态梯度 — P1（Tier1）/ P2（Tier2）/按需（Tier3）

**目标**：给 FDE 一个随客户合规要求升级的部署菜单，而不是一刀切私有化。

**Tier 1 — hosted + hybrid 硬化（近期）**

- NyxID node 注册产品化：从"逐用户手工作坊"到 fleet 管理（批量注册、健康监控、掉线告警）；
- `private_ssh` 执行节点纳入 fleet：模板化 wrapper 部署、按 scope 的节点清单；
- webhook 双向化：补 outbound callback 契约（当前只有 fire-and-forget 202）。

**Tier 2 — single-tenant managed VPC（中期，FDE 大单标准答案）**

- helm chart / k8s manifests 进仓库（含 Garnet/Kafka/ES/Neo4j 依赖清单与 Orleans Garnet clustering 生产拓扑）；
- 配置模板化：清除烤死的运营商域名——不止 appsettings（chrono-ai.fun、llm.aelf.dev、cluster-local DNS），还包括编译期 C# 默认值（如 `NyxIdToolOptions.DefaultBaseUrl`、Studio 存储默认端点），做一次 config + code-default 全量清扫并加 CI guard 防回流；
- secret bootstrap 工具：keyring 生成、Neo4j 密码、audit hasher key 的安装器；
- NyxID 随附部署方案（部署≠修改，不违反外部仓库约束）+ 服务 slug 重注册手册；
- 镜像发布管道：版本化镜像 + SBOM + 签名。

**Tier 3 — 数据驻留（仅按大单触发）**

- per-tenant projection store（ES/Neo4j 物理隔离）；触发条件与成本模型先写清，不预建。

**验收**：Tier1——一个客户内网 API 经 node 隧道被 agent 调用、全程零入站端口；Tier2——在一个干净 k8s 集群按手册从零拉起全栈并通过生产验证矩阵。

---

## WS-F · 客户端集成 SDK（OSDK 对标）— P2

**目标**：客户与 FDE 不再手写 REST+轮询。

**交付物**

1. 从 readmodel / query port 契约生成 typed client SDK（TS + C# 起步）：202-accepted + 版本水位轮询内建为 `await until observed`；
2. workflow 触发/信号/恢复的 typed 客户端（吸收 draft-run/submit 部落知识）；
3. WS-A 落地后：object type → 客户侧 typed binding（真正的 OSDK 形态），token scope 到所需实体。

**验收**：现有 debug 脚本（draftrun.sh/bindrun.sh 一族）全部可用 SDK 重写且更短；一个外部系统用 SDK 完成"触发 workflow → 等待物化 → 读结果"闭环。

---

## WS-G · 经济回路（收割机制）— P0（制度）/ P2（工具）

**目标**：让 gravel→highway 成为制度和产品机制，不是团队自觉。

**交付物**

1. **Harvest review 制度**（Phase 0 即启动）：每单交付收尾必开，产出三选一——沉淀为模板（进 package registry）/ 沉淀为平台 issue（进 milestone）/ 显式记录"不收割"及原因。指定平台侧收割 owner。
2. **Template catalog**：bespoke workflow → 变量化/脱敏 → registry → 可复售模板 SKU；模板带版本与升级路径。
3. **AI FDE 产品化**：把 `aevatar-workflow-authoring` 一族 agent 交付技能从内部工具升级为面向客户成功团队的产品能力（agent 自己执行交付动作，人审批关键步骤）。
4. **度量**：交付周期中位数、模板复用率（新单中模板承担的步骤占比）、每单边际交付成本、harvest 产出率（每单沉淀物数量）。

**验收**：连续三单的度量数据成趋势；至少一个模板被第二个客户复用且交付周期显著缩短。
