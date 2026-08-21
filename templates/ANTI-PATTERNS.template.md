# Anti-Pattern 目录（ANTI-PATTERNS）

> 本文件由 migrate-to-openclaw skill 注入,与 `AGENTS.md` 和 `PLATFORM-BASELINE.md` **并列生效**。
> **性质**:跨专家、跨平台、被实测证明的运行时 anti-pattern 与推荐替代做法。
> **冲突处理**:与专家正文 `AGENTS.md` 冲突时,以专家正文为准。

---

## Anti-Pattern 清单 

### 1. Structured Data Transfer · Pass via Files, Never via LLM Recitation

**Applicable scenario**: When you need to pass multi-line numbers, tabular data, CSV/JSON structures, or precise numerical lists across sessions/agents (especially when delegating tasks to sub-agents, or instructing another agent to generate scripts/reports/presentations).

**Anti-pattern**:

- ❌ Copy-pasting source data text (dozens of numbers, multi-column tables) into a prompt, expecting the receiver to reproduce every digit verbatim into code or deliverables
- ❌ Asking an LLM to "recite" or "embed" structured numerical values into script literals
- ❌ Trusting a sub-agent's self-check report as equivalent to deliverable correctness

**Why**: LLMs excel at semantic understanding but not at verbatim transfer of large volumes of precise numbers. Transformer attention mechanisms have precision boundaries for high-precision, low-redundancy, low-semantic numerical sequences. What appears to be "copied over" actually suffers substitution, doubling, decimal drift, and other hallucinations on critical digits.

**Recommended alternatives**:

- ✅ Have the receiver directly `read` the source file (CSV, JSON, Excel)
- ✅ If structured data must cross sessions, write it to a JSON/CSV file first; the receiver reads only the file
- ✅ When generating scripts, prefer the most readable data organization (e.g., Python literals, one variable per line) to minimize hallucination surface
- ✅ Prefer pre-installed toolchains over ones requiring ad-hoc installation (reduces environment uncertainty)

**QA iron rules**:

- Any deliverable involving precise numbers (tables, charts, presentation files, reports) **MUST** be verified with a text-extraction tool against source data
- Do not declare task complete until data consistency verification passes
- Sub-agent reporting "done" ≠ deliverable correctness; the main agent must independently verify key fields

---

### 1. 结构化数据传递 · 让文件传递,不让 LLM 复述

**适用场景**:当你需要把多行数字、表格数据、CSV/JSON 结构、精确数值列表跨会话/跨 agent 传递(尤其向子代理下达任务、或让另一 agent 生成脚本/报告/演示文件时)。

**Anti-pattern**:

- ❌ 把源数据文本(几十个数字、多列表格)复制粘贴进 prompt,期望接收方一字不差地写回代码或产物
- ❌ 让 LLM"复述"或"嵌入"结构化数值到脚本字面量
- ❌ 相信子代理的自检报告等价于产物正确

**Why**:LLM 擅长语义理解,不擅长逐字搬运大量精确数字。Transformer 注意力机制对高精度、无冗余、无语义的数值序列有精度边界。表面上看起来"复制过去了",实际会在关键数字上出现替换、翻倍、小数点漂移等幻觉。

**替代做法(推荐)**:

- ✅ 让接收方直接 `read` 源文件(CSV、JSON、Excel)
- ✅ 若必须跨会话传递结构化数据,先写成 JSON/CSV 文件,接收方只读文件
- ✅ 生成脚本时优先用可读性最高的数据组织方式(Python 字面量,一行一个变量),减少幻觉空间
- ✅ 优先选择已安装、无需临时安装的工具链(减少环境不确定性)

**QA 铁律**:

- 任何涉及精确数值的产出(表格、图表、演示文件、报告),**必须**用文本提取工具二次验证数字与源数据一致
- 未通过数据一致性验证前,不得宣布任务完成
- 子代理汇报"完成"≠产物正确,主 agent 需独立验证关键字段

---

### 2. Multi-Task Orchestration · Analyze Dependencies Before Execution

**Applicable scenario**: Any task involving multi-skill or multi-step invocation orchestration.

**Anti-pattern**:

- ❌ Launching all tasks in parallel without analyzing data dependencies between steps
- ❌ Skipping dependency analysis — not determining which steps can run in parallel, which must run sequentially, and which are aggregation endpoints
- ❌ Starting aggregation/summary steps before their dependency steps have completed

**Recommended alternatives**:

1. The main model **MUST first analyze the task list independently**, identifying input-output dependency relationships between steps (whose output is whose input)
2. Based on the analysis, construct the execution flow for the current task: which steps can run in parallel, which must run sequentially, which are aggregation endpoints
3. Aggregation/summary steps MUST wait for all dependency steps to complete, reading their output files as input
4. Never skip dependency analysis and launch all tasks in parallel

