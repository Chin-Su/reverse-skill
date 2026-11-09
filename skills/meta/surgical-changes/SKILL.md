---
name: surgical-changes
description: Use when writing or modifying code during an analysis — Frida/Objection scripts, IDAPython/Ghidra scripts, PoCs, smali or binary patches, solver scripts, or this repository's own scripts and skills. Applies a reuse-first decision ladder and keeps every changed line traceable to the request.
---

# 最小外科改动（Surgical Changes）

> 元流程 skill：领域 skill 中需要写/改代码时调用，不参与路由。

## ACTION REQUIRED（读完后立刻执行）

1. `NOW`: 写下本次改动要满足的**可验证目标**（例如「hook 后打印 sign 明文」「patch 后跳过 root 检测」）。
2. `NEXT`: 走下方决策阶梯，找到最低一级可行方案。
3. `ACT`: 只做该方案需要的改动，并准备一条验证命令。

## 决策阶梯（从上到下，命中即停）

1. **真的需要写代码吗？** 现有工具输出/一次手动操作能否回答问题？
2. **当前 skill 的 `scripts/` 里已有吗？** 例如 `../../apk-reverse/scripts/`、`../../ida-reverse/scripts/`。
3. **`../../tool-index.md` 里有现成工具/MCP 能做吗？**
4. **一行命令能解决吗？**（jq、rabin2、frida -e、一行 Python）
5. **最小脚本 + 一条验证**：只写达到目标所需的代码。

## 规则

- `MUST`：每一行改动都能追溯到用户请求或当前目标；无法追溯的改动不要做。
- `MUST NOT`：顺手重构、改风格、重命名、删除与任务无关的代码/注释；看到问题时**提出来**，不要擅自改。
- `MUST`：保留原有校验、错误处理和安全边界；「更少代码」不等于删掉防护。
- `SHOULD`：有意为之的捷径用注释标记，便于后续收集：
  - `// shortcut: hard-coded offset for v3.2.1 only`
  - `# shortcut: no retry, single-shot PoC`
- `MUST`：二进制/smali patch 前保留原文件 hash 与备份路径，写入 Evidence。
- 修改本仓库脚本时：遵循现有风格与 PowerShell 5.1 兼容性，跑相关 `skills/scripts/test-*.ps1`。

## 交付时说明

- 改了什么（文件 + 范围）。
- 刻意没做/跳过了什么（含 `shortcut:` 列表）。
- 验证命令与结果（交给 `../verification-before-completion/SKILL.md`）。

## 常见借口与反驳 / Rationalizations

| 借口 | 反驳 |
|------|------|
| 「顺便把这个函数也整理一下」 | 不在请求内；提出建议，不要改。 |
| 「写个通用框架以后能复用」 | 先解决当前问题；复用需求出现第二次再抽象。 |
| 「删掉这段检查脚本更简洁」 | 校验与错误处理不是冗余，保留。 |
| 「从头写比读现有脚本快」 | 先查 `scripts/` 与 tool-index，重复造轮子会引入新 bug。 |

## 红旗信号 / Red Flags

- diff 中出现与目标无关的格式化/重命名。
- 新增依赖却未检查 tool-index / bootstrap manifest。
- patch 原文件无备份或 hash。
- 交付说明里没有「跳过了什么」。

## 任务完成自检（声称完成前 MUST 通过）

- [ ] 已按决策阶梯选择最低可行方案？
- [ ] 每个改动可追溯到目标，无无关修改？
- [ ] patch 有原始 hash/备份？
- [ ] 已给出验证命令与结果，以及跳过项说明？

## 语言行为契约

- **内部推理/工具选择/阶段控制**：使用 English。
- **用户可见消息/报告**：使用中文（除非用户要求其他语言）。
