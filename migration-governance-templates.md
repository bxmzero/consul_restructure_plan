# Java 到 Go 迁移治理文档模板

## 1. 目的

本文档定义 Java 到 Go 分阶段重构过程中，自定义迁移治理文档的固定结构和更新规则。

所有参与迁移的人工、主 Agent 和 Subagent 在创建或更新治理文档前，都必须先阅读本文档。

本文档只定义迁移治理产物，不替代 Superpowers 官方产物。

Superpowers 官方默认可能会把 `brainstorming` 和 `writing-plans` 产物放到 `docs/superpowers/...`。本迁移项目为了让 Java 到 Go 的规格、计划、治理文档集中管理，统一要求 Agent 在提示词中把 Superpowers 产物输出到项目根目录的 `spec/superpowers/...` 下。

如果 Superpowers 工具因为默认行为已经生成到其他目录，不要直接删除或覆盖；必须先在 `migration-roadmap.md` 中记录实际路径，再由人工决定是否迁移目录。

```text
Superpowers 产物，本项目约定落点：
- spec/superpowers/specs/YYYY-MM-DD-<topic>-design.md
- spec/superpowers/plans/YYYY-MM-DD-<feature-name>.md

自定义迁移治理产物：
- spec/migration/migration-roadmap.md
- spec/migration/migration-decisions.md
- spec/migration/migration-gaps.md
- spec/migration/migration-traceability.md
- spec/migration/handoffs/phase-N-<name>-handoff.md
```

## 2. 推荐目录

```text
spec/
├── superpowers/
│   ├── specs/
│   └── plans/
└── migration/
    ├── migration-roadmap.md
    ├── migration-decisions.md
    ├── migration-gaps.md
    ├── migration-traceability.md
    └── handoffs/
        ├── phase-1-persistence-handoff.md
        ├── phase-2-api-contract-handoff.md
        ├── phase-3-infrastructure-handoff.md
        ├── phase-4-runtime-handoff.md
        ├── phase-5-service-handoff.md
        └── phase-6-verification-handoff.md
```

如项目已有文档目录约定，可以调整目录，但必须在 `migration-roadmap.md` 中记录实际位置，并在所有阶段保持一致。

## 3. 全局规则

### 3.1 事实来源优先级

```text
Java 源码和实际运行行为
> 已确认的迁移决策
> Superpowers design/plan
> 分析报告
> Agent 推断
```

推断内容不得写成已确认事实，必须标记为假设或 GAP。

### 3.2 基线信息

每份治理文档必须包含：

- Java 项目路径或仓库地址
- Go 项目路径或仓库地址
- Java 基线分支
- Java 基线 commit SHA
- Go 当前分支
- Go 当前 commit SHA
- 当前迁移阶段
- 当前迁移批次
- 最后更新时间

### 3.3 编号规则

```text
迁移决策：DEC-001、DEC-002、DEC-003
迁移缺口：GAP-001、GAP-002、GAP-003
迁移批次：P1-orm-01、P1-orm-02、P2-api-01
```

说明：

- 批次 ID 格式为 `<阶段ID>-<业务/能力>-<步骤序号>`，例如 `P1-orm-01`
- `P1-orm-01` 表示阶段 1 持久化层迁移的 ORM 第 1 个批次
- 编号一旦分配不得重复使用
- 已废弃或已解决的编号不得删除或重新分配
- 新记录使用当前最大编号加一

### 3.4 修改规则

```text
migration-roadmap.md：更新索引和当前批次段落，不重复创建同名阶段
migration-decisions.md：追加 Decision 到当前批次段落；已有 Decision 通过状态更新或新 Decision 替代
migration-gaps.md：追加或更新当前批次 GAP 状态，不删除历史 GAP
migration-traceability.md：增量更新当前批次映射，不覆盖无关历史记录
phase-N-handoff.md：阶段结束时生成，重新验收时更新同一个文件
```

上述四个批次级治理文档必须采用统一结构：

