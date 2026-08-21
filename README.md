# migrate-to-openclaw

把非标格式平台的专家资产，迁移为标准 OpenClaw 格式的迁移执行器。

> **迁移方向**：从**非标格式**迁移到**标准 OpenClaw 格式**（AGENTS.md + skills/ + 骨架件，开源 Agent 工作区规范）。

## 核心原则

1. **忠实搬运优先**：正文/提示词/reference 尽最大可能 1:1 复制，结构改造其次
2. **必产只限显式声明**：源文档硬性措辞明确列出的条款才转为必产
3. **过程可追溯、形态不越权**：自检/合规类要求落盘到文件，不做 chat 层打勾式输出

## 迁移流程（8 步）

Step 0 源包/目标解析 → Step 1 通读+抽显式声明 → Step 2 skill 挂载策略 → Step 3 忠实搬运（shell+hash 校验）→ Step 4 双层缺口扫描（skill 级 + reference 级）→ Step 5 过程自检落盘 → Step 6 组装 agent 主文件（骨架补建 + 补丁注入 + 实测确认）→ Step 7 产出迁移记录 → Step 8 完工通告

## 结构

```
migrate-to-openclaw/
├── SKILL.md                            # 迁移执行器（主流程）
├── templates/
│   ├── PLATFORM-BASELINE.template.md   # 目标平台行为基线补丁（元行为缺口补齐）
│   ├── ANTI-PATTERNS.template.md       # 跨专家反模式清单（数据走文件/依赖分析/数值单源）
│   └── MIGRATION-NOTES.template.md     # 迁移记录模板（缺口/补丁/决策登记）
└── archive/                            # 历史打包
```

## 核心机制

- **双层缺口扫描**：skill 级（目录缺失）+ reference 级（SKILL.md 内链/相对路径引用缺失），把运行时才暴露的隐性缺口提前到搬运阶段
- **骨架补建**：目标平台 workspace 结构性默认文件必全，内容不拆不填
- **补丁注入**：平台差异显式成补丁文件（非白名单文件经引导块触达），不侵入专家正文
- **实测确认**：补丁可见性实测——hash 只证字节到位，不证生效

## 设计取舍

- 迁移不是重写：机器在包里跟着走，在平台里的原地留、去新平台找等价物
- 事故驱动迭代：每轮变化挂在触发事故上（数据幻觉→数据走文件；补丁静默失效→实测确认）

## License

[CC0 1.0 Universal](https://creativecommons.org/publicdomain/zero/1.0/)（公域贡献——详见 LICENSE 文件）。

> 注：本仓库内容为实习期间产出，发布前请确认 IP 归属与公开许可。
