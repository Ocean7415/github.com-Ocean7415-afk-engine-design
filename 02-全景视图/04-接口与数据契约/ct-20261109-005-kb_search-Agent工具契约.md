---
id: ct-20261109-005
kind: 模块卡
domain: engine
status: verified
created_at: 2026-11-09
verified_at: 2026-11-09
repo_head: 62267be
run_id: w413
anchors: ["91-脚本/monitor-web/server.mjs:632", "91-脚本/monitor-web/server.mjs:646", "91-脚本/monitor-web/server.mjs:1153", "94-班外经验库RAG/query_engine.py:94", "94-班外经验库RAG/query_engine.py:96", "94-班外经验库RAG/query_engine.py:277", "94-班外经验库RAG/query_engine.py:588", "94-班外经验库RAG/query_engine.py:635", "94-班外经验库RAG/query_engine.py:2487", "94-班外经验库RAG/rag_tuning.py:119", "94-班外经验库RAG/rrm_chunker.py:69"]
tags: ["知识库RAG", "query_engine", "frontmatter", "RRM"]
related: ["ct-20261021-004", "idx-20261109-001"]
as_of: 2026-11-09
基准: repo_head 62267be
生成式: 否
核对: 2026-11-09（全锚本窗亲验·rrm_lint PASS 留痕）
失效触发: 任一锚行漂移（query_engine.py 参数面/返回面改码·server.mjs kbSearch/parseKbBlocks 改码）即过期改 draft 走 B1 保鲜闸
---

# ct-20261109-005 · kb_search Agent 工具契约（94 检索只读口）

> **口径**：本卡=契约实勘件（拍板#515 三件套之二·零码）——Agent 把 94 检索当工具调用所需的全部接口事实。上游依据=deepread 借鉴分析 §五~§七＋任务书-W413。续号=目录现有最大 ct-20261021-004 +1（B2 假设复验过·`ls 02-全景视图/04-接口与数据契约/`）。

## 〇 一句话

Agent 检索本仓知识=两个只读口（HTTP `GET /api/kb/search?q=` 与 CLI `query_engine.py --retrieve`），均零 LLM 纯检索——参数/返回/related 透出的全部实勘结论在本卡；契约建议与改码边界在§五申报。

## 一 调用口（两个·宁停语义）

| 口 | 形态 | 备注 |
|---|---|---|
| HTTP | `GET /api/kb/search?q=<query>`（monitor-web 8424 在役·只读·零预算） | q 必填·返回块 JSON |
| CLI | `.venv312 python query_engine.py --retrieve "<query>" [--scope=…｜--kind=…｜逃生口族]` | 94 区跑测一律 .venv312（拍板#121） |

- 路由：91-脚本/monitor-web/server.mjs:1153 收 `/api/kb/search` → kbSearch（91-脚本/monitor-web/server.mjs:646）spawn `.venv312 python query_engine.py --retrieve <q>` 子进程（纯检索不配电）。
- 宁停语义（不装绿）：q 缺=400·q 超 200 字=400·检索超时 120s 帽=504·子进程退码非零/python 或检索池缺=ok:false 如实报败带 stderr 尾。
- 块拼装出口=make_block（94-班外经验库RAG/query_engine.py:588）。

## 二 参数 schema（实勘）

| 参数 | 可用口 | 必填 | 值域与语义 |
|---|---|---|---|
| query（HTTP 名 q） | HTTP+CLI | 是 | 1-200 字·含 NUL 拒 |
| scope | 仅 CLI | 否 | `--scope`（空格/等号两形态）·值域 common｜app｜engine｜process（94-班外经验库RAG/query_engine.py:96）·缺省 dual=engine+common 双段·process 仅显式口（过程面五类件）·app 现网无源如实空手·非法值 ValueError 如实报（resolve_scope 94-班外经验库RAG/query_engine.py:277） |
| kind | 仅 CLI | 否 | `--kind=<值>` 等号形态（94-班外经验库RAG/query_engine.py:2487）·metadata kind EQ 硬过滤（模块卡等 RRM 枚举值·学习语料六类候拍）·KIND_BONUS 柔加权另属显式开关 |
| access | — | — | **占位候拍**：岗位授权展开 scope+filters 的映射件（§7.1 候拍接续）未落位·现役无此参数=不可用·勿传 |

