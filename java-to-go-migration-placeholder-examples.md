# Java 到 Go 迁移提示词占位符填写范本

## 1. 目的

本文档用于说明 `java-to-go-migration-stage-prompts.md` 中常用占位符如何填写。

每次执行 Superpowers 前，先根据当前批次替换占位符，再复制对应阶段提示词。

## 2. 占位符说明

| 占位符 | 含义 | 是否必填 |
|---|---|---|
| `<JAVA_PROJECT_ROOT>` | Java 项目根目录 | 必填 |
| `<GO_PROJECT_ROOT>` | Go 项目根目录 | 必填 |
| `<JAVA_BASELINE_COMMIT>` | 本次迁移参考的 Java commit SHA | 必填 |
| `<CURRENT_BATCH>` | 当前批次 ID，例如 `P1-orm-01` | 必填 |
| `<CURRENT_BATCH_GOAL>` | 当前批次目标 | 必填 |
| `<CURRENT_BATCH_JAVA_SCOPE>` | 当前批次允许分析和迁移的 Java 源码范围 | 必填 |
| `<CURRENT_BATCH_GO_SCOPE>` | 当前批次允许新增或修改的 Go 目录或文件范围 | 建议填写 |
| `<PREVIOUS_BATCH_REFERENCES>` | 前序批次 ID，例如 `P1-orm-01` 或 `P1-orm-01,P1-orm-02` | 第二个批次以后填写 |
| `<IS_FINAL_BATCH_OF_PHASE>` | 当前批次是否是本阶段最后一个批次 | Review 时必须明确 |
| `<PREVIOUS_STAGE_HANDOFF>` | 上一阶段 handoff 文档路径 | 阶段 2 以后必填 |
| `<CUSTOM_INSTRUCTIONS>` | 当前批次额外关注点、特殊约束或人工要求；没有则填写 `无` | 可选 |

默认规则：

- 未列入 `<CURRENT_BATCH_JAVA_SCOPE>` 的 Java 代码，不属于当前批次目标。
- 当前批次不得顺手迁移 scope 外代码。
- 当前批次不得覆盖前序批次的 Superpowers specs/plans。
- `<PREVIOUS_BATCH_REFERENCES>` 只填写前序批次 ID；Agent 负责按批次 ID 自动检索 spec、plan、治理记录和 traceability 中的代码/测试路径。
- `<CUSTOM_INSTRUCTIONS>` 只能补充或收窄当前批次要求，不得覆盖主线、阶段边界和治理规则。
- 只有 `<IS_FINAL_BATCH_OF_PHASE>` 为“是”且阶段全部批次通过 Review 后，才生成 `phase-N-handoff.md`。

## 3. 范本一：阶段 1 ORM 第一个批次

适用场景：

- 阶段 1 第一次执行。
- 先迁移一组表、Entity、Mapper、MyBatis SQL。
- 例如先迁移 `bic_service_kv`。

```text
<JAVA_PROJECT_ROOT>
/Users/baoxiaomin/project_code/consul/bic-service

<GO_PROJECT_ROOT>
/Users/baoxiaomin/project_code/consul/bic-consul-go

<JAVA_BASELINE_COMMIT>
执行 git rev-parse HEAD 后填写，例如：a1b2c3d4e5f6...

<CURRENT_BATCH>
P1-orm-01

<CURRENT_BATCH_GOAL>
迁移 bic_service_kv 相关 Java Entity、MyBatis Mapper、SQL 到 Go GORM Repository，作为阶段 1 ORM 迁移的第一个试点批次。

<CURRENT_BATCH_JAVA_SCOPE>
- bic-service module 中 bic_service_kv 相关 Entity
- BicServiceKvMapper Java 接口
- BicServiceKvMapper 对应 MyBatis XML / 注解 SQL
- 使用 BicServiceKvExample / Criteria / Criterion 的实际业务调用点，只读分析查询语义
- bic_service_kv 表相关 DDL 或测试 SQL
- 只读取必要的公共基础类、分页类、数据库方言类、事务相关类

<CURRENT_BATCH_GO_SCOPE>
- internal/consulapi/repository/
- internal/consulapi/model/ 或项目中已有的持久化 model 目录
- repository 相关测试目录
- spec/migration/
- spec/superpowers/specs/
- spec/superpowers/plans/

<PREVIOUS_BATCH_REFERENCES>
无。当前是阶段 1 第一个批次。

<IS_FINAL_BATCH_OF_PHASE>
否。阶段 1 后续还会继续迁移其他 ORM Entity / Mapper。

<PREVIOUS_STAGE_HANDOFF>
无。当前是阶段 1，不存在上一阶段 handoff。

<CUSTOM_INSTRUCTIONS>
无。
```

## 4. 范本二：阶段 1 ORM 第二个批次

适用场景：

- 阶段 1 已经完成 `P1-orm-01`。
- 当前继续迁移另一组 Entity、Mapper、MyBatis SQL。
- 需要复用前一批次的目录结构、GORM 约束、测试风格和迁移决策。

