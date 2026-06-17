# Java Vert.x + Guice 到 Go Gin 重构迁移计划

## 1. 迁移目标

本计划用于将现有 Java Vert.x + Guice 项目重构/迁移为 Go + Gin 项目。

核心原则不是把 Java 代码逐行翻译成 Go，而是先恢复旧系统的真实行为规格，再基于规格实现 Go 版本。

迁移目标：

- 保持接口行为一致
- 保持 HTTP status code 一致
- 保持 JSON 字段、错误格式、鉴权、校验逻辑一致
- 保持数据库 schema 不变，除非明确批准修改
- 每个迁移接口必须有 parity test
- 按 endpoint / route group 分阶段迁移，避免 big-bang rewrite

## 2. 工具分工

| 工具 | 角色 | 是否核心 | 用途 |
| --- | --- | --- | --- |
| Reversa | 旧系统理解层 | 核心 | 从 Java 遗留系统恢复 operational specifications |
| GitHub Spec Kit | 目标规格层 | 核心 | 把 Go Gin 目标系统定义为 spec / plan / tasks |
| GSD / Get Shit Done | 长任务编排层 | 核心 | 拆分长迁移任务，管理上下文，防止上下文腐烂 |
| Claude Code 自定义 skills | 项目迁移执行层 | 核心 | 固化 Vert.x + Guice -> Gin 的项目级迁移规则 |
| Superpowers | 单任务工程纪律层 | 辅助 | TDD、debug、review、worktree 隔离 |

推荐主线：

```text
Reversa -> Spec Kit -> GSD -> Claude Code 自定义 skills -> Superpowers 辅助 TDD/review
```

## 3. 推荐目录结构

```text
.
├── CLAUDE.md
├── docs/
│   ├── java-vertx-guice-to-go-gin-migration-plan.md
│   ├── legacy/
│   │   ├── reversa-output/
│   │   ├── route-map.md
│   │   ├── guice-bindings.md
│   │   ├── api-contracts.md
│   │   ├── error-model.md
│   │   ├── db-access-map.md
│   │   └── external-clients.md
│   └── migration/
│       ├── execution-log.md
│       ├── parity-checklist.md
│       └── rollout-plan.md
├── specs/
│   ├── 000-constitution/
│   ├── 001-go-gin-foundation/
│   ├── 002-auth-middleware/
│   ├── 003-user-api-migration/
│   └── 004-order-api-migration/
├── .claude/
│   └── skills/
│       ├── migration-inventory/
│       ├── vertx-guice-to-gin-slice/
│       ├── api-parity-test/
│       └── migration-review/
└── go-service/
    ├── cmd/server/main.go
    ├── internal/config/
    ├── internal/http/
    ├── internal/service/
    ├── internal/repository/
    ├── internal/client/
    ├── internal/model/
    └── internal/app/
```

## 4. 阶段 0：项目准备

### 目标

建立迁移约束、基础目录和 AI 执行规则。

### 产物

```text
CLAUDE.md
docs/java-vertx-guice-to-go-gin-migration-plan.md
docs/legacy/
docs/migration/
.claude/skills/
go-service/
```

### CLAUDE.md 建议内容

```md
# Migration Goal

Source: Java Vert.x + Guice
Target: Go + Gin

## Rules

- Migrate endpoint by endpoint.
- Preserve HTTP status codes, JSON fields, validation behavior, auth behavior, and error format.
- Do not change database schema unless explicitly approved.
- Every migrated endpoint must have parity tests.
- Prefer constructor injection in Go.
- Do not introduce a Go DI framework unless needed.
- Do not rewrite unrelated modules.
- Do not simplify error handling silently.
- Record unverifiable behavior in docs/migration/execution-log.md.
```

### 验收标准

- 项目迁移规则明确
- 文档目录创建完成
- 后续 AI 任务可以引用统一上下文

## 5. 阶段 1：Reversa 逆向旧系统规格

### 目标

从 Java Vert.x + Guice 项目恢复系统真实行为，而不是依赖人工记忆或代码猜测。

### 分析范围

```text
1. Vert.x routes
2. Handler -> Service -> Repository 调用链
3. Guice Module / binding
4. DTO / JSON 字段
5. 参数校验逻辑
6. 鉴权与权限判断
7. 错误响应格式
8. DB 表、SQL、事务边界
9. 外部 API client
10. EventBus / Scheduler / 异步任务
```

