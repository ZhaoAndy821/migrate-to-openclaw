# migrate-to-openclaw

> **EN** — A migration executor that ports expert assets from non-standard platforms into the standard OpenClaw workspace format.
>
> **zh** — 把非标格式平台的专家资产，迁移为标准 OpenClaw 工作区格式的迁移执行器。

> **Migration Direction / 迁移方向**
>
> **EN** — From **non-standard formats** to the **standard OpenClaw format** (AGENTS.md + skills/ + skeleton components, the open-source Agent workspace convention).
>
> **zh** — 从**非标格式**迁移到**标准 OpenClaw 格式**（AGENTS.md + skills/ + 骨架件，开源 Agent 工作区规范）。

---

## Core Principles / 核心原则

**EN**

1. **Faithful porting over restructuring** — copy the source body, prompts, and references as close to 1:1 as possible; structural adaptation comes second.
2. **"Must-produce" limited to explicit declarations** — only items explicitly mandated by the source documents become required outputs.
3. **Leave a trace in files; keep the chat clean** — self-checks and compliance requirements are persisted to disk, never as chat-layer checkmark theater.

**zh**

1. **忠实搬运优于结构改造**——正文、提示词、reference 尽最大可能 1:1 复制，结构改造其次。
2. **"必产"只限显式声明**——源文档硬性措辞明确列出的条款，才转为必产。
3. **过程有痕，对话不喧宾夺主**——自检与合规要求落盘到文件，不做对话层的打勾式输出。

---

## Migration Flow / 迁移流程（8 步）

**EN**

Step 0 Resolve source package & target workspace → Step 1 Read through & extract explicit declarations → Step 2 Decide skill mounting strategy → Step 3 Faithful porting (shell copy + SHA-256 verification) → Step 4 Two-layer gap scan (skill-level + reference-level) → Step 5 Persist self-check → Step 6 Assemble agent main file (skeleton completion + patch injection + live verification) → Step 7 Produce migration notes → Step 8 Completion notice.

**zh**

Step 0 源包/目标解析 → Step 1 通读并抽取显式声明 → Step 2 决定 skill 挂载策略 → Step 3 忠实搬运（shell 复制 + SHA-256 校验）→ Step 4 双层缺口扫描（skill 级 + reference 级）→ Step 5 过程自检落盘 → Step 6 组装 agent 主文件（骨架补建 + 补丁注入 + 实测确认）→ Step 7 产出迁移记录 → Step 8 完工通告。

---

## Structure / 结构

```
migrate-to-openclaw/
├── SKILL.md                            # Migration executor (main workflow)
├── templates/
│   ├── PLATFORM-BASELINE.template.md   # Target-platform behavioral baseline patches
│   ├── ANTI-PATTERNS.template.md       # Cross-expert anti-pattern checklist
│   └── MIGRATION-NOTES.template.md     # Migration record template
└── LICENSE                             # CC0 1.0 Universal
```

---

## Core Mechanisms / 核心机制

**EN**

- **Two-layer gap scan** — skill-level (missing directories) + reference-level (broken internal links / relative-path references inside SKILL.md); surfaces hidden gaps that would otherwise only appear at runtime, moving them earlier into the porting phase.
- **Skeleton completion** — target workspace structural default files are mandatory and complete; content is neither split nor filled in.
- **Patch injection** — platform differences are materialized as patch files (reachable via a guidance block for non-whitelisted files), never injected into the expert's own prompt.
- **Live verification** — patch visibility is tested on a real run: a hash only proves bytes arrived, not that they took effect.

**zh**

- **双层缺口扫描**——skill 级（目录缺失）+ reference 级（SKILL.md 内部失效链接 / 相对路径引用缺失），把原本要到运行时才暴露的隐性缺口，提前到搬运阶段处理。
- **骨架补建**——目标平台工作区结构性默认文件必须齐全；内容不拆分、不填充。
- **补丁注入**——平台差异显式落成补丁文件（非白名单文件经引导块触达），不侵入专家正文。
- **实测确认**——补丁可见性在真实运行中验证：hash 只证明字节到位，不证明生效。

---

## Design Trade-offs / 设计取舍

**EN**

- Migration is not rewriting: what ships inside the package travels with it; what lives on the platform stays there — find the equivalent on the target.
- Iteration is incident-driven: every change is anchored to a triggering incident (data hallucination → data travels via files; silent patch failure → live verification).

**zh**

- 迁移不是重写：装在包里的跟着走；留在平台里的原地留——去目标平台找等价物。
- 迭代由事故驱动：每次变化都挂在触发事故上（数据幻觉 → 数据走文件；补丁静默失效 → 实测确认）。

---

## License / 许可

[CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/) — see `LICENSE`.

> 注：本仓库内容为实习期间产出，已确认 IP 归属与公开许可。
> Note: Content produced during an internship; IP ownership and publication permission confirmed.
