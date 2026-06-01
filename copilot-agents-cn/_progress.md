# 成稿跟踪清单（中文版）

最后更新：随对话进行中

## 目标目录结构
```
~\.copilot\
    agents\
        conductor.agent.md      ✅ 已成稿（含计划审阅迭代回路）
        analyst.agent.md        ✅ 已成稿
        planner.agent.md        ✅ 已成稿（含迭代修改能力）
        coder.agent.md          ✅ 已成稿（含禁止 commit/push 约定）
        verifier.agent.md       ✅ 已成稿（双层验证 + 回归审查 + 只报告不裁决）
        test-writer.agent.md    ✅ 已成稿（含写测试 + 运行相关测试 + 禁止 commit/push）
        note-keeper.agent.md    ✅ 已成稿（三类写入 + 一致性审查 + 上报机制 + 禁止 commit/push）
    schemas\
        notes.md                ✅ 已成稿（含设计哲学 + 更新协议）
        planner-output.md       ✅ 已成稿
```

## 文件清单与对应路径
| 当前文件名 | 最终路径 | 状态 |
|-----------|---------|------|
| notes.md | schemas\notes.md | ✅ |
| planner-output.md | schemas\planner-output.md | ✅ |
| conductor.agent.md | agents\conductor.agent.md | ✅ |
| analyst.agent.md | agents\analyst.agent.md | ✅ |
| planner.agent.md | agents\planner.agent.md | ✅ |
| coder.agent.md | agents\coder.agent.md | ✅ |
| verifier.agent.md | agents\verifier.agent.md | ✅ |
| test-writer.agent.md | agents\test-writer.agent.md | ✅ |
| note-keeper.agent.md | agents\note-keeper.agent.md | ✅ |

## 全局约定（所有相关 agent 必须一致）
- **暂不限制 tools**：由于 MCP server 名称尚不稳定，当前不在 frontmatter 中配置 tools allowlist，先靠 prompt 约束职责边界，避免误伤 Confluence/Jira 等只读信息源。
- **用户入口控制**：Conductor 显式 `user-invocable: true`；其他 agent 均为 `user-invocable: false`，只作为被 Conductor 调度的专职 agent。
- **禁止版本控制操作**：任何 agent 都不得执行 git commit/push。有写权限的 agent（Coder ✅、Test Writer ✅、Note Keeper ✅）均已显式写明。commit/push 由用户决定或明确指派。
- **无状态 + Conductor 带上下文**：返工/迭代时，agent 不自动记得上一版，依赖 Conductor 把上一版产出 + 修改意见一起带入。已在 Planner ✅、Coder ✅、Test Writer ✅ 写明。

## 跨文件依赖（已全部核对）
- ✅ Verifier 与 Conductor 第六步已同步（双层验证 + 回归审查 + 三类处置）。
- ✅ Note Keeper 与 notes.md 的更新协议完全呼应，且明确"需核实/澄清时上报 Conductor"的边界。
- ✅ 所有写权限 agent 已加禁止 commit/push。

## 状态：全部中文版已成稿 ✅
下一步：等用户最终审阅定稿后，统一翻译成英文版。

## 独立审查后的修订（最终版）
经一次完整的交叉审查，修复了四处衔接/一致性问题：
1. notes.md：说明 project-notes 模板中的 `[标注]` 是占位符，须替换为真实标注。
2. verifier：放宽"三样输入缺一不可"，适配无计划的独立 review 场景；双层验证说明无计划时第一层可跳过。
3. coder + verifier：补齐多项目接收说明，与 Conductor/Planner/Analyst 对称。
4. conductor：补全"探索性讨论收敛后如何衔接到标准工作流（从委派 Planner 开始，复用探索阶段已摸清的现状）"。
（问题"测试是否进 ADR"判定为无需强制，交 Conductor 临场判断。）

## 增强：Notes 作为"渐进生长的活知识体"（新一轮）
新增一条核心能力——用户零散认知补充的"核实 + 融入"闭环。改动四个文件：
1. notes.md：设计哲学第3条升级为"渐进生长的活知识体"，新增第4条"融入前尽量代码核实"；更新协议重构为五步（判断是否核实 → 定位 → 融入merge → 一致性审查 → Metadata），明确 merge 的四种关系（填补/深化/修正/关联）。
2. analyst：新增"核实型调查"工作模式——先读现有 Notes，再带用户论断到代码查证，输出 证实/证伪/部分证实/无法判定。明确此 verification 与 Verifier 的 verification 不同。
3. conductor：新增"持续完善 Notes：吸纳用户零散认知补充"章节，含"默认倾向代码核实、纯意图才跳过"的裁量原则；常见场景对应条目同步更新。
4. note-keeper：任务 C 升级为"融入（merge）"，加入四种融入方式；核心原则4强化 merge 语义。
设计要点：意图与实现常被一起陈述，意图可不核实但实现论断要尽力核实（防止把过时记忆当事实写入）。

## 待办：定稿后统一翻译成英文版
所有中文版定稿后，统一改写为英文版。

## 修订：入口控制、测试执行与 Notes Metadata
根据新一轮审阅意见完成：
1. 所有非 Conductor agent 增加 `user-invocable: false`，Conductor 增加 `user-invocable: true`；暂不配置 tools allowlist。
2. Test Writer 职责扩展为编写并运行相关 unit test，汇报通过/失败/无法运行；若测试公正合理且失败非环境原因，应向 Conductor 报告疑似 Coder 实现问题。
3. Conductor 第七步同步更新为委派 Test Writer 编写并运行相关测试，并根据失败归因裁决后续动作。
4. Notes Metadata 从模糊的 branch/commit_id 改为 source_branch/source_commit_id/notes_scope/workspace_state；ADR 支持未提交状态，final_commit_id 可为 pending。

## 增强：新文档/diagram/skill 作为 Notes 知识输入
新增用例：用户可随时提供 Confluence 页面、diagram、截图/PDF、设计文档、聊天摘录、只言片语、预先写好的 skill 等材料来完善 Notes。改动点：
1. Conductor：新增"新文档/图/skill/零散材料完善 Notes"常见场景；持续完善 Notes 流程增加材料可访问性判断、文档消化、代码核实与来源保留。
2. Analyst：新增"文档消化型调查"工作模式，负责抽取候选知识、分类、核实实现论断、给出 Notes section 与标注建议；skill 默认作为知识来源阅读，不安装/执行。
3. Note Keeper：任务 C 扩展为"用户认知补充或新材料"，强调 merge 提炼后的知识而非粘贴原文，并保留必要来源。
4. notes.md：定点更新协议纳入新文档/diagram/skill 等不确定格式 artifact，明确先判断可读性和是否需代码核实。
