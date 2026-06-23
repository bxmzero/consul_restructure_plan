# Java 到 Go 重构阶段提示词

本文档只保留可复制给 Agent 的提示词。使用方式见：

```text
/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
```

---

## 1. 阶段 1：持久化层迁移

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
JAVA_BASELINE_COMMIT = <填写 Java 基线 commit>
CURRENT_BATCH = <填写当前批次 ID，例如 P1-orm-01>
CURRENT_BATCH_GOAL = <填写当前批次目标>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <无 / 前序批次 spec、plan、traceability、decision、gap>
IS_FINAL_BATCH_OF_PHASE = <是 / 否>
PREVIOUS_STAGE_HANDOFF = <无 / 上一阶段 handoff 路径>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 1：持久化层迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

文档输出位置要求：
- Superpowers specs 输出到：<GO_PROJECT_ROOT>/spec/superpowers/specs/
- Superpowers plans 输出到：<GO_PROJECT_ROOT>/spec/superpowers/plans/
- 自定义迁移治理文档输出到：<GO_PROJECT_ROOT>/spec/migration/
- 如果 Superpowers 因默认行为生成到 docs/superpowers/，不要直接删除或覆盖，先在 migration-roadmap.md 中记录实际路径并等待人工确认。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
- 批次 ID：<CURRENT_BATCH>
- 批次目标：<CURRENT_BATCH_GOAL>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>
- 是否当前阶段最后一个批次：<IS_FINAL_BATCH_OF_PHASE>
- Superpowers spec/plan 文件名必须包含批次 ID，例如 P1-orm-01-design.md、P1-orm-01-plan.md。
- 当前批次只能处理 Java source scope 和 Go target scope 内的内容；scope 外默认不是本批次目标。

阶段目标：
- 将 Java Entity、MyBatis Mapper、SQL 和事务基础能力迁移为 Go 持久化 Model、ORM 和 Repository。
- 保持数据字段、查询结果、写入副作用、事务和数据库方言语义可追溯、可验证。
- 为后续 API 和 Service 迁移提供稳定的数据访问基础。

当前阶段范围：
- Java Entity
- MyBatis Mapper
- 静态 SQL、动态 SQL和数据库方言
- Go 持久化 Model
- Go ORM / Repository
- 事务基础能力
- 数据库错误转换
- 持久化层测试

GORM 实现约束：
- 本阶段 Go ORM / Repository 默认使用 GORM。
- 必须先根据 Java MyBatis SQL 判断 SQL 类型，再决定 Go 侧实现方式。
- 如果 MyBatis 中是静态 SQL，Go 中使用 GORM Raw / Exec / Scan，并保留静态 SQL 常量。
- 如果 MyBatis 中是动态 SQL，例如 `<if>`、`<where>`、`<foreach>`、条件拼接、方言分支等，Go 中按 GORM 动态查询方式重构。
- 动态 SQL 重构时必须保持 MyBatis 的条件启用规则、默认值、排序、空值处理和返回行为一致。
- GORM 可以用于连接管理、事务、Raw / Exec / Scan、动态 Where / Clauses 构造和 SQL 日志。
- 不要使用 AutoMigrate。
- 不要依赖 GORM 隐式 CRUD 实现行为敏感方法。
- 静态 SQL 必须以常量形式保留，方便和 Java MyBatis SQL 对照。
- 动态 SQL 必须在 contract 或 migration-decisions.md 中记录 MyBatis 动态条件与 Go GORM 条件构造的对应关系。
- 优先支持 PostgreSQL。
- SQL Server / 达梦数据库的差异如果暂不实现，需要标记为 GAP。

当前阶段明确不做：
- HTTP Router 和 Handler
- 真实 Service 业务实现
- Cache
- Scheduler
- Thread / ThreadPool
- Notify
- 完整业务链路

必须遵守主线文档中的全局溯源规则：
- Java 到 Go 文件默认 1:1 映射，例外必须记录原因。
- 每个 Go 文件头标注 Java source contentPath。
- Go 函数、结构体、接口、常量等关键符号标注具体 Java 来源。
- 更新 migration-traceability.md。
- 记录 Java 基线 commit SHA。