```markdown
# <文档标题>

## 索引

## <CURRENT_BATCH>

## <CURRENT_BATCH>

## 变更记录
```

规则：

- 每个批次必须使用二级标题 `## <CURRENT_BATCH>` 作为写入隔离边界。
- 同一文档内不得出现重复的批次二级标题。
- 当前批次只能新增或更新自己的 `## <CURRENT_BATCH>` 段落。
- 不得修改其他批次段落，除非记录 Decision/GAP、影响范围和重新验证结果。
- `## 索引` 和 `## 变更记录` 可以增量更新，但不得删除历史批次、历史 Decision 或历史 GAP。
- 上述四个文档中的所有表格必须包含 `批次 ID` 列；全局记录使用 `GLOBAL` 作为 `批次 ID`。

### 3.5 批次隔离规则

同一阶段可以拆成多个批次执行，例如阶段 1 ORM 可以拆成 `P1-orm-01`、`P1-orm-02`。

每个批次必须有清晰边界，避免前后批次或不同阶段互相干扰。

批次启动前必须明确：

- 批次 ID，例如 `P1-orm-01`
- 当前批次目标
- 当前批次 Java source scope，即本次允许分析和迁移的 Java 源码路径
- 当前批次 Go target scope，即本次允许新增或修改的 Go 目录或文件
- 当前批次允许读取的历史文档
- 当前批次禁止修改的文件或能力范围

Superpowers specs/plans 文件名必须包含批次 ID：

```text
spec/superpowers/specs/YYYY-MM-DD-P1-orm-01-design.md
spec/superpowers/plans/YYYY-MM-DD-P1-orm-01-plan.md
spec/superpowers/specs/YYYY-MM-DD-P1-orm-kv-01-design.md
spec/superpowers/plans/YYYY-MM-DD-P1-orm-kv-01-plan.md
```

批次之间的更新规则：

- 当前批次只能修改本批次 scope 内的 Go 代码和测试。
- 当前批次可以读取前序批次产物，但不得覆盖前序批次的 specs/plans。
- 前序批次引用只需要填写批次 ID，例如 `P1-orm-01` 或 `P1-orm-01,P1-orm-02`；Agent 必须按批次 ID 自动检索对应 spec、plan、migration 记录和 traceability 中的代码/测试路径，不要求人工逐个列文件路径。
- 当前批次如需修改前序批次已验收代码，必须记录影响批次、修改原因、验证证据，并更新 `migration-decisions.md` 或 `migration-gaps.md`。
- 批次完成时只更新 `migration-roadmap.md`、`migration-traceability.md`、`migration-decisions.md`、`migration-gaps.md`。
- `phase-N-handoff.md` 只在整个阶段完成并通过 Review 后生成；不要每个批次都生成一个阶段 handoff。

### 3.6 禁止事项

- 不允许删除历史 Decision 或 GAP
- 不允许把未验证内容标记为已完成
- 不允许覆盖其他阶段的映射记录
- 不允许在没有证据的情况下关闭 GAP
- 不允许只记录 Go 文件而不记录 Java 来源
- 不允许使用本机绝对路径作为 Java `contentPath`
- 不允许把 Superpowers 的 Execution Handoff 和阶段 `handoff.md` 混为一谈
- 不允许用相同文件名覆盖前序批次的 Superpowers specs/plans
- 不允许在批次未覆盖的 Java source scope 外顺手迁移代码

## 4. migration-roadmap.md 模板

### 4.1 用途

记录整个 Java 到 Go 重构项目的阶段状态、当前批次、依赖、门禁和阻塞项。

该文件是迁移项目的总进度入口。

### 4.2 更新时机

- 迁移项目启动时创建
- 每个批次开始时更新
- 每个批次完成时更新
- 每个阶段完成时更新
- 阶段状态、阻塞状态或基线发生变化时更新

### 4.3 模板

