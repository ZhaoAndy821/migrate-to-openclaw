---
name: migrate-to-openclaw
description: 把源平台（非标格式）专家包搬到 OpenClaw(开源 agent 工作区格式)workspace 的迁移执行器。原则:忠实搬运优于结构改造,"必产"只限源文档显式声明,过程可追溯但形态不越权。含平台基线约束补丁机制(Step 6.5)。
---

# 专家迁移：源平台 → OpenClaw 开源格式

## 何时使用

用户下达 "把 <expert> 从源平台搬到 OpenClaw"、"迁移专家 <expert>"、"跨平台翻译 <expert>" 一类指令时使用本 skill。

## 顶层三条铁律

1. **搬运优于改造**:正文/提示词/reference 尽最大可能 1:1 复制粘贴。结构改造只在 OC 平台协议兼容必需时执行,每一处必须交代。
2. **必产限于显式声明**:**只**允许把源文档中出现"必须/应当/须/一定要/禁止/不得/严禁/避免/不允许"等硬性措辞、或结构标签明确列出的条款转为"必产"。其他一切归模型自主发挥,不做强推。
3. **过程有痕,chat 不喧宾夺主**:源文档若要求过程自检、格式合规、防偏移等,落盘到迁后 agent workspace 的 memory 目录;**不**在 chat 层做打勾式、勾选式、数值化通过判定式的输出。
4. **非白名单文件须显式引导**:OC 平台只自动注入固定 8 个 basename(`AGENTS.md`/`SOUL.md`/`TOOLS.md`/`IDENTITY.md`/`USER.md`/`HEARTBEAT.md`/`BOOTSTRAP.md`/`MEMORY.md`)。凡注入目标 workspace 的其他文件名,**必**在白名单文件内写入显式读取引导;否则该文件不进 system prompt,磁盘有文件而 agent 永不可见,等同未注入。

## 术语

| 术语 | 含义 |
|------|------|
| 源包 | 待迁源平台专家包根目录,含 `agents/<expert>.md`、`skills/*/SKILL.md`、`references/`、`plugin.json` 等 |
| 目标 workspace | OC 侧承接该专家的 workspace 目录,命名约定 `workspace-<expert>` |
| 执行者 workspace | 运行本 skill 的那个 agent 自己所在的 workspace 根目录(本 skill 挂在其 `skills/migrate-to-openclaw/` 下)。名称由该 agent 自定,不做假设 |
| 显式声明 | 源文档中用硬性措辞或结构标签明确写出的规则 |
| 推测性补全 | 目标 workspace 产出中存在、但源文档未显式声明的行为/产物 |
| MIGRATION-NOTES | 每次迁移完成的交付简报,落到 `<执行者 workspace>/migration-notes/<expert>-R<n>.md` |

## 迁移流程(8 步)

### Step 0 · 源包与目标解析

**源包路径**:
- 用户在触发指令中指定 → 直接采用
- 未指定 → **主动去找**:在常见迁移导出位置(如 `<源平台市场>/plugins/`、用户 `Downloads` 或 `Documents` 目录)搜同名 `<expert>` 目录
- 找到 → 在 `MIGRATION-NOTES.md` 里明确交代找到的路径与查找依据
- 找到 >1 个同名目录 → **不得**自行择一。须向用户列出全部候选(含完整路径、最后修改时间、目录大小),由用户指定后再继续
- 找不到 → 停手,向用户询问源包位置

**目标 workspace 路径**:
- 默认解析规则:**同 gateway 下**同名 `workspace-<expert>` 目录(相对执行者 workspace 的父级)
- 若目标 workspace 不存在 → 走 OC 平台的新建 agent 标准流程创建
- 用户可显式指定其他路径覆盖默认

### Step 1 · 通读源包 + 抽显式声明清单

按顺序读:

1. `<源包>/plugin.json`(判断 skill 挂载模式)
2. `<源包>/agents/<expert>.md`(agent 主文件)
3. `<源包>/skills/*/SKILL.md`(每个 skill 的定义)
4. `<源包>/references/*` 和各 skill 下 `references/*`(浏览一遍即可)

