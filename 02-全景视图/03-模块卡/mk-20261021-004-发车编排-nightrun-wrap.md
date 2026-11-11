---
id: mk-20261021-004
kind: 模块卡
domain: engine
status: verified
created_at: 2026-10-21
verified_at: 2026-10-21
repo_head: 67e5b04
run_id: queue-93-w040
anchors: ["pipeline-lab/scripts/nightrun-wrap.mjs:17", "pipeline-lab/scripts/nightrun-wrap.mjs:47", "pipeline-lab/scripts/nightrun-wrap.mjs:617", "pipeline-lab/scripts/nightrun-wrap.mjs:631", "90-过程/拍板记录表.md:103", "90-过程/执行台账.md:583"]
tags: ["模块卡", "发车", "nightrun-wrap", "夜车"]
related: ["mk-20261021-005", "mk-20261021-011"]
---
> **口径**：本卡话术/命名遵循《[00-话术统一口径正本](../00-话术统一口径正本.md)》（拍板#312）——双名制首现双写·撞车词带消歧（窗票/工序票·判色/判分）·两层口径 L1 车间（票级）/L2 派工台（窗级）。
> **消歧（本卡·口径正本 §3）**：票（工序票 L1／窗票 L2·随文标注）。

# 模块卡 · 发车编排（nightrun-wrap）

## §0 一句话

**解决什么**：把「夜车一班」压缩成一条命令——发车（queue 派班跑票）→收车后四连（病卡/诊断/方案件/晨报）全自动串起来。**为什么存在**：夜车要无人值守跑通宵，编排件让每个子件「该静则静、单件炸不挡后续」（`90-过程/拍板记录表.md:103` 拍板#84② 编排语义），人只看晨报。**边界**：wrap 零判断逻辑全下放子件；不碰判卷语义；LLM 密钥值零入账零打印。

## §典型事件链

| # | 事件 | 锚 |
|---|---|---|
| 1 | 编排定形：拍板#84②——FAIL 接维修环入编排（fail-loop·授权制改造） | `90-过程/拍板记录表.md:103` |
| 2 | 实物：queue-40 班全链留痕在册——queue-events.jsonl 9 行（dispatched/gate_passed/gate_failed 三事件型）+ledger 11 行（enqueued/entry_state 两事件型） | `90-过程/执行台账.md:583`（W-041 实勘） |
| 3 | 工厂自检：发车前 factoryPreflight 先查 LLM 配置（拍板#74·配置坏=发车前停·不烧夜里的钱） | 本卡接口节·锚 pipeline-lab/scripts/nightrun-wrap.mjs:617 |

## 接口

CLI：`node pipeline-lab/scripts/nightrun-wrap.mjs --manifest <m> [--root <r>] [--ts-dir <引擎ts>] [--skip-queue] [--max-calls 5] [--env <ts/.env>] [--班次 <名>]`——一条命令：发车（queue）→收车后四连（病卡/诊断/方案件/晨报）。
导出读账函数：readFinalStates 重放式取各版本终态（锚 pipeline-lab/scripts/nightrun-wrap.mjs:17·ledger 状态机+gate 事件双源覆盖）·readRounds（锚 pipeline-lab/scripts/nightrun-wrap.mjs:47）·factoryPreflight 发车前工厂侧 LLM 配置自检（锚 pipeline-lab/scripts/nightrun-wrap.mjs:617）。

## 实现概览

四件自带「该静则静」（无病零卡/非 FAIL skip/无诊断不出方案）——wrap 零判断逻辑全跑；单件异常不挡后续（warnings 收集·AFK 韧性）。main 编排（锚 pipeline-lab/scripts/nightrun-wrap.mjs:631）：发车 queue 跑切片→收车后逐件拉起（诊断 diagnose/方案 propose-fix/病卡回写/晨报渲染）。

```mermaid
flowchart TB
    M[manifest 排期单<br/>versions[] 波次] --> PF{factoryPreflight<br/>LLM 配置自检}
    PF -->|配置坏| STOP[发车前停<br/>不烧夜里的钱]
    PF -->|OK| Q[queue 发车<br/>逐版串行派班]
    Q --> FAIL{班终态}
    FAIL -->|FAIL| FL[维修环 fail-loop<br/>拍板#84②]
    FL --> C[收车后四连<br/>顺序固定]
    FAIL -->|PASS| C
    C --> C1[病卡回写<br/>无病零卡]
    C --> C2[诊断 diagnose<br/>幂等 skip]
    C --> C3[方案件 propose-fix<br/>无产出不出]
    C --> C4[晨报 render-morning<br/>+信号]
    C --> RJ[班次状态反投<br/>+用例库回写]
```

**节点详注表**（图正下方·每节点做什么/边界/锚点）：

| 节点 | 详注 |
|---|---|
| manifest 排期单 | 夜车输入：versions[] 每元素=一波，逐版串行；decision_ref 入队资格闸（契约细节见 ct-20261021-002）——wrap 只消费不修改（业主资产）。 |
| factoryPreflight | 发车前工厂侧 LLM 配置自检（锚 pipeline-lab/scripts/nightrun-wrap.mjs:617·拍板#74）：配置坏=发车前停——钱和夜都不白烧。 |
| queue 发车 | 调 queue 派班（--skip-queue 可跳·重放收车半段）；逐版 V0→Vn 串行，班内判卷闸语义归 queue-runner（见时序卡 sq-20261021-001）。 |
| 维修环 fail-loop | FAIL 班接维修环（拍板#84②·授权制改造）：诊断→编译→重跑，预算尽即停不硬修（环内细节见 mk-20261021-011）。 |
| 收车后四连 | 顺序固定：病卡（:425 无病零卡）→诊断（:450 幂等 skip）→方案件（:481 诊断有产出才出）→晨报（:588）；单件异常 warnings 收集不挡后续——「该静则静」是子件自带语义，wrap 零判断。 |
| 班次状态反投 | 班毕把状态/轮数/WBS 摘要反投回队列面+用例库表 C-KEY 回写（缺回写参数=skip·幂等可重跑）。 |

## 调用方与被调用

被调：注册计划任务 register-nightrun-task.ps1（夜车定时）/业主手动发车。
调用：queue（发车·queue-runner 面）+维修环三件（diagnose/propose-fix·见卡 mk-20261021-011）+报告渲染件；配置解析 import 自 diagnose（resolveLlmConfigFromEnvText·拍板#74 发车前自检）。

## 从哪测

发车语义在 ts 面测试（node --test 全量·queue-runner.test.ts 等）；wrap 本体=夜车真班驱动（沙盒/e2e 在 pipeline-lab/tests）。

## 参数与预算

--max-calls 缺省 5 透传诊断（独立分账·与夜车预算互不挤占）；--skip-queue 跳发车只跑收车半段；LLM 密钥值零入账零打印。

## 时效

基准=仓 head 67e5b04（2026-10-21·W-040 核对）；失效触发=四连编排/终态重放口径改动（看执行台账 pipeline-lab 新笔）。
