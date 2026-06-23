# Java 到 Go 重构阶段 1-6 启动提示词

## 1. 使用说明

六个阶段必须分别执行 Superpowers，不允许通过一个任务直接完成全部阶段。

这些提示词必须在已经安装并启用 Superpowers 的 Claude Code 环境中执行。Agent 必须实际调用并遵循提示词中指定的 Superpowers skill，不能只使用普通对话模拟同名流程。如果指定 skill 不可用，应停止当前阶段并明确说明，不得跳过后直接实现。

每个阶段的基本流程：

```text
阶段启动提示词
-> brainstorming
-> 人工确认范围和方案
-> writing-plans
-> 分批实现和 TDD
-> code review
-> verification-before-completion
-> 更新 traceability / decisions / gaps
-> 生成 handoff
-> 进入下一阶段
```

所有阶段共同读取：

```text
/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
```

其中：

- `java-to-go-migration-mainline.md` 定义六阶段主线、边界和全局溯源规则。
- `migration-governance-templates.md` 定义所有自定义迁移治理文档的固定结构、状态、编号和增量更新规则。

Agent 创建或更新以下文件时，不得自行发明格式：

```text
migration-roadmap.md
migration-decisions.md
migration-gaps.md
migration-traceability.md
phase-N-handoff.md
```

使用前替换以下占位符：

```text
<JAVA_PROJECT_ROOT>
<GO_PROJECT_ROOT>
<JAVA_BASELINE_COMMIT>
<CURRENT_BATCH>
<PREVIOUS_STAGE_HANDOFF>
```

每个阶段启动时只执行 brainstorming，不立即写代码。人工确认 brainstorming 结果后，再执行本文档第 9 节的通用后续提示词。

---

## 2. 阶段 1：持久化层迁移

```text
请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 1：持久化层迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
<CURRENT_BATCH>

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
6. 阶段测试和行为对齐方案。
7. 主要风险、假设和 GAP。
8. 阶段完成条件。
9. writing-plans 阶段应拆分的任务类型。

不要开始实现。等待我确认 brainstorming 结果后再进入 writing-plans。
```

---

## 3. 阶段 2：API 契约层迁移

```text
请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 2：API 契约层迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 上一阶段交接：<PREVIOUS_STAGE_HANDOFF>

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
<CURRENT_BATCH>

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

## 4. 阶段 3：横切基础设施迁移

```text
请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 3：横切基础设施迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 上一阶段交接：<PREVIOUS_STAGE_HANDOFF>

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
<CURRENT_BATCH>

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

## 5. 阶段 4：并发与生命周期迁移

```text
请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 4：并发与生命周期迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 上一阶段交接：<PREVIOUS_STAGE_HANDOFF>

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
<CURRENT_BATCH>

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

## 6. 阶段 5：Service 业务层迁移

```text
请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 5：Service 业务层迁移。
当前阶段只进行分析和方案确认，不修改 Java 或 Go 代码。

开始前请完整阅读：
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 上一阶段交接：<PREVIOUS_STAGE_HANDOFF>

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前批次：
<CURRENT_BATCH>

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

## 7. 阶段 6：系统级功能校验

```text
请显式调用并遵循 Superpowers 的 brainstorming skill，不要使用普通对话替代该 skill。

当前只执行 Java -> Go 重构的阶段 6：系统级功能校验。
当前阶段只进行验证方案分析，不立即修改代码。

开始前请完整阅读：
- 主线文档：/Users/baoxiaomin/project_code/vibe_prj/plan/java-to-go-migration-mainline.md
- 治理模板：/Users/baoxiaomin/project_code/vibe_prj/plan/migration-governance-templates.md
- 阶段 1-5 的 handoff 文档
- migration-decisions.md
- migration-gaps.md
- migration-traceability.md

所有自定义迁移治理文档必须严格遵循治理模板，不得省略必填字段，不得覆盖无关历史记录。

项目位置：
- Java 项目：<JAVA_PROJECT_ROOT>
- Go 项目：<GO_PROJECT_ROOT>
- Java 基线 commit：<JAVA_BASELINE_COMMIT>

当前校验批次：
<CURRENT_BATCH>

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

## 8. 每阶段通用确认清单

在确认 brainstorming 结果前检查：

```text
1. 是否只覆盖当前阶段？
2. 是否明确列出非目标？
3. 是否读取上一阶段 handoff？
4. 是否记录 Java 基线 commit？
5. 是否规划 Java/Go 文件和符号溯源？
6. 是否规划 migration-traceability.md 更新？
7. 是否定义阶段测试和完成条件？
8. 是否识别风险、假设和 GAP？
9. 是否避免提前实现后续阶段？
```

---

## 9. 人工确认后的通用后续提示词

### 9.1 生成阶段计划

```text
我已确认当前阶段的 brainstorming 结果。

请显式调用并遵循 Superpowers 的 writing-plans skill，将已确认方案转换为可执行计划。不要使用普通任务列表替代该 skill 的计划流程。

要求：
- 只规划当前阶段。
- 按可独立验证的批次拆分任务。
- 每个任务明确输入、输出、允许修改范围、禁止修改范围和验收标准。
- 每个任务包含 Java/Go 文件和关键符号溯源要求。
- 所有自定义治理文档必须遵循 migration-governance-templates.md。
- 每个批次完成后更新 migration-traceability.md。
- 阶段结束时更新 migration-decisions.md、migration-gaps.md 并生成 handoff.md。
- 不要规划后续阶段实现。
```

### 9.2 执行当前阶段计划

```text
请按照已确认的当前阶段计划执行。

请显式调用并遵循 Superpowers 的 subagent-driven-development 和 test-driven-development skills，不要只在文字中声称使用了这些流程。

要求：
- 一次只执行一个已规划任务或批次。
- 先验证或编写测试，再实现代码。
- 不得超出当前阶段范围。
- 不得并行修改同一文件。
- 每个 Go 文件和关键符号必须标注 Java 来源。
- 所有自定义治理文档必须遵循 migration-governance-templates.md，只做增量更新，不覆盖历史记录。
- 每完成一个批次更新 migration-traceability.md、决策和 GAP。
- 每个批次完成后报告测试结果、变更文件和未解决问题。
```

### 9.3 阶段 Review 和完成验证

```text
当前阶段实现任务已经完成。

请显式调用并遵循 Superpowers 的 requesting-code-review 和 verification-before-completion skills，不要跳过 review 或仅以摘要代替验证。

检查：
- 是否符合当前阶段范围和非目标。
- Java/Go 行为是否完成阶段性对齐。
- 测试和验证是否通过。
- Go 文件头和关键符号是否标注 Java 来源。
- 自定义治理文档是否符合 migration-governance-templates.md。
- migration-traceability.md 是否完整。
- migration-decisions.md 和 migration-gaps.md 是否更新。
- 是否存在未解决的阻塞问题。

只有全部阶段门禁通过后，才生成当前阶段 handoff.md，并明确下一阶段可以依赖的稳定成果。
```