请在 brainstorming 中输出：
1. 本阶段迁移边界和非目标。
2. Java 持久化文件清单和依赖关系。
3. 建议的迁移批次划分。
4. Java Entity/Mapper 到 Go Model/Repository 的映射策略。
5. 静态 SQL、动态 SQL、事务和数据库方言的处理原则。
6. GORM 实现规则，包括静态 SQL 使用 Raw / Exec / Scan、动态 SQL 使用 GORM 条件构造、禁止 AutoMigrate、禁止依赖隐式 CRUD。
7. 阶段测试和行为对齐方案。
8. 主要风险、假设和 GAP。
9. 阶段完成条件。
10. writing-plans 阶段应拆分的任务类型。

不要开始实现。等待我确认 brainstorming 结果后再进入 writing-plans。
```

---

## 2. 阶段 2：API 契约层迁移

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
JAVA_BASELINE_COMMIT = <填写 Java 基线 commit>
CURRENT_BATCH = <填写当前批次 ID，例如 P2-api-01>
CURRENT_BATCH_GOAL = <填写当前批次目标>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <无 / 前序批次 spec、plan、traceability、decision、gap>
IS_FINAL_BATCH_OF_PHASE = <是 / 否>
PREVIOUS_STAGE_HANDOFF = <上一阶段 handoff 路径>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 2：API 契约层迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 上一阶段交接：<PREVIOUS_STAGE_HANDOFF>

阶段前置条件：
- 阶段 1 持久化层迁移必须已完成全部批次，并生成 phase-1-handoff.md。
- 如果阶段 1 只有部分 ORM 批次完成，不得提前启动阶段 2。
- 当前阶段只生成 API Router / Handler / DTO / Constant / Enum / Service Stub，不回头补做 ORM 批次。

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

文档输出位置要求：
- Superpowers specs 输出到：<GO_PROJECT_ROOT>/spec/superpowers/specs/
- Superpowers plans 输出到：<GO_PROJECT_ROOT>/spec/superpowers/plans/
- 自定义迁移治理文档输出到：<GO_PROJECT_ROOT>/spec/migration/
- 如果 Superpowers 因默认行为生成到 docs/superpowers/，不要直接删除或覆盖，先在 migration-roadmap.md 中记录实际路径并等待人工确认。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
- 批次 ID：<CURRENT_BATCH>
- 批次目标：<CURRENT_BATCH_GOAL>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>
- 是否当前阶段最后一个批次：<IS_FINAL_BATCH_OF_PHASE>
- Superpowers spec/plan 文件名必须包含批次 ID，例如 P2-api-01-design.md、P2-api-01-plan.md。
- 当前批次只能处理 Java source scope 和 Go target scope 内的内容；scope 外默认不是本批次目标。

阶段目标：
- 将 Java Controller 对外契约迁移为 Go Router 和 Handler。
- 迁移接口层 Request、Response、DTO、Constant 和 Enum。
- 通过小粒度 Service Interface 和 Fake/Stub 固定 Handler 依赖边界。
- 在真实 Service 尚未实现时完成 API 契约验证。

当前阶段范围：
- Controller 路由
- HTTP Method 和 Path
- Request Parameter
- Response Parameter
- 传输层 DTO
- 接口层 Constant 和 Enum
- Router 和 Handler
- Service Interface
- Service Fake / Stub
- API 契约测试

当前阶段明确不做：
- 真实 Service 业务实现
- 完整数据库业务链路
- Scheduler 和后台任务
- 完整 Cache 行为
- 完整 Notify 行为

边界要求：
- 只迁移接口契约相关 DTO、Constant 和 Enum。
- 业务领域模型和业务规则留到阶段 5。
- Stub 只用于验证接口契约，不能成为最终业务实现。
- 不得修改阶段 1 已验收的 Repository 契约，除非记录变更原因并重新验证。

必须遵守主线文档中的文件、符号和增量映射规则。

请在 brainstorming 中输出：
1. 本阶段接口范围和非目标。
2. Java Controller 与 Go Router/Handler 的映射方案。
3. Request、Response、DTO、Constant、Enum 的边界划分。
4. Service Interface 和 Stub 的职责边界。
5. 建议的接口迁移批次。
6. API 契约验证方案。
7. 与阶段 1 的依赖和影响。
8. 主要风险、假设和 GAP。
9. 阶段完成条件。

不要开始实现。等待我确认 brainstorming 结果后再进入 writing-plans。
```

