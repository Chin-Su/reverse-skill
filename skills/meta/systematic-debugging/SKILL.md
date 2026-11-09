---
name: systematic-debugging
description: Use when a reverse-engineering or pentest step fails or behaves unexpectedly — a Frida hook never fires, a script crashes, a decompiler output looks wrong, a payload does not trigger, a tool returns empty, or bootstrap succeeds but the tool still fails. Replaces trial-and-error with a gated reproduce → isolate → hypothesize → fix loop.
---

# 系统化排错（Systematic Debugging for RE / Pentest）

> 元流程 skill：领域 skill 中任何工具/脚本/假设失败时调用，不参与路由。缺工具仍走 bootstrap（`../../scripts/bootstrap-reverse.ps1`），本 skill 处理「工具在但结果不对」。

## ACTION REQUIRED（读完后立刻执行）

1. `NOW`: 用一句话写下「预期 vs 实际」（含命令、目标、版本）。
2. `NOW`: 停止随机改参数；进入阶段 1。
3. `ACT`: 按四阶段推进，每阶段出口条件满足后才进入下一阶段。

## 四阶段流程

### 阶段 1 — 复现（Reproduce）

- MUST 得到一个最小、可重复触发的失败命令/脚本。
- 记录环境：工具版本（来自 `../../tool-index.md`）、设备/架构、目标 hash。
- **出口**：同一命令连续 2 次得到同样的失败。偶发失败先记录频率，不要急着修。

### 阶段 2 — 隔离（Isolate）

- 一次只改变一个变量：目标进程 / 时机（spawn vs attach）/ 地址或符号 / 参数 / 工具版本。
- 优先用二分法缩小范围：已知正常的最小样例 ↔ 当前失败样例。
- 常见 RE 隔离点：

| 现象 | 先检查 |
|------|--------|
| Frida hook 不触发 | 进程/线程是否正确；类是否已加载；方法重载签名；是否被内联/JIT；so 加载时机 |
| 反编译结果离谱 | 架构/基址/Thumb 模式；是否加壳或 OLLVM；函数边界；数据被当代码 |
| 脚本崩溃 | 输入格式；编码（UTF-8/GBK）；路径空格；PowerShell 5.1 vs pwsh 差异 |
| payload 不生效 | 是否命中目标端点；编码/转义层数；WAF/中间件改写；响应是否真来自目标 |
| 工具返回空 | 输入是否真的被读到；过滤条件；权限；MCP 服务是否在线 |

- **出口**：能说出「问题出在 X 层，而不是 Y 层」，并有一条 Evidence 支撑。

### 阶段 3 — 假设与实验（Hypothesize）

- 每个假设 MUST 写成可证伪形式：「如果 H 成立，那么执行 E 会观察到 O」。
- 每个假设对应一条 Evidence（假设、实验、观察、结论 confirmed/refuted）。
- 连续 3 个假设被否定：`MUST` 换方法（静态↔动态、换工具交叉验证）或向用户报告，不要继续同一路线。

### 阶段 4 — 修复与验证（Fix + Verify）

- 只修复已证实的根因；改动遵循 `../surgical-changes/SKILL.md`。
- 用阶段 1 的复现命令重新验证，然后进入 `../verification-before-completion/SKILL.md`。

## 常见借口与反驳 / Rationalizations

| 借口 | 反驳 |
|------|------|
| 「换几个参数试试就行」 | 不记录假设的尝试无法收敛，先隔离。 |
| 「工具有 bug」 | 先用已知正常样例证明工具本身失败，再下结论。 |
| 「应该是反调试」 | 那是假设；找到检测点或绕过前后对比才算证实。 |
| 「重装一下工具」 | 重装是变量之一；先确认版本/路径确实是原因。 |

## 红旗信号 / Red Flags

- 同时改了两个以上变量后宣称「修好了」。
- 没有复现命令就开始改代码。
- 第 4 次尝试同一思路。
- 把「问题消失了」当成「理解了根因」。

## 任务完成自检（声称完成前 MUST 通过）

- [ ] 有最小复现命令？
- [ ] 根因有 Evidence 支撑（而非猜测）？
- [ ] 每个假设都记录了 confirmed/refuted？
- [ ] 修复后用同一复现命令验证通过？

## 语言行为契约

- **内部推理/工具选择/阶段控制**：使用 English。
- **用户可见消息/报告**：使用中文（除非用户要求其他语言）。