```markdown
# Java 到 Go 迁移路线图

## 索引

### 基线信息

| 批次 ID | 字段 | 值 |
|---|---|---|
| GLOBAL | Java 项目 | `<JAVA_PROJECT>` |
| GLOBAL | Java 基线分支 | `<JAVA_BRANCH>` |
| GLOBAL | Java 基线 commit | `<JAVA_COMMIT_SHA>` |
| GLOBAL | Go 项目 | `<GO_PROJECT>` |
| GLOBAL | Go 当前分支 | `<GO_BRANCH>` |
| GLOBAL | Go 当前 commit | `<GO_COMMIT_SHA>` |
| GLOBAL | 当前阶段 | `<PHASE>` |
| GLOBAL | 当前批次 | `<CURRENT_BATCH>` |
| GLOBAL | 最后更新时间 | `<YYYY-MM-DD HH:mm>` |

### 阶段总览

| 批次 ID | 阶段 | 名称 | 状态 | 当前/最近批次 | 前置依赖 | 验证结果 | 阻塞项 |
|---|---|---|---|---|---|---|---|
| GLOBAL | 1 | 持久化层迁移 | 进行中 | `P1-orm-01` | 无 | 部分通过 | `GAP-001` |
| GLOBAL | 2 | API 契约层迁移 | 未开始 | - | 阶段 1 | 未执行 | - |
| GLOBAL | 3 | 横切基础设施迁移 | 未开始 | - | 阶段 2 | 未执行 | - |
| GLOBAL | 4 | 并发与生命周期迁移 | 未开始 | - | 阶段 3 | 未执行 | - |
| GLOBAL | 5 | Service 业务层迁移 | 未开始 | - | 阶段 1-4 | 未执行 | - |
| GLOBAL | 6 | 系统级功能校验 | 未开始 | - | 阶段 1-5 | 未执行 | - |

### 批次索引

| 批次 ID | 阶段 | 目标 | Java source scope | Go target scope | Superpowers spec | Superpowers plan | 状态 | 最近更新时间 |
|---|---|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `<PHASE>` | `<GOAL>` | `<JAVA_SCOPE>` | `<GO_SCOPE>` | `<SPEC_PATH>` | `<PLAN_PATH>` | `<STATUS>` | `<TIME>` |

### 当前阶段门禁

| 批次 ID | 门禁项 | 状态 | 证据 |
|---|---|---|---|
| GLOBAL | 当前阶段范围内工作完成 | `<STATUS>` | `<EVIDENCE>` |
| GLOBAL | 测试和验证通过 | `<STATUS>` | `<EVIDENCE>` |
| GLOBAL | 阻塞 Review 问题关闭 | `<STATUS>` | `<EVIDENCE>` |
| GLOBAL | migration-decisions.md 已更新 | `<STATUS>` | `<EVIDENCE>` |
| GLOBAL | migration-gaps.md 已更新 | `<STATUS>` | `<EVIDENCE>` |
| GLOBAL | migration-traceability.md 已更新 | `<STATUS>` | `<EVIDENCE>` |
| GLOBAL | Go 文件和关键符号已标注 Java 来源 | `<STATUS>` | `<EVIDENCE>` |
| GLOBAL | Java 基线 commit 已记录 | `<STATUS>` | `<EVIDENCE>` |
| GLOBAL | 阶段 handoff.md 已生成 | `<STATUS>` | `<EVIDENCE>` |

## `<CURRENT_BATCH>`

### 批次信息

| 批次 ID | 字段 | 值 |
|---|---|---|
| `<CURRENT_BATCH>` | 阶段 | `<PHASE>` |
| `<CURRENT_BATCH>` | 批次目标 | `<GOAL>` |
| `<CURRENT_BATCH>` | Java source scope | `<JAVA_SCOPE>` |
| `<CURRENT_BATCH>` | Go target scope | `<GO_SCOPE>` |
| `<CURRENT_BATCH>` | 前序批次 | `<PREVIOUS_BATCH_REFERENCES>` |
| `<CURRENT_BATCH>` | 是否阶段最后批次 | `<IS_FINAL_BATCH_OF_PHASE>` |
| `<CURRENT_BATCH>` | Spec | `<SUPERPOWERS_SPEC_PATH>` |
| `<CURRENT_BATCH>` | Plan | `<SUPERPOWERS_PLAN_PATH>` |
| `<CURRENT_BATCH>` | 状态 | `<STATUS>` |
| `<CURRENT_BATCH>` | 开始时间 | `<START_TIME>` |
| `<CURRENT_BATCH>` | 完成时间 | `<END_TIME_OR_EMPTY>` |

### 执行摘要

| 批次 ID | 项目 | 内容 |
|---|---|---|
| `<CURRENT_BATCH>` | 完成内容 | `<SUMMARY>` |
| `<CURRENT_BATCH>` | 测试结果 | `<TEST_RESULT>` |
| `<CURRENT_BATCH>` | Review 结果 | `<REVIEW_RESULT>` |
| `<CURRENT_BATCH>` | 关联 Decision | `<DEC_IDS>` |
| `<CURRENT_BATCH>` | 关联 GAP | `<GAP_IDS>` |
| `<CURRENT_BATCH>` | 关联 Traceability | `<TRACEABILITY_SECTION_OR_ROWS>` |
| `<CURRENT_BATCH>` | 下一步 | `<NEXT_STEP>` |

## 变更记录

| 批次 ID | 时间 | 类型 | 摘要 | 证据 |
|---|---|---|---|---|
| `<CURRENT_BATCH>` | `<TIME>` | `<START/UPDATE/REVIEW/DONE>` | `<SUMMARY>` | `<EVIDENCE>` |
```

