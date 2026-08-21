# 发布检查清单 · migrate-to-openclaw

> 逐项核对，全部 ☑ 后再推 GitHub。审阅人可在此文件上直接勾选/批注。

## A. 合规红线（最高优先）

- [ ] **IP 归属确认**：已获导师/公司明确许可，本技能可公开开源（未确认前**不要**推送）
- [ ] 确认公开范围不违反实习/劳动合同条款

## B. 敏感信息复查（仓库内）

- [ ] 无封闭产品名残留（审阅人按已知源平台真实名自查）——示例：
  ```bash
  grep -rn "<源平台真实名>\|<源平台目录名>" . --include="*.md" --include="LICENSE" --include=".gitignore" 2>/dev/null
  ```
- [ ] 无平台机制名（源平台系统提示词段落名）
- [ ] 无内部路径残留（平台私有运行时目录 / 绝对路径 `盘符:\` 或 `/<user>/`）
- [ ] 无密钥/token/password 残留
- [ ] 无导师/评审/对话引用

## C. 命名与一致性

- [ ] 仓库名 `migrate-to-openclaw` 定稿
- [ ] SKILL.md `name:` 字段 = `migrate-to-openclaw`
- [ ] 全包无旧名残留（`agent-skill-porting` / `to-openclaw-agent` / `wb-oc-migration`）——复查：
  ```bash
  grep -rn "agent-skill-porting\|to-openclaw-agent\|wb-oc-migration" . 2>/dev/null
  ```

## D. 仓库内容完整性

- [ ] `README.md`：简介 / 迁移方向声明 / 结构 / 核心机制 / 设计取舍 齐全
- [ ] `LICENSE`：CC0 1.0 全文在位
- [ ] `SKILL.md`：frontmatter + 8 步流程 + 铁律完整
- [ ] `templates/`：PLATFORM-BASELINE / ANTI-PATTERNS / MIGRATION-NOTES 三件在位
- [ ] `.gitignore` 生效（`git status` 无敏感/临时文件混入）

## E. 发布动作

- [ ] 初始 commit 已创建（信息清晰）
- [ ] GitHub 新建公开仓库 `migrate-to-openclaw`
- [ ] 推送完成
- [ ] 仓库描述与 topic 已填（如：agent skills / openclaw / migration / loop-engineering）

## F. 发布后

- [ ] 用浏览器打开 GitHub 页面，肉眼复查 README 渲染与文件列表
- [ ] 保留本地工作副本，删除/归档敏感原始材料