---

## 3. 阶段 3：横切基础设施迁移

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
JAVA_BASELINE_COMMIT = <填写 Java 基线 commit>
CURRENT_BATCH = <填写当前批次 ID，例如 P3-infra-01>
CURRENT_BATCH_GOAL = <填写当前批次目标>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <无 / 前序批次 spec、plan、traceability、decision、gap>
IS_FINAL_BATCH_OF_PHASE = <是 / 否>
PREVIOUS_STAGE_HANDOFF = <上一阶段 handoff 路径>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 3：横切基础设施迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 上一阶段交接：<PREVIOUS_STAGE_HANDOFF>

阶段前置条件：
- 阶段 2 API 契约层迁移必须已完成全部批次，并生成 phase-2-handoff.md。
- 如果阶段 2 只有部分 API 批次完成，不得提前启动阶段 3。
- 当前阶段只迁移横切基础设施，不回头补做 ORM 或 API 批次。

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

文档输出位置要求：
- Superpowers specs 输出到：<GO_PROJECT_ROOT>/spec/superpowers/specs/
- Superpowers plans 输出到：<GO_PROJECT_ROOT>/spec/superpowers/plans/
- 自定义迁移治理文档输出到：<GO_PROJECT_ROOT>/spec/migration/
- 如果 Superpowers 因默认行为生成到 docs/superpowers/，不要直接删除或覆盖，先在 migration-roadmap.md 中记录实际路径并等待人工确认。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
- 批次 ID：<CURRENT_BATCH>
- 批次目标：<CURRENT_BATCH_GOAL>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>
- 是否当前阶段最后一个批次：<IS_FINAL_BATCH_OF_PHASE>
- Superpowers spec/plan 文件名必须包含批次 ID，例如 P3-infra-01-design.md、P3-infra-01-plan.md。
- 当前批次只能处理 Java source scope 和 Go target scope 内的内容；scope 外默认不是本批次目标。

阶段目标：
- 迁移被多个业务模块共同依赖的横切能力。
- 为阶段 5 的真实 Service 实现提供稳定的公共接口。

当前阶段范围：
- Interceptor 到 Middleware 的迁移
- 统一错误处理
- 鉴权和 Token
- Cache 抽象及实现
- Cache 失效机制
- Notify 消息通知抽象
- 日志、Tracing 和公共上下文
- 公共配置能力

当前阶段明确不做：
- 完整 Scheduler 实现
- Thread / ThreadPool 模型
- 容器完整启动关闭流程
- 具体业务 Service 编排
- 完整端到端业务链路

边界要求：
- 横切能力应通过稳定接口暴露，不直接承载具体业务流程。
- 不得绕过阶段 2 已建立的 API 契约。
- 不得提前实现阶段 4 的并发和生命周期模型。
- 不得提前实现阶段 5 的业务编排。

必须遵守主线文档中的文件、符号和增量映射规则。

请在 brainstorming 中输出：
1. 横切能力清单和 Java 来源。
2. Java Interceptor、Cache、Notify、错误和鉴权到 Go 能力的映射方案。
3. 公共接口边界和依赖方向。
4. 建议的迁移批次。
5. 阶段测试和隔离验证方案。
6. 与阶段 1、阶段 2 的依赖和影响。
7. 主要风险、假设和 GAP。
8. 阶段完成条件。

