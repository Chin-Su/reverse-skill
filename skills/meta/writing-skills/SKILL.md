---
name: writing-skills
description: Use when creating a new skill in this repository or upgrading an existing SKILL.md — writing the frontmatter description, deciding what goes in SKILL.md versus references/, and adding the required anatomy (ACTION REQUIRED, Rationalizations, Red Flags, completion self-check).
---

# 编写与升级 Skill（Writing Skills）

> 元流程 skill。接入 bootstrap / 路由 / Kali 同步的机械步骤见 `../../CONTRIBUTING.md`，本文件只讲「怎么把 skill 写得会被正确触发、会被执行」。

## ACTION REQUIRED（读完后立刻执行）

1. `NOW`: 确认是新建还是升级；新建先按 `../../CONTRIBUTING.md` §1 判断是否真的需要新 skill。
2. `NEXT`: 先写 frontmatter `description`，再写正文。
3. `ACT`: 按下方 anatomy 补齐章节，然后跑「验证」一节的命令。

## 1. Frontmatter

```yaml
---
name: <dir-name>            # 与目录名一致，小写 + 连字符
description: Use when <触发场景/信号>. <可选：一句话说明边界或首选工具>
---
```

- `description` 是**触发条件**，不是流程摘要：写「什么时候用」，不要写「先做 A 再做 B」。模型只看 description 决定是否加载。
- 包含用户会说的关键词（中英均可），例如 `APK`、`jadx`、`加固`、`hook`。
- 只用 `name` / `description` 两个字段（Agent Skills 规范）；其他元数据放正文。

## 2. 渐进披露（Progressive Disclosure）

- `SKILL.md` = 入口 + 流程 + 门闩，目标 ≤ ~400 行。
- 长清单、命令速查、payload、阶段细节 → `references/<topic>.md`，在 SKILL.md 用一行链接并说明「何时读取」。
- 可执行的规则优先做成 `scripts/` 中的确定性脚本，而不是散文。

## 3. 必需 anatomy（顺序建议）

1. `## ACTION REQUIRED（读完后立刻执行）` — 3-6 步，含 precedent 读取、tool-index、bootstrap，以及：
   - 工具/脚本失败 → `../meta/systematic-debugging/SKILL.md`
   - 写/改代码 → `../meta/surgical-changes/SKILL.md`
2. `## 适用范围` / 工具依赖 / 工作流 / 按需自举 / 路由上下文（见 CONTRIBUTING §3）。
3. `## 常见借口与反驳 / Rationalizations` — 4-6 行**本领域特有**的偷懒借口 + 反驳。
4. `## 红旗信号 / Red Flags` — 4-6 条「正在走偏」的可观察信号。
5. `## 任务完成自检（声称完成前 MUST 通过）` — 首行指向 `../meta/verification-before-completion/SKILL.md`。
6. 语言行为契约；仅在真实分岔点放下一步菜单（CONTRIBUTING §3.4）。

## 4. 组合规则

- 领域 skill MAY 调用 meta skill；meta skill MUST NOT 调用领域 skill（避免循环与上下文膨胀）。
- 领域 skill 之间的交接通过「路由上下文」与 `../../ops/role-map.md`，不要互相内嵌全文。

## 5. 写作风格

- 使用 RFC 2119（`MUST/MUST NOT/SHOULD/MAY`），祈使句。
- 用表格和清单，少用长段落；每条规则能被检查。
- 例子用本领域真实场景，不要通用占位。
- 借鉴外部 skill 时只借思路、自己重写，并在 `../README.md` 归属表登记来源与许可证；安装外部 skill 前走 `../../ops/skill-supply-chain.md`。

## 6. 验证

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/extract-summaries.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/verify-routing-coherence.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/test-routing.ps1
```

改动若删除了 `verify-routing-coherence.ps1` 断言的关键词会直接 FAIL：升级旧 skill 时只增不删，迁移内容时保留被断言的字符串。

## 常见借口与反驳 / Rationalizations

| 借口 | 反驳 |
|------|------|
| 「description 写详细点，把流程也放进去」 | 流程放正文；description 只负责触发。 |
| 「全部放 SKILL.md 方便查看」 | 超长文件会被略读，细节下沉到 references/。 |
| 「这个领域不需要红旗信号」 | 每个领域都有典型走偏方式，写下来。 |

## 红旗信号 / Red Flags

- description 不以 "Use when" 开头或没有用户关键词。
- SKILL.md 超过 ~400 行且无 references/ 拆分。
- 缺少任务完成自检或未链接 verification meta skill。
- 从外部 repo 整段复制文本。

## 任务完成自检（声称完成前 MUST 通过）

- [ ] description 是触发条件且以 "Use when" 开头？
- [ ] anatomy 章节齐全，长内容已下沉 references/？
- [ ] `extract-summaries` / `verify-routing-coherence` / `test-routing` 全部通过？

## 语言行为契约

- **内部推理/工具选择/阶段控制**：使用 English。
- **用户可见消息/报告**：使用中文（除非用户要求其他语言）。