### 输出文档

```text
docs/legacy/route-map.md
docs/legacy/guice-bindings.md
docs/legacy/api-contracts.md
docs/legacy/error-model.md
docs/legacy/db-access-map.md
docs/legacy/external-clients.md
docs/legacy/async-jobs.md
```

### 每个 endpoint 必须回答

```text
- method/path 是什么？
- path/query/header/body 参数是什么？
- 参数是否必填？
- 参数校验失败返回什么？
- 鉴权失败返回什么？
- 成功响应 JSON 长什么样？
- 错误响应 JSON 长什么样？
- status code 是什么？
- 调用了哪些 handler/service/repository/client？
- 访问了哪些表？
- 是否有事务？
- 是否有异步副作用？
```

### 验收标准

- 主要接口全部进入 `route-map.md`
- 每个接口都有可执行的行为契约
- 关键 Guice binding 已映射到 Go 构造依赖
- 错误模型、鉴权模型、DTO 字段已记录

## 6. 阶段 2：GitHub Spec Kit 定义目标 Go Gin 规格

### 目标

把 Reversa 恢复出的旧系统行为，转换成 Go Gin 目标系统的正式规格。

### 初始化命令

```bash
specify init --here --force --integration claude
```

如果使用 Codex：

```bash
specify init --here --force --integration codex
```

### 推荐 Spec Kit 流程

```text
/speckit.constitution
/speckit.specify
/speckit.clarify
/speckit.plan
/speckit.tasks
/speckit.analyze
/speckit.implement
```

### specs 建议拆分

```text
specs/
  000-constitution/
  001-go-gin-foundation/
  002-auth-middleware/
  003-error-model/
  004-user-api-migration/
  005-order-api-migration/
  006-external-client-migration/
  007-async-job-migration/
```

### Go Gin 目标结构

```text
go-service/
  cmd/server/main.go
  internal/config/
  internal/http/router.go
  internal/http/handler/
  internal/http/middleware/
  internal/service/
  internal/repository/
  internal/client/
  internal/model/
  internal/app/wire.go
```

### Vert.x + Guice 到 Go Gin 映射

| Java / Vert.x / Guice | Go / Gin |
| --- | --- |
| Vert.x Router | Gin router groups |
| RoutingContext | *gin.Context |
| Handler class | handler struct |
| Service class | service struct |
| Repository class | repository struct |
| Guice @Inject | constructor injection |
| Guice Module | internal/app/wire.go |
| Future / Promise | context.Context + goroutine / errgroup |
| JsonObject / DTO | Go struct + json tags |
| Exception handler | Gin error middleware |
| WebClient | net/http, resty, or typed client |
| JUnit / Mockito | go test + httptest + testify/mock |

### 验收标准

- 每个 Spec Kit task 可以直接变成一个 Claude Code 执行任务
- 每个 task 有明确输入、输出、验收标准
- 目标系统结构、错误模型、鉴权模型、依赖注入方式已固定

## 7. 阶段 3：GSD 拆分长任务

### 目标

把大迁移拆成 atomic tasks，避免长上下文里丢失前面结论。

### 原则

```text
一个任务 = 一个 endpoint / 一个 middleware / 一个 client / 一个 repository 能力
```

### 推荐执行顺序

```text
1. Go Gin 基础工程
2. 配置加载
3. 日志
4. health/version endpoint
5. 统一错误模型
6. 鉴权 middleware
7. 只读查询接口
8. 简单写接口
9. 复杂事务接口
10. 外部 API client
11. 异步任务 / EventBus 替代方案
12. 全量 parity review
```

### 每个 GSD task 必须包含

```text
- 背景
- 输入 Java 文件
- 目标 Go 文件
- 旧 Java 行为
- Go 实现要求
- 测试要求
- 验收标准
- 不允许修改的范围
```

### 单任务模板