不要开始实现。等待我确认 brainstorming 结果后再进入 writing-plans。
```

---

## 4. 阶段 4：并发与生命周期迁移

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
JAVA_BASELINE_COMMIT = <填写 Java 基线 commit>
CURRENT_BATCH = <填写当前批次 ID，例如 P4-runtime-01>
CURRENT_BATCH_GOAL = <填写当前批次目标>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <无 / 前序批次 spec、plan、traceability、decision、gap>
IS_FINAL_BATCH_OF_PHASE = <是 / 否>
PREVIOUS_STAGE_HANDOFF = <上一阶段 handoff 路径>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 4：并发与生命周期迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 上一阶段交接：<PREVIOUS_STAGE_HANDOFF>

阶段前置条件：
- 阶段 3 横切基础设施迁移必须已完成全部批次，并生成 phase-3-handoff.md。
- 如果阶段 3 只有部分批次完成，不得提前启动阶段 4。
- 当前阶段只迁移并发、调度、线程池、策略和生命周期模型，不回头补做 ORM、API 或横切基础设施批次。

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

文档输出位置要求：
- Superpowers specs 输出到：<GO_PROJECT_ROOT>/spec/superpowers/specs/
- Superpowers plans 输出到：<GO_PROJECT_ROOT>/spec/superpowers/plans/
- 自定义迁移治理文档输出到：<GO_PROJECT_ROOT>/spec/migration/
- 如果 Superpowers 因默认行为生成到 docs/superpowers/，不要直接删除或覆盖，先在 migration-roadmap.md 中记录实际路径并等待人工确认。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
- 批次 ID：<CURRENT_BATCH>
- 批次目标：<CURRENT_BATCH_GOAL>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>
- 是否当前阶段最后一个批次：<IS_FINAL_BATCH_OF_PHASE>
- Superpowers spec/plan 文件名必须包含批次 ID，例如 P4-runtime-01-design.md、P4-runtime-01-plan.md。
- 当前批次只能处理 Java source scope 和 Go target scope 内的内容；scope 外默认不是本批次目标。

阶段目标：
- 将 Java Scheduler、Thread、ThreadPool、Strategy 和容器生命周期迁移为可控制、可关闭、可验证的 Go 运行时模型。
- 明确启动、运行、取消、关闭和错误传播行为。

当前阶段范围：
- Scheduler
- Thread / ThreadPool
- Worker 和任务队列
- Strategy
- 启动 Loader
- 启动和关闭顺序
- Graceful Shutdown
- Context Cancellation
- 后台错误传播
- 资源释放
- 并发和生命周期测试

当前阶段明确不做：
- 完整 Service 业务实现
- 所有接口业务链路打通
- 系统级 Java/Go 最终校验

边界要求：
- 当前阶段只建立运行时模型和稳定接口。
- 具体业务编排留到阶段 5。
- 必须保留 Java 中影响业务结果的调度、顺序、重试、超时和关闭语义。
- 不以机械复制 Java 线程模型为目标，但任何行为变化必须记录。

必须遵守主线文档中的文件、符号和增量映射规则。

请在 brainstorming 中输出：
1. Java 并发和生命周期组件清单。
2. Scheduler、Thread、ThreadPool、Strategy、Loader 到 Go 运行时模型的映射方案。
3. 启动、运行、取消、关闭和错误传播模型。
4. 建议的迁移批次和依赖顺序。
5. 并发、资源释放和生命周期验证方案。
6. 与阶段 3 横切能力的依赖关系。
7. 主要风险、假设和 GAP。
8. 阶段完成条件。

不要开始实现。等待我确认 brainstorming 结果后再进入 writing-plans。
```

---

## 5. 阶段 5：Service 业务层迁移

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
JAVA_BASELINE_COMMIT = <填写 Java 基线 commit>
CURRENT_BATCH = <填写当前批次 ID，例如 P5-service-01>
CURRENT_BATCH_GOAL = <填写当前批次目标>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <无 / 前序批次 spec、plan、traceability、decision、gap>
IS_FINAL_BATCH_OF_PHASE = <是 / 否>
PREVIOUS_STAGE_HANDOFF = <上一阶段 handoff 路径>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 5：Service 业务层迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 上一阶段交接：<PREVIOUS_STAGE_HANDOFF>

阶段前置条件：
- 阶段 1-4 必须已完成全部批次，并分别生成 phase-1 到 phase-4 handoff。
- 如果阶段 4 只有部分批次完成，不得提前启动阶段 5。
- 当前阶段只实现 Service 业务链路，不回头补做 ORM、API、横切基础设施或运行时模型批次。

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