### 4.4 状态枚举

```text
未开始
设计中
计划中
进行中
阻塞
待验证
已完成
```

## 5. migration-decisions.md 模板

### 5.1 用途

记录迁移过程中的架构、兼容性、范围和实现决策，以及选择理由和影响范围。

### 5.2 更新时机

- brainstorming 确认设计后
- writing-plans 发现需要固定的实现策略时
- 实现过程中发生计划外设计变化时
- Review 或验证导致原决策调整时

### 5.3 模板

```markdown
# Java 到 Go 迁移决策记录

## 索引

| 批次 ID | Decision ID | 阶段 | 标题 | 状态 | 日期 | 替代/被替代 |
|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `DEC-XXX` | `<PHASE>` | `<TITLE>` | 已确认 | `<DATE>` | - |

## `<CURRENT_BATCH>`

### DEC-XXX：`<决策标题>`

#### 基本信息

| 批次 ID | 字段 | 值 |
|---|---|---|
| `<CURRENT_BATCH>` | Decision ID | `DEC-XXX` |
| `<CURRENT_BATCH>` | 状态 | 提议 / 已确认 / 已废弃 / 已替代 |
| `<CURRENT_BATCH>` | 日期 | `<YYYY-MM-DD>` |
| `<CURRENT_BATCH>` | 阶段 | `<PHASE>` |
| `<CURRENT_BATCH>` | 决策人 | `<HUMAN_OR_AGENT>` |
| `<CURRENT_BATCH>` | Java 基线 commit | `<JAVA_COMMIT_SHA>` |

#### 背景

<为什么需要做这个决策>

#### Java 现状与证据

| 批次 ID | Java contentPath | Java 已迁移符号 | 证据摘要 |
|---|---|---|---|
| `<CURRENT_BATCH>` | `<PATH>` | `<CLASS/METHOD/FIELD>` | `<EVIDENCE>` |

#### 候选方案

1. `<OPTION_A>`
2. `<OPTION_B>`
3. `<OPTION_C>`

#### 最终选择

<选择的方案>

#### 选择原因

<为什么选择>

#### 影响范围

| 批次 ID | 类型 | 路径/对象 |
|---|---|---|
| `<CURRENT_BATCH>` | Go 文件 | `<GO_PATHS>` |
| `<CURRENT_BATCH>` | 测试 | `<TEST_PATHS>` |
| `<CURRENT_BATCH>` | 后续阶段 | `<AFFECTED_PHASES>` |

#### 验证方式

<如何证明决策有效>

#### 关联记录

| 批次 ID | 类型 | 关联项 |
|---|---|---|
| `<CURRENT_BATCH>` | GAP | `<GAP_IDS>` |
| `<CURRENT_BATCH>` | Traceability | `<TRACEABILITY_SECTION_OR_ROWS>` |
| `<CURRENT_BATCH>` | 替代 Decision | `<DEC_ID_OR_NONE>` |
| `<CURRENT_BATCH>` | 被替代 Decision | `<DEC_ID_OR_NONE>` |

## 变更记录

| 批次 ID | 时间 | Decision ID | 类型 | 摘要 |
|---|---|---|---|---|
| `<CURRENT_BATCH>` | `<TIME>` | `DEC-XXX` | `<CREATE/UPDATE/REPLACE>` | `<SUMMARY>` |
```