**Iron rule**: Process must be traceable; form must not overstep. Each task's dependency structure is derived by the main model from the current task's own logic — no universal fixed template exists, and rigid replication is forbidden.

---

### 2. 多任务编排 · 先自主分析依赖再执行

**适用场景**:任何涉及多技能/多步骤调用的任务编排。

**Anti-pattern**:

- ❌ 把所有任务一锅端并行,忽略步骤间的数据依赖
- ❌ 跳过依赖分析,不判断哪些步骤可并行、哪些必须串行、哪些是聚合终点
- ❌ 聚合/汇总类步骤未等依赖步骤完成就启动

**替代做法(推荐)**:

1. 主模型**必须先自行分析任务列表**,识别各步骤间的输入输出依赖关系(谁的产出是谁的输入)
2. 基于分析结果构建当前任务的执行流程:哪些可并行、哪些必须串行、哪些是聚合终点
3. 聚合/汇总类步骤必须等所有依赖步骤完成后启动,通过读取前序产出文件作为输入
4. 不得跳过依赖分析直接并行启动全部任务

**铁律**:过程可追溯,形态不越权。每次任务的依赖结构由主模型按当前任务自主推导,不存在通用固定模板,不得死板复制。

---

### 3. Numerical Processing · Unified Data Table + Script-Enforced Transfer

**Applicable scenario**: Any task involving numerical computation, numerical transfer, or cross-step numerical references.

**Anti-pattern**:

- ❌ Each step/sub-agent independently deciding numerical values, causing contradictions within the same task
- ❌ Transferring numbers via LLM recitation in prompts/context, causing precision loss or hallucination
- ❌ Building a data table but having the LLM read/write values manually instead of processing them via script — read/write by LLM still carries severe hallucination risk even with a table present

**Recommended alternatives**:

1. The main model or the first executing step MUST establish a **unified data table** (e.g., `data-table.json` or `data-table.md`) at task initiation
2. All subsequent steps that need to reference numerical values **MUST read from this table**; they must not decide values independently or fill them from memory
3. If subsequent steps produce new computed results, they MUST **write back** to this table, maintaining a single source of truth
4. Any number referenced by two or more steps (computed results, intermediate outputs, extracted source values) MUST be included in this table
5. **All numerical processing and transfer MUST be done via script** (Python, shell, etc.) — the LLM must not manually read values from the table and write them into outputs; a script must programmatically read the table and inject values into the target deliverable

**Iron rule**: Numerical single-source + script-enforced. Any number referenced by multiple steps must pass through the unified data table AND be transferred via script, never by LLM read/write recitation.

---

### 3. 数值处理 · 统一数值表 + 脚本强制传递

**适用场景**:任何涉及数值计算、数值传递、跨步骤引用数字的任务。

**Anti-pattern**:

- ❌ 各步骤/子代理独立决定数值,导致同一任务内数字互相矛盾
- ❌ 数值通过 LLM 在 prompt/上下文中复述传递,导致精度丢失或幻觉
- ❌ 建立了数值表但让 LLM 手动 read/write 数值而非用脚本处理——即使有表,LLM 手动读写依然有极强的幻觉风险

**替代做法(推荐)**:

1. 主模型或第一个执行步骤在任务启动时,建立一份**统一数值表**(如 `data-table.json` 或 `data-table.md`)
2. 所有后续步骤需要引用数值时,**必须从该表读取**,不得自行决定或凭记忆填写
3. 若后续步骤产生新的计算结果,须**回写**到该表,保持单一数据源
4. 凡被两个及以上步骤引用的数字(计算结果、中间产出、源数据提取值),都应纳入该表
5. **所有数值处理和传递必须通过脚本执行**(Python、shell 等)——不得由 LLM 手动从表中读取数值再写入产物;必须由脚本程序化地读取数值表并注入目标产物

**铁律**:数值单源 + 脚本强制。凡被多步骤引用的数字,必须经由统一数值表传递,且必须通过脚本执行,不得由 LLM read/write 复述。

---

## 附:与 PLATFORM-BASELINE 的关系

- **PLATFORM-BASELINE.md**:补齐 OpenClaw 平台相对 源平台 的元行为缺失(工作记忆、最终回复自足、Skill routing、Deliverable rendering 等),条目多为 BLOCKING REQUIREMENT。
- **ANTI-PATTERNS.md**(本文件):记录跨专家共性 anti-pattern,条目为推荐/铁律但按条目粒度自适配。
- 两文件独立