```md
# Task: Migrate GET /api/users/:id

## Background

Migrate one read-only endpoint from Java Vert.x + Guice to Go Gin.

## Java Inputs

- UserRoute.java
- UserHandler.java
- UserService.java
- UserRepository.java
- UserDto.java

## Go Targets

- go-service/internal/http/handler/user_handler.go
- go-service/internal/service/user_service.go
- go-service/internal/repository/user_repository.go
- go-service/internal/model/user.go
- go-service/internal/http/handler/user_handler_test.go

## Required Behavior

- Preserve route path and method.
- Preserve success JSON.
- Preserve not-found behavior.
- Preserve validation errors.
- Preserve auth behavior.

## Acceptance Criteria

- gofmt passes.
- go test ./... passes.
- httptest covers success, not found, invalid id, unauthorized.
- JSON field names match Java response.
```

## 8. 阶段 4：Claude Code 自定义 Skills

### 目标

把项目级迁移方法固化，避免每次都重新解释 Vert.x + Guice 到 Go Gin 的迁移规则。

### 推荐 skills

```text
.claude/skills/
  migration-inventory/
  vertx-guice-to-gin-slice/
  api-parity-test/
  migration-review/
```

### migration-inventory/SKILL.md

```md
---
description: Analyze Java Vert.x + Guice project and produce migration inventory.
---

# Goal

Analyze the Java project without editing files.

# Output

- route map
- Guice binding map
- handler/service/repository dependency map
- DTO and JSON field map
- error model
- auth model
- DB access map
- external client map

# Constraints

- Do not edit code.
- Prefer precise file and class references.
- Record uncertainty explicitly.
```

### vertx-guice-to-gin-slice/SKILL.md

```md
---
description: Migrate one Java Vert.x + Guice endpoint slice to Go Gin.
---

# Goal

Migrate exactly one endpoint or one small route group.

# Procedure

1. Read Java route, handler, service, repository, DTO, validators, and tests.
2. Document current behavior:
   - method/path
   - path/query/body/header params
   - auth requirements
   - validation failures
   - success response
   - error response
   - status codes
3. Implement equivalent Gin handler.
4. Use constructor injection.
5. Preserve JSON field names exactly.
6. Add httptest parity tests.
7. Run gofmt and go test.
8. Report any unverifiable behavior.

# Constraints

- Do not rewrite unrelated endpoints.
- Do not change database schema.
- Do not silently simplify error handling.
- Do not introduce a Go DI framework unless requested.
```

### api-parity-test/SKILL.md

```md
---
description: Create parity tests between Java endpoint behavior and Go Gin implementation.
---

# Goal

Prove that the Go endpoint matches the Java endpoint behavior.

# Checklist

- HTTP method and path
- required headers
- auth failures
- validation failures
- success response JSON shape
- error response JSON shape
- status codes
- empty/null/default field behavior
- pagination/sorting/filtering behavior

# Test Style

- Use Go httptest.
- Prefer table-driven tests.
- Keep mocks small and explicit.
```

### migration-review/SKILL.md

```md
---
description: Review migrated Go code against Java behavior.
---

# Review Focus

- missing status code parity
- JSON field mismatch
- error format mismatch
- auth/validation mismatch
- DB transaction mismatch
- context cancellation missing
- concurrency issues
- missing test cases
- over-broad refactor

# Output

Findings first, ordered by severity.
Include file and line references.
```

## 9. 阶段 5：Superpowers 单切片工程闭环

### 目标

Superpowers 不作为主框架，而是在每个迁移切片里保证工程纪律。

### 推荐用法

```text
1. Superpowers planning
   明确当前切片范围。

2. Superpowers test-driven-development
   先写 Go parity test。

3. 自定义 skill: vertx-guice-to-gin-slice
   实现 Gin handler/service/repository。

4. Superpowers debugging
   修复测试失败。

5. Superpowers requesting-code-review
   检查迁移差异和工程质量。
```

### 单切片完成标准

```text
- Go 代码编译通过
- gofmt 通过
- go test ./... 通过
- status code 对齐
- JSON 字段对齐
- 错误格式对齐
- 鉴权行为对齐
- 参数校验行为对齐
- 日志和 context 行为合理
```

## 10. 阶段 6：按接口切片迁移

### 推荐顺序

```text
1. GET /health
2. GET /version
3. GET 查询类接口
4. POST 简单创建接口
5. PUT/PATCH 更新接口
6. DELETE 接口
7. 事务复杂接口
8. 外部 API 调用接口
9. 异步任务
```