## 6. migration-gaps.md 模板

### 6.1 用途

记录无法确认、尚未迁移、行为不一致或需要人工判断的内容。

### 6.2 更新时机

- Java 行为无法从源码确认时
- Java 和 Go 行为存在差异时
- 当前阶段明确推迟某项能力时
- 数据库方言、外部依赖或运行环境无法验证时
- Review 或测试发现未覆盖行为时

### 6.3 模板

```markdown
# Java 到 Go 迁移缺口记录

## 索引

| 批次 ID | GAP ID | 阶段 | 标题 | 严重级别 | 状态 | 阻塞阶段 | 负责人 |
|---|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `GAP-XXX` | `<PHASE>` | `<TITLE>` | 高 | 待确认 | `<BLOCKED_PHASE>` | `<OWNER>` |

## `<CURRENT_BATCH>`

### GAP-XXX：`<缺口标题>`

#### 基本信息

| 批次 ID | 字段 | 值 |
|---|---|---|
| `<CURRENT_BATCH>` | GAP ID | `GAP-XXX` |
| `<CURRENT_BATCH>` | 状态 | 待确认 / 已确认 / 处理中 / 已解决 / 接受风险 / 排除 |
| `<CURRENT_BATCH>` | 严重级别 | 阻塞 / 高 / 中 / 低 |
| `<CURRENT_BATCH>` | 发现日期 | `<YYYY-MM-DD>` |
| `<CURRENT_BATCH>` | 发现阶段 | `<PHASE>` |
| `<CURRENT_BATCH>` | 阻塞阶段 | `<BLOCKED_PHASE>` |
| `<CURRENT_BATCH>` | 负责人 | `<OWNER>` |
| `<CURRENT_BATCH>` | Java 基线 commit | `<JAVA_COMMIT_SHA>` |

#### 问题描述

<缺口是什么>

#### Java 来源与证据

| 批次 ID | Java contentPath | Java 关键符号 | 证据 |
|---|---|---|---|
| `<CURRENT_BATCH>` | `<PATH>` | `<CLASS/METHOD/FIELD>` | `<EVIDENCE>` |

#### Go 影响范围

| 批次 ID | 项目 | 内容 |
|---|---|---|
| `<CURRENT_BATCH>` | Go contentPath | `<PATHS>` |
| `<CURRENT_BATCH>` | 影响阶段 | `<PHASES>` |
| `<CURRENT_BATCH>` | 影响行为 | `<BEHAVIOR>` |

#### 当前处理

<临时处理或当前结论>

#### 关闭条件

<满足什么证据后可以关闭>

#### 最终处理结果

<未解决时保持为空>

#### 关联记录

| 批次 ID | 类型 | 关联项 |
|---|---|---|
| `<CURRENT_BATCH>` | Decision | `<DEC_IDS>` |
| `<CURRENT_BATCH>` | Traceability | `<TRACEABILITY_SECTION_OR_ROWS>` |
| `<CURRENT_BATCH>` | 测试 | `<TEST_PATHS>` |

## 变更记录

| 批次 ID | 时间 | GAP ID | 类型 | 摘要 |
|---|---|---|---|---|
| `<CURRENT_BATCH>` | `<TIME>` | `GAP-XXX` | `<CREATE/UPDATE/CLOSE>` | `<SUMMARY>` |
```