文档输出位置要求：
- Superpowers specs 输出到：<GO_PROJECT_ROOT>/spec/superpowers/specs/
- Superpowers plans 输出到：<GO_PROJECT_ROOT>/spec/superpowers/plans/
- 自定义迁移治理文档输出到：<GO_PROJECT_ROOT>/spec/migration/
- 如果 Superpowers 因默认行为生成到 docs/superpowers/，不要直接删除或覆盖，先在 migration-roadmap.md 中记录实际路径并等待人工确认。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
- 批次 ID：<CURRENT_BATCH>
- 批次目标：<CURRENT_BATCH_GOAL>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>
- 是否当前阶段最后一个批次：<IS_FINAL_BATCH_OF_PHASE>
- Superpowers spec/plan 文件名必须包含批次 ID，例如 P5-service-01-design.md、P5-service-01-plan.md。
- 当前批次只能处理 Java source scope 和 Go target scope 内的内容；scope 外默认不是本批次目标。

阶段目标：
- 实现真实 Service 业务逻辑。
- 连接 Router/Handler、Repository、Cache、Notify、Scheduler、Worker 和生命周期能力。
- 替换阶段 2 的 Service Fake/Stub，按业务链路逐批打通系统。

当前阶段范围：
- Service Interface 的真实实现
- 业务规则
- 事务编排
- Repository 调用
- Cache 调用
- Notify 调用
- Scheduler / Worker 协作
- Handler 到数据层的完整业务链路
- Fake / Stub 替换
- Service 和业务链路测试

当前阶段明确不做：
- 无关业务能力扩展
- 未经确认的架构重写
- 为追求理想设计而改变现有对外行为
- 尚未进入迁移范围的 Java 业务模块

边界要求：
- 以完整业务能力作为迁移批次，不以 Java 类数量作为批次依据。
- 复用阶段 1-4 已验收的接口和实现。
- 修改前置阶段契约时必须记录原因、影响并重新验证。
- Java 业务规则和实际行为是最终事实。

必须遵守主线文档中的文件、符号和增量映射规则。

请在 brainstorming 中输出：
1. Service 业务能力和完整调用链清单。
2. 建议的纵向业务迁移批次。
3. Java Service 到 Go Service 的职责映射。
4. 事务、Cache、Notify、Scheduler 和 Worker 的协作边界。
5. Stub 替换顺序。
6. 业务规则和副作用验证方案。
7. 与阶段 1-4 的依赖和可能影响。
8. 主要风险、假设和 GAP。
9. 阶段完成条件。

不要开始实现。等待我确认 brainstorming 结果后再进入 writing-plans。
```

---

## 6. 阶段 6：系统级功能校验

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
JAVA_BASELINE_COMMIT = <填写 Java 基线 commit>
CURRENT_BATCH = <填写当前批次 ID，例如 P6-verify-01>
CURRENT_BATCH_GOAL = <填写当前批次目标>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <前序批次 spec、plan、traceability、decision、gap>
IS_FINAL_BATCH_OF_PHASE = <是 / 否>
PREVIOUS_STAGE_HANDOFF = <阶段 1-5 handoff 路径>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 6：系统级功能校验。
当前阶段只进行验证方案分析，不立即修改代码。

开始前请完整阅读：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 阶段 1-5 的 handoff 文档
- migration-decisions.md
- migration-gaps.md
- migration-traceability.md

阶段前置条件：
- 阶段 1-5 必须已完成全部批次，并分别生成 phase-1 到 phase-5 handoff。
- 如果阶段 5 只有部分业务批次完成，不得提前启动阶段 6。
- 当前阶段只做系统级功能校验和差异分类，不隐式重写阶段 1-5 的核心设计。

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

文档输出位置要求：
- Superpowers specs 输出到：<GO_PROJECT_ROOT>/spec/superpowers/specs/
- Superpowers plans 输出到：<GO_PROJECT_ROOT>/spec/superpowers/plans/
- 自定义迁移治理文档输出到：<GO_PROJECT_ROOT>/spec/migration/
- 如果 Superpowers 因默认行为生成到 docs/superpowers/，不要直接删除或覆盖，先在 migration-roadmap.md 中记录实际路径并等待人工确认。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前校验批次：
- 批次 ID：<CURRENT_BATCH>
- 批次目标：<CURRENT_BATCH_GOAL>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>
- 是否当前阶段最后一个批次：<IS_FINAL_BATCH_OF_PHASE>
- Superpowers spec/plan 文件名必须包含批次 ID，例如 P6-verify-01-design.md、P6-verify-01-plan.md。
- 当前批次只能验证 Java source scope 和 Go target scope 内的功能；不得隐式扩大到未声明范围。

阶段目标：
- 从系统和外部调用方视角验证 Java 与 Go 的功能和行为是否一致。
- 判断 Go 实现是否具备灰度、回滚和替换条件。

当前阶段范围：
- Java/Go 同请求对比
- HTTP 状态码、Body 和 Header
- 错误行为
- 数据库副作用
- Cache 副作用
- Notify 消息副作用
- 并发和 Scheduler 行为
- 启动和关闭行为
- 性能和稳定性
- 文件和符号溯源完整性

当前阶段明确不做：
- 未经确认的大规模架构重写
- 新增无关业务功能
- 为了让测试通过而改变既定业务契约
- 未评估影响的批量修复

边界要求：
- 验证失败时先记录差异和根因，再决定是否修复。
- 必要修复必须回到对应责任阶段，重新执行测试、review 和阶段门禁。
- 不得在阶段 6 隐式重写前面阶段的核心设计。
- 所有 Java/Go 差异必须分类为缺陷、可接受差异、GAP 或排除项。

请在 brainstorming 中输出：
1. 系统级验证范围和优先级。
2. Java/Go 对比矩阵。
3. 接口、数据库、Cache、Notify、并发和生命周期验证方案。
4. 验证数据、环境和基线要求。
5. 差异分类和回退处理流程。
6. 需要回到阶段 1-5 修复的判定标准。
7. 灰度、回滚和替换准入条件。
8. 主要风险、假设和剩余 GAP。
9. 最终验收报告结构。

不要立即修改代码。等待我确认验证方案后再进入 writing-plans。
```

