# bic_service_kv AI 重构执行步骤与提示词

目标：用 `bic_service_kv` 先跑通一条最小 Java -> Go 重构闭环。

本轮只做：

```text
Java BicServiceKvMapper
-> Go KVRepository
-> GORM Raw/Exec 或动态条件构造
-> 真实数据库测试
```

本轮不做：

```text
HTTP handler
Gin router
cache
scheduler
threadpool
strategy
完整 session lock 业务
blocking query 业务
```

---

## 步骤 1：用 Spec Kit 生成规格

执行：

```text
/speckit.specify
```

提示词：

```text
为 bic-service Consul 兼容业务迁移创建 Go Repository / ORM 基础层规格。

源分析报告：
/Users/baoxiaomin/project_code/consul/bic-service-consul-兼容业务块分析报告.md

先从一个试点 Repository 开始：

bic_service_kv

目标：
在迁移 scheduler、threadpool、strategy、cache 之前，先验证完整的 AI 实现工作流是否可行。

范围：
- 将 Java MyBatis 中 BicServiceKvMapper 的行为映射为 Go Repository 方法。
- 保持 SQL 行为和字段语义一致。
- 现在不要实现 HTTP 接口。
- 现在不要实现 cache、scheduler 或 session lock 业务逻辑。
- 只实现 KV 所需的数据库访问基础层。

目标 Go 包：
internal/consulapi/repository

Repository 实现要求：
- 定义 bic_service_kv 的 Go model。
- 定义 Repository interface。
- 可以使用 GORM，但必须先根据 Java MyBatis SQL 判断 SQL 类型。
- 如果 MyBatis 中是静态 SQL，Go 中使用 GORM Raw / Exec / Scan，并保留静态 SQL 常量。
- 如果 MyBatis 中是动态 SQL，例如 `<if>`、`<where>`、`<foreach>`、条件拼接、方言分支等，Go 中按 GORM 动态查询方式重构。
- 动态 SQL 重构时要保持 MyBatis 的条件启用规则、默认值、排序、空值处理和返回行为一致。
- GORM 可以用于连接管理、事务、Raw / Exec / Scan、动态 Where/Clauses 构造和 SQL 日志。
- 不要使用 AutoMigrate。
- 不要依赖 GORM 隐式 CRUD 实现行为敏感方法。
- 静态 SQL 必须以常量形式保留，方便和 Java MyBatis SQL 对照。
- 动态 SQL 必须在 contract 中记录 MyBatis 动态条件与 Go GORM 条件构造的对应关系。
- 优先支持 PostgreSQL。
- SQL Server / 达梦数据库的差异如果暂不实现，需要标记为 GAP。
- 为 Repository 行为添加真实数据库测试。
- 保持 createIndex、modifyIndex、lockIndex、sessionId、flags、key、value 等字段语义一致。

多 subagent 协作规则：
- 请把本任务设计为可由多个 subagent 协作执行，但必须保持职责边界清晰。
- Java Mapper Analyzer：只读取 Java BicServiceKvMapper、Entity、MyBatis SQL，输出 parity contract，不允许写 Go 实现代码。
- Test Designer：读取 parity contract，编写 PostgreSQL schema、seed fixture 和 Go Repository tests，不允许实现 Repository。
- Go Repository Implementer：读取 parity contract 和失败测试，实现 Repository，不允许随意修改测试意图。
- Reviewer：对比 Java contract、Go Repository、测试覆盖，检查 SQL、字段、NULL、排序、affected rows、事务和 GAP。
- 推荐执行顺序是 Analyzer -> Test Designer -> Implementer -> Reviewer。
- 不要让多个 subagent 同时修改同一个文件。
- Analyzer 和 Reviewer 默认只读。
- Test Designer 只能修改测试、fixture 和 contract 中测试相关内容。
- Implementer 只能修改 model、repository 和 SQL 实现。

验收标准：
- Go 代码可以编译通过。
- Repository 测试通过。
- 每个已迁移的 Java Mapper 方法都有 Go Repository 对应方法或明确标记为 GAP。
- 不新增 HTTP handler。
- 不实现无关的 Consul API 行为。
- 不实现 cache、scheduler、threadpool、strategy。
```