## 7. migration-traceability.md 模板

### 7.1 用途

记录 Java 文件、Java 关键符号、Go 文件、Go 关键符号、测试和迁移状态之间的映射，用于遗漏检查和增量同步。

### 7.2 更新时机

- 每个批次开始时登记待迁移范围
- 每个任务完成时更新对应记录
- 每个批次 Review 后更新验证结果
- Java 基线变化时执行增量扫描并更新
- 阶段结束前完成一次完整性检查

### 7.3 模板

```markdown
# Java 到 Go 迁移溯源矩阵

## 索引

### 基线信息

| 批次 ID | 字段 | 值 |
|---|---|---|
| GLOBAL | Java 基线分支 | `<JAVA_BRANCH>` |
| GLOBAL | Java 基线 commit | `<JAVA_COMMIT_SHA>` |
| GLOBAL | Go 分支 | `<GO_BRANCH>` |
| GLOBAL | Go commit | `<GO_COMMIT_SHA>` |
| GLOBAL | 最后更新时间 | `<TIME>` |

### 批次索引

| 批次 ID | 阶段 | Java source scope | Go target scope | 映射数量 | 状态 | 最近更新时间 |
|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `<PHASE>` | `<JAVA_SCOPE>` | `<GO_SCOPE>` | `<COUNT>` | `<STATUS>` | `<TIME>` |

## `<CURRENT_BATCH>`

### 批次范围

| 批次 ID | 字段 | 值 |
|---|---|---|
| `<CURRENT_BATCH>` | 阶段 | `<PHASE>` |
| `<CURRENT_BATCH>` | Java source scope | `<JAVA_SCOPE>` |
| `<CURRENT_BATCH>` | Go target scope | `<GO_SCOPE>` |
| `<CURRENT_BATCH>` | Java 基线 commit | `<JAVA_COMMIT_SHA>` |
| `<CURRENT_BATCH>` | Go commit | `<GO_COMMIT_SHA>` |
| `<CURRENT_BATCH>` | 状态 | `<STATUS>` |

### 文件与符号映射

| 批次 ID | Java contentPath | Java 已迁移符号 | Go contentPath | Go 关键符号 | 映射类型 | 状态 | 验证结果 | GAP/备注 |
|---|---|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `bic-service/src/.../FooMapper.java` | `selectFoo` | `internal/.../foo_repository.go` | `GetFoo` | 1:1 | 已验证 | 测试通过 | 无 |

### MyBatis Example 映射

MyBatis Example / Criteria / Criterion 不直接记录为 `XxxExample.java -> XxxExample.go`。

如果 Java 业务代码使用 Example 拼装查询条件，溯源记录应写成：

| 批次 ID | Java 业务调用点 | Example 条件摘要 | Mapper 方法 | Go contentPath | Go Repository 方法 / QueryFilter | 状态 | 验证结果 | GAP/备注 |
|---|---|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `FooService.listFoo` | `key like ? AND session_id IS NULL` | `FooMapper.selectByExample` | `internal/.../foo_repository.go` | `FooQuery`, `ListFoo` | 已验证 | 测试通过 | `FooExample.java` 仅作为 DSL 来源 |

### Java 重构吸收 / 排除记录

如果某个 Java 文件或符号被 Agent 阅读，但重构后不需要生成对应 Go 文件，必须在这里记录。该记录用于说明“有意不生成 Go 文件”，不是迁移遗漏。

| 批次 ID | Java contentPath | Java 文件/符号 | 处理方式 | Go 承接位置 | 原因 | 验证结果 | GAP/备注 |
|---|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `bic-service/src/.../BicServiceKvExample.java` | `BicServiceKvExample`, `Criteria`, `Criterion` | 重构吸收 | `internal/.../kv_repository.go`: `KVQuery`, `List` | MyBatis Example 是查询 DSL，不生成 Go DSL；实际查询语义由业务调用点和 Repository QueryFilter 承接 | Example 调用点测试通过 | 无 |
| `<CURRENT_BATCH>` | `bic-service/src/.../UnusedExample.java` | `UnusedExample` | 排除不迁移 | 无 | 未发现业务调用点，不臆造 Go 实现 | 人工确认或静态检索确认 | `GAP-XXX` |

### 测试映射

| 批次 ID | Go 被测源码 | Go 测试文件 | 测试 package | 是否同目录 | 测试名称 | 覆盖 Java 行为 | 状态 | GAP/备注 |
|---|---|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `internal/.../foo_repository.go` | `internal/.../unit_foo_repository_test.go` | `repository` | 是 | `<TEST_NAME>` | `<JAVA_BEHAVIOR>` | `<STATUS>` | 无 |

## 变更记录

| 批次 ID | 时间 | Java commit 范围 | 变化文件 | 影响 Go 文件 | 处理状态 | 关联 GAP |
|---|---|---|---|---|---|---|
| `<CURRENT_BATCH>` | `<TIME>` | `<OLD>..<NEW>` | `<JAVA_PATHS>` | `<GO_PATHS>` | `<STATUS>` | `<GAP_IDS>` |
```

