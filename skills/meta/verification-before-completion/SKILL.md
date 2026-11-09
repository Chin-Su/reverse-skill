---
name: verification-before-completion
description: Use when about to claim a reverse-engineering or pentest result is done, found, bypassed, decrypted, exploitable, or fixed — before writing the conclusion, report, or "完成". Forces fresh evidence, EXTRACTED/INFERRED provenance labels, and an explicit "not checked" list.
---

# 完成前验证门（Verification Before Completion）

> 元流程 skill：由领域 skill 的「任务完成自检」调用，不参与路由。全局规则见 `../../../RULES.md` §Self-Audit Before Claiming "Complete"；本文件只补「证据新鲜度 + 出处标签 + 有界修复循环」，不重复 RULES。

## ACTION REQUIRED（读完后立刻执行）

1. `NOW`: 列出本轮你准备宣称的每一条结论（找到 / 绕过 / 解密 / 可利用 / 已修复 / 完成）。
2. `NOW`: 为每条结论找到**本会话内**产生的命令输出、脚本输出、截图或产物路径；找不到的 MUST 现在重跑取证。
3. `NEXT`: 给每条结论打 `EXTRACTED` 或 `INFERRED` 标签，并挂到 Evidence ID（`../../ops/evidence-finding-path.md`）。
4. `ACT`: 不通过的结论进入「有界修复循环」；通过后再写报告。

## 核心规则

- `MUST NOT` 在没有新鲜证据时使用「已完成 / 已确认 / 成功 / 应该可以 / 看起来没问题」等措辞。
- 「新鲜」= 本会话、针对当前产物版本产生。旧会话笔记、field-journal、模型记忆只能作为线索，不能作为证据。
- 证据 MUST 可复现：给出命令（或脚本路径 + 参数）、目标 hash/版本、关键输出片段。

### 出处标签（Provenance）

| 标签 | 含义 | 示例 |
|------|------|------|
| `EXTRACTED` | 直接在产物或工具输出中看到 | jadx 输出中 `getSign()` 调用 `HmacSHA256`；Frida 日志打印出明文 key |
| `INFERRED` | 由多条 EXTRACTED 推导，未被直接观察 | 「该 so 中 `sub_1A2B0` 是 AES 轮函数」（仅凭常量 + 结构判断） |

- 报告中 `INFERRED` 结论 MUST 附置信度（高/中/低）和推导依据。
- 一条 Finding 若只有 `INFERRED` 支撑，MUST 标为「待验证」，不得写成已确认漏洞。

### 有界修复循环（最多 3 轮）

```
验证 → FAIL → 定位原因（../systematic-debugging/SKILL.md）→ 修正 → 重新验证
```

- 每轮 MUST 记录一条 Evidence（失败现象 + 改动 + 结果）。
- 3 轮后仍 FAIL：`MUST` 停止，向用户报告阻塞点、已尝试方案、下一步建议；不得悄悄降级目标或换题。

## 报告必含「未检查」清单

结论段之后 MUST 附：

```markdown
### 未检查 / 跳过 / Not checked
- <未覆盖的分支/平台/版本/输入>（原因）
- <跳过的验证步骤>（原因）
```

没有跳过项时写「无」，不要省略该小节。

## 常见借口与反驳 / Rationalizations

| 借口 | 反驳 |
|------|------|
| 「脚本没报错，所以 hook 生效了」 | 没报错 ≠ 命中；MUST 有命中日志或返回值变化。 |
| 「上次会话已经验证过」 | 产物/环境可能已变；重跑或明确标为历史证据。 |
| 「逻辑上一定是这个算法」 | 那是 `INFERRED`；用测试向量或动态对比转成 `EXTRACTED`。 |
| 「再试一次应该就好了」 | 已到 3 轮上限就停下报告，不要无限循环。 |
| 「用户只要结论」 | 结论可以简短，但证据锚点和未检查清单不可省。 |

## 红旗信号 / Red Flags

- 报告里出现「应该」「大概」「理论上」却没有 `INFERRED` 标签。
- 结论引用的输出无法在当前会话的工具调用中找到。
- 同一验证已失败 3 次但仍在改参数重试。
- 「未检查」小节缺失。

## 任务完成自检（声称完成前 MUST 通过）

- [ ] 每条结论都有本会话的新鲜证据与 Evidence ID？
- [ ] 每条结论都标注了 `EXTRACTED` / `INFERRED`（后者带置信度）？
- [ ] 修复循环未超过 3 轮，超过则已停止并报告？
- [ ] 报告含「未检查 / 跳过」小节？

## 语言行为契约

- **内部推理/工具选择/阶段控制**：使用 English。
- **用户可见消息/报告**：使用中文（除非用户要求其他语言）；标签格式「中文 / English」。
