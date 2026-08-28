# aevatar FDE Blueprint

把 aevatar 改造成 **FDE（Forward Deployed Engineer）实施平台**的改造方案。

📄 **方案说明页**：https://eanzhao-os.github.io/aevatar-fde-blueprint/

## 一句话

aevatar 的能力本体（typed 准入 / sealed profile / 灰度治理）已是一流，缺的是数据本体（客户业务对象层）与收割机制（bespoke 交付 → 可复售模板的回路）。本方案用七个工作流、四个阶段补齐，让 "gravel road → paved highway" 成为平台机制而非团队自觉。

## 目录

| 文档 | 内容 |
|---|---|
| [plan/00-overview.md](plan/00-overview.md) | 方案总览：愿景、核心判断、工作流一览、阶段划分 |
| [plan/01-context.md](plan/01-context.md) | 背景与差距分析（FDE / Ontology 概念 + aevatar 对位 + 私有化审计） |
| [plan/02-workstreams.md](plan/02-workstreams.md) | 七个工作流详细设计（WS-A 数据本体层 … WS-G 经济回路） |
| [plan/03-roadmap.md](plan/03-roadmap.md) | 阶段、里程碑、验收标准、依赖关系、不做清单 |
| [plan/04-operating-model.md](plan/04-operating-model.md) | FDE 组织（Delta/Echo pod）、engagement 生命周期、回流协议、度量 |
| [plan/05-risks.md](plan/05-risks.md) | 风险与对策、诚实的不确定性清单 |

## 状态

draft v1 · 2026-08-25 · owner: eanzhao

方案落地项将转为 [aevatar](https://github.com/aevatarAI) 仓库的 issue/milestone 管理；本 repo 只保留策略层（见 05-risks R9）。

## 调研依据

- 6 路概念/代码库调研（FDE 概念史、Foundry Ontology 与 AIP、FDE×Agent 平台行业格局、aevatar 语义层盘点、aevatar 交付面盘点、补漏批评）
- 3 路部署审计（打包与启动要求、外部服务耦合、hybrid 执行机制）
- 仓库事实核对至 aevatar `feature/integrate`（2026-08-25）