从 agent 主文件和 SKILL.md 中抽出**四类显式声明清单**:

| 类别 | 判定依据 |
|------|---------|
| **Must-Do** | "必须/应当/须/一定要" + 动作 |
| **Must-Deliver** | "必须产出/必须输出/必须交付" + 内容形态(含 chat/落盘/memory) |
| **Must-Not-Do** | "禁止/不得/不该/严禁/避免/不允许" + 动作 |
| **Must-Not-Trigger** | 禁止触发的模式(如寒暄、角色扮演等) |

**是**:硬性措辞、结构标签(`### 输出要求`、`### 禁忌`、`<forbidden>`、`<required>`)、明确列表化
**否**:描述性语句(通常/一般/倾向于/可以)、举例、隐含期待

清单每条要标注**出处**(文件路径+段落/行号),供下游校验。

### Step 2 · skill 挂载策略判断

读 `<源包>/plugin.json` 的 `skills[]`:

- **长度 = 0** → **纯 agent 型**:无技能包。专家仅依 agent 主文件人格与提示词工作。目标 workspace 不建 `skills/` 目录(若 Step 6.7 骨架衍生可留空壳);MIGRATION-NOTES §1 备注"纯 agent 型,无技能包"
- **长度 = 1** → **条件触发型**:SKILL.md 描述该条件下需触发的行为,agent 主文件只保留核心人格;skill 单独挂在 `<目标 workspace>/skills/<skill-name>/`
- **长度 > 1** → **常驻型**:多技能协同,全部搬到 `<目标 workspace>/skills/<skill-name-N>/`,agent 主文件通过 skill mounting 声明

### Step 3 · 忠实搬运

**搬运纪律**:一切正文/提示词/reference 类文件的复制粘贴,**必须用文件系统 shell 命令执行**(如 `Copy-Item` / `cp`),**禁止**由模型 read → write 逐个搬--模型 write 存在字节偏移风险,即使字节数一致也不保证内容一致。

**Hash 校验**:每一份搬运的文件,落盘后计算源与目标 SHA-256 或 MD5 比对;不一致即视为搬运失败,必须重搬。所有搬运结果记入 `MIGRATION-NOTES.md` §4 的"逐文件 hash 校验表"。

**搬运范围**:

1. **agent 主文件**:
   - `<源包>/agents/<expert>.md` → `<目标 workspace>/AGENTS.md`(或 `<目标 workspace>/<expert>.md`,视 OC agent 结构约定)
   - 正文语言按源保留(源为中文即中文,源为英文即英文,源含英文声明段落也保留)
   - 结构改造只在 OC 平台协议兼容必需时执行(如 XML 标签 → markdown 章节),每一处记入 `MIGRATION-NOTES.md`