---

## 7. 人工确认后的通用后续提示词

### 7.1 生成当前批次计划

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
CURRENT_BATCH = <填写当前批次 ID>
CURRENT_BATCH_GOAL = <填写当前批次目标>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <无 / 前序批次 spec、plan、traceability、decision、gap>
IS_FINAL_BATCH_OF_PHASE = <是 / 否>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

我已确认当前阶段的 brainstorming 结果。

请显式调用并遵循 Superpowers 的 writing-plans skill，将已确认方案转换为可执行计划。不要使用普通任务列表替代该 skill 的计划流程。

当前只为一个批次生成计划：
- 批次 ID：<CURRENT_BATCH>
- 批次目标：<CURRENT_BATCH_GOAL>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>

继续遵守：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- Superpowers plans 输出到：<GO_PROJECT_ROOT>/spec/superpowers/plans/
- 自定义迁移治理文档输出到：<GO_PROJECT_ROOT>/spec/migration/

要求：
- 只规划当前批次，不规划当前阶段的其他批次。
- 不规划后续阶段实现。
- 如果发现当前 scope 仍然过大，只提出进一步拆分建议，不在本次计划中扩大范围。
- 每个任务明确输入、输出、允许修改范围、禁止修改范围和验收标准。
- 每个任务包含 Java/Go 文件和关键符号溯源要求。
- 如果当前阶段涉及 ORM / Repository，必须把 GORM 实现约束写入计划任务和验收标准。
- 所有自定义治理文档必须遵循 migration-governance-templates.md。
- 每个批次完成后更新 migration-traceability.md。
- 批次结束时更新 migration-roadmap.md、migration-decisions.md、migration-gaps.md。
- 只有 <IS_FINAL_BATCH_OF_PHASE> 为“是”且本阶段所有批次都通过 Review 时，才规划生成 phase-N-handoff.md。
```

### 7.2 执行当前批次计划

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
CURRENT_BATCH = <填写当前批次 ID>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <无 / 前序批次 spec、plan、traceability、decision、gap>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

请按照已确认的当前阶段计划执行。

请显式调用并遵循 Superpowers 的 subagent-driven-development 和 test-driven-development skills，不要只在文字中声称使用了这些流程。

当前只执行一个批次：
- 批次 ID：<CURRENT_BATCH>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>

继续遵守：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 当前阶段计划文档必须来自：<GO_PROJECT_ROOT>/spec/superpowers/plans/
- 自定义迁移治理文档输出到：<GO_PROJECT_ROOT>/spec/migration/

要求：
- 只执行当前批次计划。
- 先验证或编写测试，再实现代码。
- 不得超出当前阶段范围。
- 不得超出当前批次 Java source scope 和 Go target scope。
- 不得修改前序批次已验收成果，除非记录 Decision/GAP、影响范围和重新验证结果。
- 不得并行修改同一文件。
- 如果当前阶段涉及 ORM / Repository，必须使用 GORM，并遵守静态 SQL 使用 Raw / Exec / Scan、动态 SQL 使用 GORM 条件构造、禁止 AutoMigrate、禁止依赖隐式 CRUD。
- 每个 Go 文件和关键符号必须标注 Java 来源。
- 所有自定义治理文档必须遵循 migration-governance-templates.md，只做增量更新，不覆盖历史记录。
- 每完成一个批次更新 migration-traceability.md、决策和 GAP。
- 每个批次完成后报告测试结果、变更文件和未解决问题。
```