### 单接口迁移提示词

```text
/vertx-guice-to-gin-slice

Migrate GET /api/users/:id.

Java source:
- com.example.route.UserRoute
- com.example.handler.UserHandler
- com.example.service.UserService
- com.example.repository.UserRepository

Target:
- go-service/internal/http/handler/user_handler.go
- go-service/internal/service/user_service.go
- go-service/internal/repository/user_repository.go
- go-service/internal/model/user.go

Preserve behavior exactly.
Add httptest parity tests.
Run gofmt and go test ./...
```

### 单接口迁移记录模板

```md
# Migration Record: GET /api/users/:id

## Java Source

- UserRoute.java
- UserHandler.java
- UserService.java
- UserRepository.java

## Go Target

- user_handler.go
- user_service.go
- user_repository.go
- user_handler_test.go

## Behavior Parity

- [ ] method/path
- [ ] request params
- [ ] auth
- [ ] validation
- [ ] success response
- [ ] error response
- [ ] status code
- [ ] DB access
- [ ] logs

## Verification

- [ ] gofmt
- [ ] go test ./...
- [ ] manual curl
- [ ] parity runner

## Notes

Unverified behavior:
```

## 11. 阶段 7：并行运行与流量对比

### 目标

Java 服务和 Go 服务并行运行，通过真实请求或录制请求做行为对比。

### 对比项

```text
- status code
- response headers
- response body
- error body
- DB side effects
- latency
- logs
- external API calls
```

### 建议 parity runner 结构

```text
tests/parity/
  cases/
    user_get_by_id.json
    order_create.json
  runner.go
  README.md
```

### case 示例

```json
{
  "name": "get user by id success",
  "method": "GET",
  "path": "/api/users/123",
  "headers": {
    "Authorization": "Bearer test-token"
  },
  "expect": {
    "status": 200,
    "jsonFields": ["id", "name", "email", "createdAt"]
  }
}
```

## 12. 阶段 8：灰度切换与下线 Java 模块

### 切换前 checklist

```text
- 所有核心 endpoint 已迁移
- parity tests 通过
- 回归测试通过
- 性能压测通过
- 监控指标完整
- 日志字段可搜索
- panic recover middleware 存在
- timeout / context cancellation 已处理
- DB transaction 行为确认一致
- rollback 方案准备完成
```

### 切换策略

```text
1. Go 服务先接内部测试流量
2. 小比例灰度
3. 观察关键指标和错误率
4. 扩大流量
5. Java 服务作为 fallback 保留一段时间
6. 完全切换
7. 下线 Java 旧模块
```

### 关键监控指标

```text
- request count
- status code distribution
- p95 / p99 latency
- error rate
- DB query latency
- external API latency
- panic count
- timeout count
- auth failure count
```

## 13. 迁移风险与应对

| 风险 | 说明 | 应对 |
| --- | --- | --- |
| 行为不一致 | Java 代码有隐含逻辑，Go 实现遗漏 | Reversa + parity tests |
| 错误格式变化 | 客户端依赖旧错误 JSON | 统一 error-model spec |
| Guice binding 遗漏 | 运行时依赖没有迁移 | guice-bindings.md |
| 事务边界变化 | Go repository 拆分后事务不一致 | 单独记录 transaction map |
| 异步行为变化 | Vert.x EventBus 语义被简化 | 单独设计 async-job spec |
| 上下文腐烂 | 长任务里 AI 忘记早期结论 | GSD atomic tasks |
| 过度重构 | 顺手改架构导致范围失控 | 每个 task 限定修改范围 |

## 14. 最终验收标准

项目级验收：

```text
- 所有迁移范围内接口完成 Go Gin 实现
- 所有接口都有 parity test
- go test ./... 通过
- Java / Go 响应对比通过
- 关键业务流程人工验收通过
- 灰度期间错误率和延迟在可接受范围内
- rollback 方案验证完成
- Java 旧模块下线计划明确
```

## 15. 一句话原则

```text
以行为规格为中心迁移，不以代码翻译为中心迁移。
```

## 16. 参考链接

- Reversa: https://github.com/sandeco/reversa
- GitHub Spec Kit: https://github.com/github/spec-kit
- Superpowers: https://github.com/obra/superpowers