- **HTTP 口现役只透 q**：91-脚本/monitor-web/server.mjs:1153 实勘只拼 q 单参——scope/kind 过滤走 CLI 口；HTTP 口要过滤候另票扩参（改码）。
- 逃生口族（CLI）：--no-rerank/--no-hybrid/--no-structure 等=回旧口径的显式口（缺省全开态·dual+rerank+结构通道编排）。

## 三 返回 schema（实勘·对任务书预记的修正）

- HTTP 返回 `{ok, q, count, blocks[], warnings[], zeroHit}`；**命中块={source, text} 两键**——parseKbBlocks（91-脚本/monitor-web/server.mjs:632）机械解析：`- 出处: …` 行=块头入 source·两空格缩进行=块体拼接入 text；`⚠️` 开头行=warnings（弱命中等如实转递·非命中）。
- source=出处行全量：`出处: <相对路径> · §<节名> · domain=… · scope=… · layer=…`＋legacy/FRESH_MARK 标（94-班外经验库RAG/query_engine.py:635）——非裸路径，§节名与层标随行。
- text=块体截断拼接·总预算 BLOCK_MAX_CHARS=1500（94-班外经验库RAG/query_engine.py:94·硬截断宁可少不可超）。
- **对任务书预记「top5 块+score+file_path+related」的实勘修正（B 级留痕候追认）**：块数=TOP_K 缺省 5（94-班外经验库RAG/rag_tuning.py:119·env TOP_K 每查询读可调）✓；**score 零透出**（精排归一分只在引擎内部排序·块头不带）；**file_path 由 source 字段承载**（出处行全量含 §节名层标）；**related 零透出**（§四实勘）。

## 四 related 透出实勘结论（B1 假设兑现）

**结论：库内有·输出无——related 现役不透出于检索输出。**三层证据（本窗亲验·复验命令留痕窗产物）：

1. **metadata 层有**：`_frontmatter_metadata` 把 frontmatter related（内部键 `_related` 列表）逗号串写入块 metadata（94-班外经验库RAG/rrm_chunker.py:69 `for lst in ("anchors", "tags", "related")`）——related 随块入 docstore。
2. **引擎输出层无**：`grep -n "related" query_engine.py` 零命中（本窗复验在案）；块头构造（94-班外经验库RAG/query_engine.py:635）只拼相对路径/§节名/domain/scope/layer/as_of(FRESH_MARK)/legacy 标——无 related 键。
3. **HTTP 层无**：parseKbBlocks 只拆 {source, text}（91-脚本/monitor-web/server.mjs:632）——related 未过任何一层出口。

**Agent 顺链替代路（零码可用）**：按命中块 source 路径打开源文件读其 frontmatter `related` 二跳找关联卡——此纪律已写入总览卡 idx-20261109-001 §三与提示词段（00-共享/Agent-RAG检索纪律提示词段.md）。若要输出透出 related/score=改 make_block 增键属改码——候另票（本票禁区·零码变更申报见§五）。

## 五 零码变更申报

本卡+项目卡（idx-20261109-001）+提示词段三件=纯新增文档；rrm_lint.py/rrm_chunker.py/query_engine.py/server.mjs/标签词表/队列写口零触碰。related 未透出=只写契约建议不改码（任务书禁区兑现·B1 假设以现行码实勘为准已兑现）。

## 时效

| 键 | 值 |
|---|---|
| 基准 | repo_head 62267be（2026-11-09 现勘·锚行全亲验） |
| 生成式 | 否 |
| 核对 | 2026-11-09（rrm_lint PASS·窗产物留痕） |
| 失效触发 | 见 frontmatter 失效触发键 |