```text
<JAVA_PROJECT_ROOT>
/Users/baoxiaomin/project_code/consul/bic-service

<GO_PROJECT_ROOT>
/Users/baoxiaomin/project_code/consul/bic-consul-go

<JAVA_BASELINE_COMMIT>
和 P1-orm-01 使用相同 Java 基线时填写同一个 commit SHA；如果 Java 代码有变化，填写新的 commit SHA，并在 migration-traceability.md 记录增量。

<CURRENT_BATCH>
P1-orm-02

<CURRENT_BATCH_GOAL>
迁移另一组 Java Entity、MyBatis Mapper、SQL 到 Go GORM Repository，并复用 P1-orm-01 已建立的 Repository 结构、GORM 约束和测试风格。

<CURRENT_BATCH_JAVA_SCOPE>
- 本批次对应的 Entity Java 文件
- 本批次对应的 Mapper Java 接口
- 本批次对应的 MyBatis XML / 注解 SQL
- 本批次对应 MyBatis Example / Criteria / Criterion 的实际业务调用点，只读分析查询语义
- 本批次相关 DDL 或测试 SQL
- 只读取必要的公共基础类、分页类、数据库方言类、事务相关类

<CURRENT_BATCH_GO_SCOPE>
- internal/consulapi/repository/
- internal/consulapi/model/ 或项目中已有的持久化 model 目录
- repository 相关测试目录
- spec/migration/
- spec/superpowers/specs/
- spec/superpowers/plans/

<PREVIOUS_BATCH_REFERENCES>
P1-orm-01

<IS_FINAL_BATCH_OF_PHASE>
否。除非这是阶段 1 最后一个 ORM 批次，否则不要生成 phase-1-handoff.md。

<PREVIOUS_STAGE_HANDOFF>
无。当前仍然是阶段 1，不存在上一阶段 handoff。

<CUSTOM_INSTRUCTIONS>
无。
```

## 5. 范本三：阶段 1 ORM 最后一个批次

适用场景：

- 当前批次完成后，阶段 1 的 ORM / Repository 范围全部完成。
- Review 通过后，需要生成 `phase-1-persistence-handoff.md`。

```text
<CURRENT_BATCH>
P1-orm-NN

<CURRENT_BATCH_GOAL>
完成阶段 1 最后一组 Java Entity、MyBatis Mapper、SQL 到 Go GORM Repository 的迁移，并对阶段 1 全部批次做完整性收口。

<CURRENT_BATCH_JAVA_SCOPE>
- 阶段 1 最后一组待迁移 Entity / Mapper / MyBatis SQL
- 阶段 1 完整性检查所需读取的前序批次映射记录

<CURRENT_BATCH_GO_SCOPE>
- internal/consulapi/repository/
- internal/consulapi/model/ 或项目中已有的持久化 model 目录
- repository 相关测试目录
- spec/migration/
- spec/superpowers/specs/
- spec/superpowers/plans/

<PREVIOUS_BATCH_REFERENCES>
P1-orm-01,P1-orm-02,...,P1-orm-(NN-1)

<IS_FINAL_BATCH_OF_PHASE>
是。当前批次 Review 通过后，需要执行阶段 1 完整性检查，并生成 phase-1-persistence-handoff.md。

<PREVIOUS_STAGE_HANDOFF>
无。当前仍然是阶段 1，不存在上一阶段 handoff。

<CUSTOM_INSTRUCTIONS>
无。
```

## 6. 范本四：阶段 2 API 第一个批次

适用场景：

- 阶段 1 已经全部完成。
- 已生成 `phase-1-persistence-handoff.md`。
- 当前开始迁移第一组 Controller / Router / Handler / DTO。

```text
<JAVA_PROJECT_ROOT>
/Users/baoxiaomin/project_code/consul/bic-service

<GO_PROJECT_ROOT>
/Users/baoxiaomin/project_code/consul/bic-consul-go

<JAVA_BASELINE_COMMIT>
填写阶段 2 使用的 Java 基线 commit SHA。

<CURRENT_BATCH>
P2-api-01

<CURRENT_BATCH_GOAL>
迁移第一组 Java Controller 接口契约到 Go Gin Router / Handler / DTO，Service 层仅通过 interface 打桩。

<CURRENT_BATCH_JAVA_SCOPE>
- 本批次对应的 Java Controller
- 本批次接口相关 RequestParam / ResponseParam
- 本批次接口相关 Constant / Enum / DTO
- 只读取必要的 Service interface，不实现真实业务逻辑

<CURRENT_BATCH_GO_SCOPE>
- internal/consulapi/router/
- internal/consulapi/handler/
- internal/consulapi/dto/ 或项目中已有的接口参数目录
- internal/consulapi/service/ 中的 interface / stub
- handler / router 相关测试目录
- spec/migration/
- spec/superpowers/specs/
- spec/superpowers/plans/

<PREVIOUS_BATCH_REFERENCES>
无。当前是阶段 2 第一个批次。

<IS_FINAL_BATCH_OF_PHASE>
否。阶段 2 后续还会继续迁移其他 Controller / API 批次。

<PREVIOUS_STAGE_HANDOFF>
<GO_PROJECT_ROOT>/spec/migration/handoffs/phase-1-persistence-handoff.md

<CUSTOM_INSTRUCTIONS>
无。
```

## 7. 快速填写模板

如果只是临时启动一个批次，可以先用下面的精简模板：

```text
<JAVA_PROJECT_ROOT>

<GO_PROJECT_ROOT>

<JAVA_BASELINE_COMMIT>

<CURRENT_BATCH>

<CURRENT_BATCH_GOAL>

<CURRENT_BATCH_JAVA_SCOPE>
- 

<CURRENT_BATCH_GO_SCOPE>
- 

<PREVIOUS_BATCH_REFERENCES>

<IS_FINAL_BATCH_OF_PHASE>
否 / 是

<PREVIOUS_STAGE_HANDOFF>

<CUSTOM_INSTRUCTIONS>
无。
```