### 7.3 批次 Review 和完成验证

```text
本次输入值：
JAVA_PROJECT_ROOT = <填写 Java 项目根目录>
GO_PROJECT_ROOT = <填写 Go 项目根目录>
CURRENT_BATCH = <填写当前批次 ID>
CURRENT_BATCH_JAVA_SCOPE = <填写当前批次 Java source scope>
CURRENT_BATCH_GO_SCOPE = <填写当前批次 Go target scope>
PREVIOUS_BATCH_REFERENCES = <无 / 前序批次 spec、plan、traceability、decision、gap>
IS_FINAL_BATCH_OF_PHASE = <是 / 否>

说明：
- 下文所有 `<...>` 占位符均使用“本次输入值”中的对应值。
- 如果某项输入值为空或不明确，请先暂停并要求我补充，不要自行猜测。

当前批次实现任务已经完成。

请显式调用并遵循 Superpowers 的 requesting-code-review 和 verification-before-completion skills，不要跳过 review 或仅以摘要代替验证。

当前 Review 类型：
- 批次 ID：<CURRENT_BATCH>
- 是否当前阶段最后一个批次：<IS_FINAL_BATCH_OF_PHASE>
- Java source scope：<CURRENT_BATCH_JAVA_SCOPE>
- Go target scope：<CURRENT_BATCH_GO_SCOPE>
- 前序批次参考：<PREVIOUS_BATCH_REFERENCES>

继续遵守：
- 提示词使用说明：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-prompt-usage.md
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- Superpowers specs/plans 位置：<GO_PROJECT_ROOT>/spec/superpowers/
- 自定义迁移治理文档位置：<GO_PROJECT_ROOT>/spec/migration/

检查：
- 是否符合当前阶段范围和非目标。
- 是否只覆盖当前批次 scope，没有顺手迁移其他批次或后续阶段。
- 是否覆盖当前批次对应的 Java 文件、Java 符号、Go 文件、Go 关键符号和测试。
- Java/Go 行为是否完成阶段性对齐。
- 如果当前阶段涉及 ORM / Repository，是否遵守 GORM 实现约束，包括静态 SQL、动态 SQL、AutoMigrate、隐式 CRUD、SQL 常量和 GAP 记录。
- 测试和验证是否通过。
- Go 文件头和关键符号是否标注 Java 来源。
- 自定义治理文档是否符合 migration-governance-templates.md。
- migration-traceability.md 是否完整。
- migration-decisions.md 和 migration-gaps.md 是否更新。
- 是否存在未解决的阻塞问题。

如果 <IS_FINAL_BATCH_OF_PHASE> 为“否”：
- 只做批次 Review。
- 更新 migration-roadmap.md、migration-traceability.md、migration-decisions.md、migration-gaps.md。
- 不生成 phase-N-handoff.md。
- 明确下一批次需要读取的前序批次参考。

如果 <IS_FINAL_BATCH_OF_PHASE> 为“是”：
- 对本阶段所有批次执行阶段级完整性检查。
- 确认本阶段所有批次的测试、Review、traceability、Decision、GAP 都已闭环。
- 只有全部阶段门禁通过后，才生成当前阶段 phase-N-handoff.md。
- handoff 中必须明确下一阶段可以依赖的稳定成果和不得依赖的 GAP。
```