### 7.4 映射类型

```text
1:1  一个 Java 文件对应一个主要 Go 文件
1:N  一个 Java 文件拆分为多个 Go 文件
N:1  多个 Java 文件合并为一个 Go 文件
NEW  没有直接 Java 文件来源的 Go 原生实现
ABSORBED  Java 文件/符号不生成 Go 对应文件，其语义被 Go 代码重构吸收
EXCLUDED  Java 文件/符号经确认不迁移，并记录排除原因
```

### 7.5 迁移状态

```text
待分析
迁移中
已迁移
待验证
已验证
GAP
排除
重构吸收
```

### 7.6 更新规则

- 同一个 Java 符号发生状态变化时更新现有记录，不新增重复记录
- 1:N 映射可以按每个 Go 文件拆成多行，但必须使用相同 Java 来源
- N:1 映射可以合并为一行，但必须列出全部 Java 来源
- Go-native 文件使用映射类型 `NEW`，并说明服务于哪个 Java 迁移能力
- Java 文件或符号被 Go 代码重构吸收时，必须使用 `ABSORBED` 或“重构吸收”记录，不得当作遗漏忽略
- Java 文件或符号确认不迁移时，必须使用 `EXCLUDED` 或“排除不迁移”记录原因、证据和是否关联 GAP
- 记录只允许增量更新，不得清空后重新生成

## 8. phase-N-handoff.md 模板

### 8.1 用途

记录当前阶段完成后的稳定成果、验证证据、剩余风险和下一阶段允许依赖的边界。

该文档是自定义的跨阶段交接文档，不是 Superpowers 官方 Execution Handoff。

### 8.2 更新时机

- 当前阶段所有任务完成后创建
- Code Review 完成后更新
- verification-before-completion 通过后补充最终证据
- 阶段重新验收时更新原文件

### 8.3 文件命名

```text
phase-1-persistence-handoff.md
phase-2-api-contract-handoff.md
phase-3-infrastructure-handoff.md
phase-4-runtime-handoff.md
phase-5-service-handoff.md
phase-6-verification-handoff.md
```

### 8.4 模板