然后继续执行：

```text
/speckit.plan
/speckit.tasks
```

---

## 步骤 2：用 GSD 拆执行任务

在 Spec Kit 生成 `tasks.md` 后执行。

提示词：

```text
使用 GSD 管理 bic_service_kv Repository pilot 的执行。

输入任务：
specs/001-bic-service-kv-repository/tasks.md

执行规则：
- 一次只执行一个原子任务。
- 当前阶段只处理 bic_service_kv。
- 不要处理 HTTP handler、cache、scheduler、threadpool、strategy。
- 使用 GSD 调度多个 subagent，但不要让它们自由并行修改同一批文件。
- 按以下顺序执行：Java Mapper Analyzer -> Test Designer -> Go Repository Implementer -> Reviewer。
- 每个 subagent 必须声明输入、输出、允许修改文件、禁止修改文件。
- Analyzer 和 Reviewer 默认只读。
- Test Designer 只能修改测试、fixture 和 contract 中测试相关内容。
- Implementer 只能修改 model、repository 和 SQL 实现。
- 如果 Reviewer 发现实现和 Java contract 不一致，回到 Implementer 修复。
- 如果 Reviewer 发现 contract 本身错误，回到 Analyzer 修正。
- 每个任务必须包含：
  - 任务目标
  - Java 源文件
  - Go 目标文件
  - 允许修改的文件范围
  - 测试要求
  - 验收标准
  - GAP 记录要求
- 每完成一个任务，更新执行进度和未解决问题。

第一批任务优先级：
1. 提取 BicServiceKvMapper parity contract。
2. 将 MyBatis SQL 分类为静态 SQL 和动态 SQL。
3. 准备 PostgreSQL schema 和 seed fixture。
4. 定义 Go model 和 Repository interface。
5. 编写真实数据库 parity tests。
6. 静态 SQL 使用 GORM Raw / Exec / Scan 实现。
7. 动态 SQL 使用 GORM 动态查询方式重构。
8. Review SQL、字段、NULL、排序、affected rows、事务差异。
```

---

## 步骤 3：用 Superpowers 做 TDD 和 Review

对每个 GSD 原子任务使用。

提示词：

```text
对当前 GSD 任务使用 TDD 和 review 流程。

执行顺序：
1. 先读取 Java BicServiceKvMapper 和相关 Entity。
2. 先提取 Mapper parity contract。
3. 判断每个 MyBatis 方法使用的是静态 SQL 还是动态 SQL。
4. 静态 SQL 使用 GORM Raw / Exec / Scan，并保留 SQL 常量。
5. 动态 SQL 按 GORM 动态查询方式重构，例如 Where、Clauses、Scopes 或条件构造函数。
6. 先写 Go Repository 测试。
7. 测试必须使用真实 PostgreSQL，不要只使用 mock。
8. 测试失败后，再实现 Go Repository。
9. 不要使用 AutoMigrate。
10. 不要实现 HTTP、cache、scheduler、threadpool、strategy。
11. 测试通过后 review 差异。

subagent 纪律：
- Analyzer 阶段只做 Java Mapper 分析和 parity contract，不写 Go 实现。
- Test Designer 阶段只写 fixture 和测试，不写 Repository 实现。
- Implementer 阶段只根据失败测试实现代码，不随意改变测试意图。
- Reviewer 阶段只做审查和 GAP 记录，默认不直接改实现。
- TDD 必须保持红绿流程：先有失败测试，再实现，再通过测试。

Review 重点：
- MyBatis 静态 SQL 是否使用 GORM Raw / Exec / Scan 保留。
- MyBatis 动态 SQL 的条件启用规则是否被 GORM 动态查询正确表达。
- SQL 是否和 Java MyBatis 对齐。
- 字段名和 DB 语义是否一致。
- NULL 和空字符串行为是否一致。
- 空结果行为是否一致。
- affected rows 行为是否一致。
- 排序是否一致。
- createIndex / modifyIndex / lockIndex / sessionId / flags / key / value 语义是否一致。
- SQL Server / 达梦差异是否标记为 GAP。
```

---

## 步骤 4：让 Claude Code 实现代码

对当前原子任务执行。