2. **skills/**:
   - 目录数、SKILL.md、`references/`、`templates/`、`assets/` **完整搬**
   - 每个 SKILL.md 目标要求 hash 一致
3. **references/**:
   - 文件 1:1 存在于 `<目标 workspace>/references/`

**不搬清单**:

- **头像 / avatar 类视觉资产**(`avatars/*.png` 等)--头像属运行时视觉资产,与专家的提示词与技能内容无关,跨设备/跨平台易踩权限坑,**默认不搬**;源包 `plugin.json` 明示需要、或用户明确指定时才搬,并在 `MIGRATION-NOTES.md` §9 登记
- **平台私有元数据**(如 <源平台运行时标记>、<源平台元数据目录> 中的时间戳/来源标记类文件,非规则性元数据)--不影响专家行为,跳过
- **平台 plugin.json 中的展示元信息**(displayName / profession / defaultInitPrompt / quickPrompts / tags 等)如果源专家在 chat 层需要用到,转成 Markdown 归档一份到 `<目标 workspace>/PLUGIN-META.md`;否则跳过

### Step 4 · 双层缺口清单

**Step 4 分两层扫描依赖完备性**:

#### Step 4.1 · skill 级缺口

若 agent 主文件声明用到但 `<源包>/skills/` 目录下**本身就没有**的 skill:

- 生成缺口清单
- **主动询问用户**:"这个 skill 是我去找,还是你提供?"
- 用户决策后 → 写入 `MIGRATION-NOTES.md` §7.1
- **不擅自**搜索、创建、猜测该 skill 内容

#### Step 4.2 · reference 级缺口

**对每份已搬运的 SKILL.md 扫描其内部引用,逐一验证引用文件是否存在于源包内**。

扫描模式(至少覆盖以下形式):

| # | 引用形式 | 例 | 扫描正则(powershell / grep) |
|---|--------|-----|-------|
| 1 | Markdown 内链 | `[editing.md](editing.md)` 、 `[schema](references/schema.md)` | `\[[^\]]+\]\(([^)]+\.(md\|json\|yaml\|py\|js\|sh\|template))\)` |
| 2 | 直接提及相对路径 | "Read editing.md for details"、"see references/pptxgenjs.md" | `[Rr]ead\s+([a-z][\w./-]+\.md)` 、 `(?:references\|templates\|assets\|scripts)/[\w./-]+` |
| 3 | 代码块引用归位 | `include("references/foo.md")` | 根据实际情况自适应 |

**处置**:

- 引用存在 → 验证已搬到目标对应位置(hash 校验)
- **引用不存在**→列入 `MIGRATION-NOTES.md` §7.2 **reference 级缺口**,同时启动**官方包历兵库兑底查找**:

  1. **查官方统一技能包库**:源平台存在一个官方插件市场(marketplace)存放可复用的官方 skill,把缺失的 reference 文件名 / 所属 skill 名当关键词,在官方包库中搜同名或同名根目录。

     - 典型位置形式:从 `<源平台官方技能包库>` 下存在平行于 experts 的官方包名字空间(具体层级以本机实际为准)
     - 使用时**不硬编路径**:先搜索存在性→实际相对位置交给搬运命令

     - 搜到 → 在 MIGRATION-NOTES.md §7.2 对应行补上"不带硬路径的补完提示"(如"官方包库 `<official-marketplace>/plugins/<skill-name>/` 中可找到 skill 包 X")
     - **搜不到** → 保持"主动找 / 你提供 / 忽略"三选交用户

  2. **合并方式判断**:

     - 若 SKILL.md 内使用相对路径引用(如 `[editing.md](editing.md)` 、 `python scripts/xxx.py`)→ 官方包除 SKILL.md 外的内容**合并到**当前 skill 目录,保留专家包自带的 SKILL.md(因两侧 SKILL.md 可能有微调)
     - 若引用使用绝对或已命名命名空间路径 → 官方包作为独立 skill 挂载(保留其自带 SKILL.md)

  3. **搬运仍用 shell + hash 逐文件校验**(Step 3 铁律不变)

- **不擅自**创建、猜测、从专家描述内推导 reference 内容;官方包兑底查不到时,回到"主动询问用户"铁律

**例(扫描脚本参考)**:

```powershell
# 无代码块前提:仅抽 4.2 扫描的 markdown 内链一类,实际拉全时需包含上表中多种模式
$root = "<源包>/skills"
Get-ChildItem $root -Recurse -Filter "SKILL.md" | ForEach-Object {
  $skill = $_.Directory.Name
  $content = Get-Content $_.FullName -Raw
  $matches = [regex]::Matches($content, '\[([^\]]+)\]\(([^)]+\.md)\)')
  foreach ($m in $matches) {
    $target = $m.Groups[2].Value
    $exists = Test-Path (Join-Path $_.DirectoryName $target)
    [PSCustomObject]@{ Skill=$skill; Ref=$target; Exists=$exists }
  }
} | Where-Object { -not $_.Exists }
```

**为什么双层**:仅扫 skill 目录只能拓 skill 本身是否存在,但 skill 内部声明的 reference 依赖(模板、脚本、参考文档)若源包本就没包括,会在专家运行时造成"运行时才发现的隐性缺口"--专家会在 chat 中声明"the skill references external docs that aren't in our workspace"。Step 4.2 就是把这类隐性缺口提前到搬运阶段目见。

### Step 5 · 过程自检类要求落盘化

源文档若声明"必须自检 / 必须评估 / 必须验证"但**未声明"必须输出自检结果"**:

- ❌ 不在 chat 层做打勾式、勾选式、数值化通过判定式的输出
- ✅ 落盘到 `<目标 workspace>/memory/`(如 `memory/self-check-YYYY-MM-DD.md`)
- ✅ chat 层允许一句自然的话交代自检完成(无固定模板)

若源文档**显式声明了自检输出形式**(含具体模板),才按源文档形式在 chat 层输出--但此情形必须能在显式声明清单里找到对应条款。

### Step 6 · 组装 agent 主文件

原则:

- 忠实保留正文;结构调整仅出于 OC 平台协议兼容
- Must-Do / Must-Deliver 条款以**源文档原文措辞**呈现
- Must-Not-Do / Must-Not-Trigger 条款以**清晰、去感叹号、去恐吓化**的语气呈现
- workspace 命名、memory 位置等 OC 平台约定单独一节说明,不与源文档正文混杂

### Step 6.5 · 注入平台基线约束补丁

OpenClaw 平台相对源平台缺少一些通用元行为(如工作记忆归档、最终回复自足等)。本步骤把这些已知缺口以**与专家正文并列**的形式注入目标 workspace,不侵入专家提示词本身。

**操作**:把本 skill 自带的 `templates/PLATFORM-BASELINE.template.md` 复制一份到 `<目标 workspace>/PLATFORM-BASELINE.md`(同 Step 3 搬运铁律:文件系统 shell 命令 + hash 校验)。

**原则**:

- 模板内容不得删改或精简(版本另在 skill 侧统一控制,目标 workspace 不得自行定制)
- 若实际需要专家级定制(例如某专家声明不写 memory),写进 `MIGRATION-NOTES.md` §9 已知偏离项,以便后续回溯
- 补丁本身不含死框架(无 fenced code 必产块、无数值化通过判定、无打勾式 checklist),与顶层铁律 3 一致
- `PLATFORM-BASELINE.md` **不随 system prompt 自动加载**(平台白名单硬编码,见顶层铁律 4),须经 Step 6.8 写入 `TOOLS.md` 的引导块方能生效;内容冲突时以 `AGENTS.md`(专家正文)为准

**登记**：在 `MIGRATION-NOTES.md` §8 新增内容清单里登记本次注入的平台基线补丁

### Step 6.6 · 注入Anti-Pattern 补丁

除了平台基线（Step 6.5）之外，还存在一类 **跨专家、跨平台、被实测证明的“Anti-Pattern 目录”**（如结构化数据在与子代理传递时的幻觉模式），它们不属于“平台能力缺失”但会重复伤专家。本步骤把这些教训以与平台基线同样的**与专家正文并列**形式注入目标 workspace。

**操作**：把本 skill 自带的 `templates/ANTI-PATTERNS.template.md` 复制一份到 `<目标 workspace>/ANTI-PATTERNS.md`（同 Step 3 搬运铁律：shell + hash）。

**与 Step 6.5 的边界划分**：

| 维度 | Step 6.5 PLATFORM-BASELINE | Step 6.6 ANTI-PATTERNS |
|------|--------------------------|--------------------------|
| 性质 | 平台能力缺失补齐 | 跨专家实测验证的 anti-pattern 目录 |
| 措辞 | BLOCKING REQUIREMENT | RECOMMENDED / 铁律（逐条灵活） |
| 修改频率 | 低（平台行为改变才动） | 高（每次新发现 anti-pattern 时追加） |
| 条目来源 | 平台对比分析 | 实测/专家自检/benchmark |

**原则**：

- 模板内容由 skill 一块控，目标 workspace 不得自行定制（修改回流 skill）
- **目前收录的 anti-pattern（往后累积）**：“不要让 LLM 传递结构化数据，让文件传递”——适用于任何需要跨 agent 传递多行数字 / 结构化数据的场景
- **与专家正文冲突时**，以专家正文为准（同 Step 6.5 原则）
- 无需逐条充当 BLOCKING；每条自带适用范围与措辞强弱
- 不写死框架、数值化通过判定、打勾式 checklist（与顶层铁律 3 一致）

**登记**：在 `MIGRATION-NOTES.md` §8 新增内容清单里登记本次注入的Anti-Pattern 条目数。

### Step 6.7 · OC 侧骨架完整性补建(强制)

目标 workspace 必须具备 OC 平台标准骨架,**结构必全**。`TOOLS.md` 必须由 Step 6.8 写入两条补丁引导块;除此之外,清单里没让填的一律不填:

**骨架清单(必全)**:

| 项 | 内容规范 |
|----|---------|
| `IDENTITY.md` | 空卡脚手架(平台默认模板,不填) |
| `SOUL.md` | 留空——人格全部由 `AGENTS.md` 独承,本技能不向骨架文件拆分或注入任何人格内容;后续由谁填写非本技能关切 |
| `USER.md` | 空档案脚手架(平台默认模板,不填) |
| `MEMORY.md` | 占位标题 |
| `HEARTBEAT.md` | 平台默认空模板 |
| `TOOLS.md` | 脚手架 + **必填**:由 Step 6.8 写入平台补丁引导块 |
| `memory/` | 目录(缺失则建) |

**原则**:

- 专家正文(人格/语气/身份/规则)**只存在于 `AGENTS.md`,1:1 于源**,永不拆入骨架文件
- 骨架文件已存在时不覆写(平台生成的默认内容保留原样)
- 补建清单登记 `MIGRATION-NOTES.md` §8 新增内容清单
- 骨架之外为测试便利额外添加的目录(非 OC 标准骨架、非本 skill 要求)不属本步,由执行者按工作需要自行处理并在 notes 中注明性质

### Step 6.8 · TOOLS.md 补丁引导块注入(强制)

Step 6.5 / 6.6 注入的两份补丁文件不在平台自动加载白名单内(顶层铁律 4)。本步骤在目标 `TOOLS.md` 内写入引导块,使其可见。

**为何写 `TOOLS.md` 而非 `AGENTS.md`**:`AGENTS.md` 受 Step 3 的 1:1 搬运与 hash 校验约束,追加任何内容都会破坏 hash 记录、违顶层铁律 1。`TOOLS.md` 为 Step 6.7 建立的脚手架,本无源可对,填之不破任何搬运约束,且语义上正属平台/工具侧注记,非专家正文,不触 L241「专家正文只存在于 AGENTS.md」。

**操作**:在 `<目标 workspace>/TOOLS.md` 内写入:

```markdown
## 平台补丁 · 强制读取

本 workspace 根目录下两份补丁文件不随 system prompt 自动加载,
须在会话开始、执行任务前主动读取:

- `PLATFORM-BASELINE.md` —— 平台能力缺口补齐
- `ANTI-PATTERNS.md` —— 跨专家实测 anti-pattern 目录

两者与 `AGENTS.md`(专家正文)冲突时,以专家正文为准。
```

**原则**:

- 引导块只放**指针**,不得搬入补丁全文——放指针不算侵入专家正文,放全文才算
- 若某专家未注入其中一份补丁,引导块相应删去该行
- 本步骤所填内容即 Step 6.7 清单中 `TOOLS.md` 的必填项

**登记**:在 `MIGRATION-NOTES.md` §8 登记引导块已注入。

### Step 6.9 · 补丁可见性实测(强制)

**不实测不算完成**。本步骤存在的理由:v1.4 及以前的 Step 6.5/6.6 正因未实测机制假设,鬼影运行数轮而不觉——hash 校验全绿,但两份补丁从未进入任何 agent 的 system prompt。**hash 只证字节到位,不证生效**。

**操作**:迁移完成后启动一次目标 agent,确认两条:

1. `TOOLS.md` 引导块已进 system prompt(白名单文件,应可见)
2. agent 能据引导块主动 read 两份补丁文件

**判定**:两条皆确认方为通过。任一不达,回 Step 6.8 复查。

**登记**:实测结果记入 `MIGRATION-NOTES.md` §8,含实测时间与观察到的实际行为。**不得以「应该可以」「理论上生效」代替实测记录**。

### Step 6.10 · 迁移过程日志落盘

本 skill 每轮迁移的过程日志固定落于:

```
logs/migrations/<expert>-R<n>-<timestamp>.log
```

`<n>` 为迁移轮次,`<timestamp>` 取 `YYYYMMDD-HHmmss`。

日志记**过程**,`MIGRATION-NOTES.md` 记**结论**,二者不混。

### Step 7 · 产出 MIGRATION-NOTES

落到 `<执行者 workspace>/migration-notes/<expert>-R<n>.md`,格式见 `templates/MIGRATION-NOTES.template.md`。

必含段落:

1. 迁移日期与轮次、触发指令
2. 源包与目标(实际路径 + 找到依据 / 新建操作)
3. 显式声明清单(四类)+ 每条出处
4. 搬运结果统计(**含逐文件 hash 校验表**)
5. 结构改造点清单(每处 + 理由 + 影响面)
6. 缺失清单(源侧有但目标侧未搬,含理由)
7. skill 缺口清单(用户决策记录)
8. 新增内容清单(目标侧有但源侧无,非结构改造)
9. 已知偏离项

### Step 8 · 完工通告

向用户复述:

- 目标 workspace 路径
- MIGRATION-NOTES 归档路径


## workspace 目录约定

迁后 `<目标 workspace>` 结构。**OC 平台标准骨架六件 + memory/ 为强制项(见 Step 6.7),其余按迁移内容落位**:

```
<目标 workspace>/
├── AGENTS.md               # 主 agent 文件(专家正文,1:1 于源,人格不外拆)
├── PLATFORM-BASELINE.md    # 平台基线约束补丁(skill 注入,与 AGENTS.md 并列)
├── ANTI-PATTERNS.md        # Anti-Pattern 补丁(skill 注入,与 AGENTS.md 并列)
├── IDENTITY.md             # 骨架(空卡脚手架,强制)
├── SOUL.md                 # 骨架(留空,强制)
├── USER.md                 # 骨架(空档案脚手架,强制)
├── MEMORY.md               # 骨架(占位标题,强制)
├── HEARTBEAT.md            # 骨架(平台默认空模板,强制)
├── TOOLS.md                # 骨架(空脚手架,强制)
├── PLUGIN-META.md          # 平台展示元信息(仅当 chat 层确需源 plugin.json 展示信息)
├── skills/
│   ├── <skill-name>/
│   │   ├── SKILL.md
│   │   ├── references/
│   │   └── templates/
│   └── references/         # skill 层共享参考(源包有则搬)
├── references/             # 顶层参考(源包有则搬)
└── memory/                 # 目录(强制)
```

## 边界与红线

- **不评价源专家的产物质量**:本 skill 只管搬运和守约,不承担"目标 workspace 跑出来的东西好不好"的责任--那是下游评分体系的事
- **不擅自"优化"源文档**:源文档措辞有明显问题时,忠实搬运并在 MIGRATION-NOTES 里标注建议,不擅改
- **不带具体实现假设**:本 skill 属于【执行者 → 专家】中间的搬运层,对两侧具体环境(机器、目录布局、执行者 agent 名称)保持不假设,只运行时结合用户指定与默认解析规则进行路径解析

## 关联文件

- MIGRATION-NOTES 存档目录:`<执行者 workspace>/migration-notes/`
- MIGRATION-NOTES 模板:`skills/migrate-to-openclaw/templates/MIGRATION-NOTES.template.md`
- 平台基线补丁模板:`skills/migrate-to-openclaw/templates/PLATFORM-BASELINE.template.md`
- 迁移过程日志目录:`logs/migrations/`(见 Step 6.10)

## MIGRATION-NOTES 轻量版

完整模板 147 行 11 章节。高频迁移场景可启用轻量版,只保留:

- §1 基本信息
- §2 源包清单
- §4.1 hash 校验表
- §9 已知偏离项

**启用条件**:同一专家第二轮及以后,且源包结构无改动。首轮迁移、或源包结构有变者,一律用完整版。
