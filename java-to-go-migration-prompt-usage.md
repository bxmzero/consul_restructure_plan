# Java 到 Go 迁移提示词使用说明

## 1. 目的

本文档说明如何使用 `java-to-go-migration-stage-prompts.md`。

`java-to-go-migration-stage-prompts.md` 只作为提示词集合使用，不作为一次性执行清单。

## 2. 基本执行方式

每次只复制一个阶段的一个批次提示词，不要把 1-6 阶段一次性发给 Agent。

推荐流程：

```text
选择当前阶段和当前批次
-> 复制对应阶段启动提示词
-> 执行 brainstorming
-> 人工确认范围和方案
-> 复制 7.1 生成当前批次计划
-> 复制 7.2 执行当前批次计划
-> 复制 7.3 批次 Review 和完成验证
```

阶段必须按顺序推进：

```text
阶段 1：ORM / Repository
-> 阶段 2：API Router / Handler / DTO
-> 阶段 3：横切基础设施
-> 阶段 4：并发与生命周期
-> 阶段 5：Service 业务链路
-> 阶段 6：系统级功能校验
```

## 3. 阶段 ID 和 Batch ID

批次 ID 使用统一格式：

```text
<阶段ID>-<业务/能力>-<步骤序号>
```

语义是“阶段_业务_步骤”，实际分隔符使用 `-`。

示例：

```text
P1-orm-01
P2-api-01
P4-runtime-02
```

| 阶段 ID | 阶段名称 | 业务/能力 ID | Batch ID 格式 | 示例 |
|---|---|---|---|---|
| `P1` | 持久化层迁移 | `orm` | `P1-orm-<NN>` | `P1-orm-01` |
| `P2` | API 契约层迁移 | `api` | `P2-api-<NN>` | `P2-api-01` |
| `P3` | 横切基础设施迁移 | `infra` | `P3-infra-<NN>` | `P3-infra-01` |
| `P4` | 并发与生命周期迁移 | `runtime` | `P4-runtime-<NN>` | `P4-runtime-01` |
| `P5` | Service 业务层迁移 | `service` | `P5-service-<NN>` | `P5-service-01` |
| `P6` | 系统级功能校验 | `verify` | `P6-verify-<NN>` | `P6-verify-01` |

如一个阶段内存在多个业务域，可以在业务/能力 ID 后追加领域名：

```text
P1-orm-kv-01
P2-api-kv-01
P4-runtime-scheduler-01
```

默认优先使用简单格式，例如 `P1-orm-01`。只有当同一阶段同一能力下需要并行区分多个业务域时，才使用扩展格式。

## 4. 批次隔离规则

一个阶段可以拆成多个批次执行。

示例：

```text
阶段 1
-> P1-orm-01：迁移一部分表的 ORM
-> P1-orm-02：迁移另一部分表的 ORM
-> P1-orm-03：迁移最后一部分表的 ORM
-> 阶段 1 完整 Review
-> 生成 phase-1-persistence-handoff.md
-> 进入阶段 2
```

批次隔离要求：

- 每次只执行一个批次。
- 当前批次只处理 `Java source scope` 和 `Go target scope` 内的内容。
- 未列入 scope 的内容默认不是当前批次目标。
- 后续批次可以读取前序批次成果，但不得覆盖前序批次的 Superpowers specs/plans。
- 如果必须修改前序批次已验收代码，必须记录 Decision/GAP、影响范围和重新验证结果。
- `phase-N-handoff.md` 只在当前阶段全部批次完成后生成，不要每个批次都生成阶段 handoff。

## 5. 共同阅读文档

每个提示词执行前，Agent 都应该阅读：

```text
/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
```

其中：

- `java-to-go-migration-mainline.md` 定义六阶段主线、边界和全局溯源规则。
- `migration-governance-templates.md` 定义自定义迁移治理文档的结构和更新规则。
- `java-to-go-migration-prompt-usage.md` 定义提示词使用方式和批次隔离规则。

## 6. 文档输出位置

Superpowers 产物：

```text
<GO_PROJECT_ROOT>/spec/superpowers/specs/
<GO_PROJECT_ROOT>/spec/superpowers/plans/
```

自定义迁移治理产物：

```text
<GO_PROJECT_ROOT>/spec/migration/migration-roadmap.md
<GO_PROJECT_ROOT>/spec/migration/migration-decisions.md
<GO_PROJECT_ROOT>/spec/migration/migration-gaps.md
<GO_PROJECT_ROOT>/spec/migration/migration-traceability.md
<GO_PROJECT_ROOT>/spec/migration/handoffs/phase-N-<name>-handoff.md
```

如果 Superpowers 因默认行为生成到 `docs/superpowers/`，不要直接删除或覆盖。先在 `migration-roadmap.md` 中记录实际路径，再由人工确认是否调整目录。

## 7. 占位符

使用阶段提示词前，替换以下占位符：

```text
<JAVA_PROJECT_ROOT>
<GO_PROJECT_ROOT>
<JAVA_BASELINE_COMMIT>
<CURRENT_BATCH>
<CURRENT_BATCH_GOAL>
<CURRENT_BATCH_JAVA_SCOPE>
<CURRENT_BATCH_GO_SCOPE>
<PREVIOUS_BATCH_REFERENCES>
<IS_FINAL_BATCH_OF_PHASE>
<PREVIOUS_STAGE_HANDOFF>
```

`java-to-go-migration-stage-prompts.md` 中每段提示词开头都有“本次输入值”块。实际使用时，只需要填写这个输入值块；正文里重复出现的 `<...>` 占位符由 Agent 按输入值块解析，不需要逐个替换。

填写范本见：

```text
/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-placeholder-examples.md
```

## 8. 每批次确认清单

在确认 brainstorming 结果前检查：

```text
1. 是否只覆盖当前阶段？
2. 是否只覆盖当前批次？
3. 是否明确当前批次 ID、Java source scope、Go target scope？
4. 是否读取必要的前序批次参考或上一阶段 handoff？
5. 是否记录 Java 基线 commit？
6. 是否规划 Java/Go 文件和符号溯源？
7. 是否规划 migration-traceability.md 更新？
8. 是否定义当前批次测试和完成条件？
9. 是否识别风险、假设和 GAP？
10. 是否避免提前实现后续阶段？
11. 是否确认不会覆盖前序批次 specs/plans 或已验收代码？
12. 是否确认只有阶段最后一个批次完成后才生成 phase-N-handoff.md？
```