```markdown
# 阶段 <N>：<阶段名称>交接文档

## 基线信息

| 字段 | 值 |
|---|---|
| 阶段 | `<PHASE>` |
| 完成批次 | `<BATCH_IDS>` |
| Java 基线分支 | `<JAVA_BRANCH>` |
| Java 基线 commit | `<JAVA_COMMIT_SHA>` |
| Go 分支 | `<GO_BRANCH>` |
| Go commit | `<GO_COMMIT_SHA>` |
| Superpowers design | `<DESIGN_PATH>` |
| Superpowers plan | `<PLAN_PATH>` |
| 完成时间 | `<TIME>` |

## 阶段目标

<阶段原始目标>

## 已完成范围

- `<COMPLETED_ITEM>`

## 未完成和排除范围

- `<NOT_COMPLETED_OR_EXCLUDED>`

## 下一阶段可依赖的稳定成果

| 类型 | contentPath | 关键符号/接口 | 稳定性说明 |
|---|---|---|---|
| Go | `<GO_PATH>` | `<SYMBOL>` | `<STABILITY>` |

## 新增或修改文件

| contentPath | 类型 | Java 来源 | 说明 |
|---|---|---|---|
| `<GO_PATH>` | 新增/修改 | `<JAVA_PATH>` | `<SUMMARY>` |

## 测试和验证证据

| 验证项 | 命令 | 结果 | 证据摘要 |
|---|---|---|---|
| 单元测试 | `<COMMAND>` | 通过/失败 | `<OUTPUT>` |

## Review 结果

- Spec compliance：`<RESULT>`
- Code quality：`<RESULT>`
- 未解决阻塞问题：`<NONE_OR_LIST>`

## 关键 Decisions

- `DEC-XXX`：`<SUMMARY>`

## 剩余 GAP

- `GAP-XXX`：`<SUMMARY>`

## Traceability 状态

- 映射记录总数：`<COUNT>`
- 已验证：`<COUNT>`
- GAP：`<COUNT>`
- 缺失 Java 来源的 Go 文件：`<COUNT>`
- 缺失 Java 来源的 Go 关键符号：`<COUNT>`

## 下一阶段允许假设

- `<ALLOWED_ASSUMPTION>`

## 下一阶段禁止假设

- `<FORBIDDEN_ASSUMPTION>`

## 阶段门禁

- [ ] 阶段范围内工作完成
- [ ] 测试和验证通过
- [ ] Review 阻塞问题关闭
- [ ] Decisions 已更新
- [ ] GAP 已更新
- [ ] Traceability 已更新
- [ ] Go 文件和关键符号来源完整
- [ ] Java 基线 commit 已记录

## 结论

阶段状态：已完成 / 阻塞 / 需要重新验收

下一阶段是否允许启动：是 / 否
```

## 9. Agent 文档维护流程

### 9.1 阶段开始

Agent 必须：

1. 阅读迁移主线文档
2. 阅读本文档
3. 阅读 `migration-roadmap.md`
4. 阅读上一阶段 `handoff.md`
5. 读取当前 Java 和 Go commit
6. 更新当前阶段和批次状态

### 9.2 批次执行中

Agent 必须：

1. 增量更新 `migration-traceability.md`
2. 新决策写入 `migration-decisions.md`
3. 新缺口写入 `migration-gaps.md`
4. 不删除历史记录
5. 不把 Subagent 对话结果当作已验证证据

### 9.3 阶段结束

Agent 必须：

1. 执行 Code Review
2. 执行 verification-before-completion
3. 更新所有治理文档
4. 生成或更新阶段 `handoff.md`
5. 检查阶段门禁
6. 只有门禁全部通过后，才允许在 roadmap 中标记阶段完成

## 10. 最小必填要求

任何治理文档禁止只创建空标题。至少满足：

```text
migration-roadmap.md：六阶段状态 + 当前批次 + 阶段门禁
migration-decisions.md：Decision 索引；没有决策时明确写“暂无”
migration-gaps.md：GAP 索引；没有 GAP 时明确写“暂无”
migration-traceability.md：基线信息 + 文件/符号映射表
phase-N-handoff.md：稳定成果 + 验证证据 + GAP + 下一阶段边界
```

Agent 如果缺少填写某个字段所需的信息，必须标记 `待确认` 或创建 GAP，不得自行编造。
