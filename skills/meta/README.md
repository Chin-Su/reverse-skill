# meta — 跨领域流程技能

这些 skill **不参与路由**（不在 `config/routing.json` 中）。它们是领域 skill 在特定时刻调用的纪律：

| Skill | 何时调用 | 调用方 |
|-------|---------|--------|
| [verification-before-completion](verification-before-completion/SKILL.md) | 准备宣称「完成/找到/绕过/可利用」之前 | 各领域 skill 的「任务完成自检」 |
| [systematic-debugging](systematic-debugging/SKILL.md) | 工具/脚本/hook/payload 失败或结果异常 | 各领域 skill 的 ACTION REQUIRED |
| [surgical-changes](surgical-changes/SKILL.md) | 编写或修改脚本、PoC、patch、本仓库代码 | 各领域 skill 的 ACTION REQUIRED |
| [writing-skills](writing-skills/SKILL.md) | 新建或升级 SKILL.md | 维护者 / `../CONTRIBUTING.md` |

组合规则：领域 skill MAY 调用 meta skill；meta skill MUST NOT 调用领域 skill。

## 思路来源与许可证（Attribution）

以下仓库仅借鉴思路，本目录内容为针对 RE/渗透场景的重新编写，未复制原文，也未安装任何外部 skill（见 `../ops/skill-supply-chain.md`）。

| 来源 | 借鉴的思路 | 许可证 |
|------|-----------|--------|
| [obra/superpowers](https://github.com/obra/superpowers) | verification-before-completion、systematic-debugging、writing-skills | MIT |
| [mattpocock/skills](https://github.com/mattpocock/skills) | 分阶段 diagnosing-bugs；可调用方向的组合规则 | MIT |
| [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) | Surgical Changes、Goal-Driven Execution（每行改动可追溯） | MIT |
| [anthropics/skills](https://github.com/anthropics/skills) | `name` + `description` frontmatter 规范；渐进披露 | Apache-2.0（逐目录确认） |
| [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | 复用优先的决策阶梯；`shortcut:` 标记；交付时说明跳过项 | MIT |
| [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) | Rationalizations 表 + Red Flags + Verification 的 skill anatomy | MIT |
| [safishamsi/graphify](https://github.com/safishamsi/graphify) | `EXTRACTED` / `INFERRED` 出处标签 | MIT / Apache-2.0 |
| [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) | 有界完成门（最多 3 轮修复） | MIT |
| [affaan-m/ECC](https://github.com/affaan-m/ECC) | 证据链式交付（plan → 验证 → 复核） | MIT |
| [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman)、[nextlevelbuilder/ui-ux-pro-max-skill](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | 已评估，本次未采用（输出风格 / UI 领域不在范围内） | Apache-2.0 / MIT |