提示词：

```text
只实现当前任务：bic_service_kv Repository pilot。

请先读取：
- 源分析报告：/Users/baoxiaomin/project_code/consul/bic-service-consul-兼容业务块分析报告.md
- Java BicServiceKvMapper
- Java 相关 Entity / DTO
- Java 中涉及 bic_service_kv 的 SQL、事务和方言分支

执行要求：
1. 如果当前角色是 Java Mapper Analyzer：
   - 只生成或更新 Mapper parity contract。
   - 判断 BicServiceKvMapper 中每个方法是静态 SQL 还是动态 SQL。
   - 输出静态 SQL / 动态 SQL 分类结果。
   - 不写 Go 实现代码。
2. 如果当前角色是 Test Designer：
   - 基于 parity contract 编写 Go Repository 测试。
   - 测试使用真实 PostgreSQL。
   - 准备 schema 和 seed fixture。
   - 不写 Repository 实现。
3. 如果当前角色是 Go Repository Implementer：
   - 读取 parity contract 和失败测试。
   - 静态 SQL 使用 GORM Raw / Exec / Scan，并将 SQL 保留为静态常量。
   - 动态 SQL 按 GORM 动态查询方式重构，不要硬编码成不可维护的大段拼接 SQL。
   - 动态 SQL 必须保持 MyBatis 的条件启用规则、空值处理、排序、默认值和返回行为。
   - 不使用 AutoMigrate。
   - 不使用 GORM 隐式 CRUD 实现行为敏感方法。
   - 不随意修改测试意图。
4. 如果当前角色是 Reviewer：
   - 对比 Java contract、Go Repository 和测试覆盖。
   - 检查 SQL、字段、NULL、排序、affected rows、事务和 GAP。
   - 默认只输出 review findings，不直接改代码。
5. 所有角色都必须遵守：
   - 不实现 HTTP handler。
   - 不实现 cache、scheduler、threadpool、strategy。
   - SQL Server / 达梦暂不实现时，必须标记为 GAP。
   - 需要运行 gofmt 和 go test ./... 的阶段必须执行并报告结果。

输出结果：
- 当前执行的是哪个 subagent 角色。
- 修改了哪些文件。
- 哪些 Java Mapper 方法已对齐。
- 哪些 MyBatis SQL 被判定为静态 SQL，使用了 GORM Raw / Exec / Scan。
- 哪些 MyBatis SQL 被判定为动态 SQL，使用了 GORM 动态查询重构。
- 哪些行为已经测试。
- 哪些行为标记为 GAP。
- 是否存在和 Java 行为不一致的风险。
```

---

## 步骤 5：验收本轮 Pilot

检查：

```text
- Spec Kit 已生成 bic_service_kv Repository 规格。
- GSD 已拆成原子任务。
- 已产出 BicServiceKvMapper parity contract。
- Go model 已定义。
- Go KVRepository interface 已定义。
- 静态 MyBatis SQL 使用 GORM Raw / Exec / Scan 实现。
- 动态 MyBatis SQL 使用 GORM 动态查询方式重构。
- 静态 SQL 是常量。
- 动态 SQL 的条件映射已记录在 contract 中。
- Repository 测试使用真实 PostgreSQL。
- gofmt 通过。
- go test ./... 通过。
- 未实现的 SQL Server / 达梦行为已标记为 GAP。
- 没有新增 HTTP handler。
- 没有实现 cache、scheduler、threadpool、strategy。
```

本轮最小代码产物：

```text
internal/consulapi/model/kv.go
internal/consulapi/repository/kv_repository.go
internal/consulapi/repository/kv_repository_gorm.go
internal/consulapi/repository/kv_repository_test.go
test/fixtures/kv/schema_postgres.sql
test/fixtures/kv/seed.sql
specs/001-bic-service-kv-repository/contracts/bic_service_kv_mapper_contract.md
```

---

## 下一步

`bic_service_kv` 跑通后，再复制同样流程：

```text
1. bic_session
2. bic_event
3. bic_service_node_adapt
4. bic_service_node_address_release
5. 健康检查规则表
```

一句话：

```text
先用 bic_service_kv 跑通 AI 重构闭环，再扩大范围。
```
