# Java 到 Go 重构主线计划

## 1. 目标

本计划用于将现有 Java 项目按阶段重构为 Go 项目。

重构不采用一次性整体改写，而是按照依赖顺序分阶段迁移。每个阶段独立分析、实施和验收，验收通过后才能进入下一阶段。

总体目标：

- 保持现有系统对外行为稳定
- 保持关键业务规则和数据语义一致
- 控制每个阶段的修改范围
- 让阶段成果可以独立验证
- 逐步替换 Java 实现，避免大爆炸式重写
- 对无法确认或暂不迁移的内容持续记录 GAP

## 2. 总体主线

```text
阶段 1：持久化层迁移
        ↓
阶段 2：API 契约层迁移
        ↓
阶段 3：横切基础设施迁移
        ↓
阶段 4：并发与生命周期迁移
        ↓
阶段 5：Service 业务层迁移
        ↓
阶段 6：系统级功能校验
```

六个阶段分别执行，不在同一次任务中完成整个项目重构。

## 3. 全局原则

### 3.1 Java 源码是最终事实

现有分析报告和迁移文档用于提供上下文，但当文档与源码不一致时，以 Java 源码和实际运行行为为准。

### 3.2 行为迁移优先于代码翻译

目标不是逐行翻译 Java，而是迁移：

- 输入和输出行为
- 数据读写行为
- 事务和一致性规则
- 错误和异常行为
- 缓存和消息副作用
- 并发和生命周期行为
- 对外接口兼容性

### 3.3 阶段边界必须稳定

每个阶段开始前明确：

- 当前目标
- 输入范围
- 输出范围
- 明确非目标
- 依赖的前置成果
- 阶段验收标准

不得以“顺便完成”为理由提前实现后续阶段内容。

### 3.4 每个阶段独立验证

阶段 6 不是第一次测试。阶段 1 到阶段 5 都必须完成本阶段验证，阶段 6 负责系统级和跨模块最终校验。

### 3.5 持续记录决策和缺口

整个迁移过程持续维护：

```text
migration-roadmap.md
migration-decisions.md
migration-gaps.md
migration-traceability.md
```

每个阶段结束时生成独立的 `handoff.md`，供下一阶段使用。

### 3.6 Java 与 Go 文件映射规则

本节及后续 3.7、3.8、3.9 的可追溯规则是整个迁移项目的全局强制规则，适用于全部六个阶段：

```text
阶段 1：持久化层迁移
阶段 2：API 契约层迁移
阶段 3：横切基础设施迁移
阶段 4：并发与生命周期迁移
阶段 5：Service 业务层迁移
阶段 6：系统级功能校验
```

任何阶段、任何执行批次、任何 Agent 都不得跳过这些规则。阶段自身的计划和实现约定只能补充这些规则，不能覆盖或取消这些规则。

为保证 Go 重构结果可以回溯到 Java 源码，迁移文件默认采用 1:1 映射：

```text
一个 Java 源文件
-> 一个主要 Go 目标文件
```

1:1 是默认规则，不是不可突破的结构限制。出现以下情况时允许 1:N 或 N:1：

- 一个 Java 文件同时承担多个职责，需要在 Go 中拆分
- 多个 Java 接口或实现共同组成一个 Go 能力
- Java DTO、Entity、Enum 在 Go 中需要按层次重新归属
- Go 语言组织方式不适合机械复制 Java 类结构

任何非 1:1 映射都必须在 `migration-traceability.md` 中记录：

- 映射类型
- 调整原因
- 涉及的 Java 文件
- 涉及的 Go 文件
- 人工核验状态

### 3.7 Go 文件必须标注 Java 来源

每个由 Java 迁移生成的 Go 文件，文件头必须写明对应的 Java 源文件 `contentPath`。

路径使用 Java 仓库内的相对路径，不使用开发者机器的绝对路径。

示例：

```go
// Legacy Java sources:
// - bic-service/src/main/java/com/example/FooService.java
// - bic-service/src/main/java/com/example/FooMapper.java
//
// Traceability: migration-traceability.md
```

要求：

- 1:1 映射时记录一个 Java `contentPath`
- 1:N 映射时，每个 Go 文件记录共同的 Java 来源
- N:1 映射时，Go 文件列出全部 Java 来源
- 纯 Go 新增基础设施文件标记为 `Go-native`，并说明其迁移目的
- Java 来源变化时同步更新文件头和映射表

文件头只负责快速定位来源，完整迁移状态、基线版本和差异记录放在 `migration-traceability.md`。

### 3.8 Go 关键代码符号必须标注 Java 来源

除文件头来源外，Go 文件中的关键代码符号也必须使用简短注释标注其对应的 Java 源代码。

适用对象：

- 导出的函数和方法
- 核心未导出函数和方法
- 结构体
- 接口
- 常量和关键变量
- 枚举式常量组
- 关键构造函数
- 承载业务语义的类型别名

注释至少包含：

- Java 类、接口、方法、字段或常量名称
- Java 仓库相对 `contentPath`
- 必要时说明是直接迁移、拆分、合并还是 Go 原生重构

示例：

```go
// KVRecord 迁移自 Java BicServiceKv。
// Java source: bic-service/src/main/java/com/example/entity/BicServiceKv.java
type KVRecord struct {
    // ...
}

// GetByKey 迁移自 Java BicServiceKvMapper.getKv。
// Java source: bic-service/src/main/java/com/example/mapper/BicServiceKvMapper.java
func (r *KVRepository) GetByKey(ctx context.Context, key string) ([]KVRecord, error) {
    // ...
}

// MaxWaitTime 迁移自 Java IConsulConstants.MAX_WAIT_TIME。
// Java source: bic-service/src/main/java/com/example/common/IConsulConstants.java
const MaxWaitTime = 180 * time.Second
```

注释规则：

- Go 导出符号的注释仍应以 Go 符号名称开头，保持 Go 文档规范
- 一个 Go 符号对应多个 Java 符号时，列出全部主要来源
- 一个 Java 方法拆成多个 Go 函数时，每个 Go 函数都标注同一 Java 来源和拆分关系
- 多个 Java 方法合并为一个 Go 函数时，注释列出全部 Java 方法
- 纯 Go 新增符号标记为 `Go-native`，并简述其服务于哪个迁移能力
- 注释保持简短，不复制 Java 业务逻辑全文
- Java 来源发生变化时，必须同步修改符号注释和映射表

### 3.9 每阶段输出增量映射表

每个阶段完成后，必须新增或更新一份 Java/Go 文件映射表，作为阶段验收和下一阶段增量调整的基线。

即使当前阶段主要产出的是 Router、Handler、Middleware、Cache、Worker、Scheduler、Service 或测试文件，也必须执行来源标注和映射表更新，不得只在持久化层执行。

统一维护在：

```text
migration-traceability.md
```

表格模板：

| 阶段 | 批次 | Java contentPath | Java 已迁移符号 | Go contentPath | 映射类型 | 状态 | 验证结果 | GAP/备注 |
|---|---|---|---|---|---|---|---|---|
| 阶段 1 | `P1-orm-01` | `bic-service/src/.../FooMapper.java` | `selectFoo` | `internal/.../foo_repository.go` | 1:1 | 已迁移 | 测试通过 | 无 |

映射类型：

```text
1:1  一个 Java 文件对应一个主要 Go 文件
1:N  一个 Java 文件拆分为多个 Go 文件
N:1  多个 Java 文件合并为一个 Go 文件
NEW  没有直接 Java 文件来源的 Go 原生基础设施
```

状态统一使用：

```text
待分析
迁移中
已迁移
已验证
GAP
排除
```

每次阶段执行只更新本次涉及的增量记录，同时保留历史映射，便于判断：

- Java 文件是否遗漏
- Go 文件是否缺少来源
- 迁移状态是否发生变化
- 上一阶段结论是否需要调整
- 新增或修改的 Java 代码是否尚未同步

为保证增量核验可靠，映射表还应在阶段标题或元数据中记录本次分析使用的 Java 基线 commit SHA。

### 3.10 批次隔离

同一阶段可以拆成多个批次执行，降低人工检查压力。例如阶段 1 持久化层可以按 Java 源码路径、Mapper 分组或业务表分为 `P1-orm-01`、`P1-orm-02`。

每个批次必须具备唯一批次 ID。批次 ID 格式为 `<阶段ID>-<业务/能力>-<步骤序号>`，例如 `P1-orm-01`，语义是“阶段-业务-步骤”。

每个批次必须明确：

- 当前批次目标
- 当前批次 Java source scope
- 当前批次 Go target scope
- 当前批次非目标
- 当前批次 Superpowers spec / plan 输出路径

批次执行原则：

- 当前批次只处理本批次 source scope，不顺手迁移范围外代码。
- 当前批次可以读取前序批次文档和代码，但不得覆盖前序批次 Superpowers specs/plans。
- 当前批次如果需要修改前序批次已验收代码，必须记录影响批次、修改原因和重新验证结果。
- 每个批次完成后增量更新 `migration-roadmap.md`、`migration-traceability.md`、`migration-decisions.md` 和 `migration-gaps.md`。
- `phase-N-handoff.md` 只在整个阶段完成后生成；不要每个批次都生成阶段 handoff。

## 4. 阶段 1：持久化层迁移

### 4.1 目标

将 Java 持久化层迁移为稳定、可独立验证的 Go 数据访问层，为后续业务实现提供基础。

### 4.2 范围

- Java Entity
- MyBatis Mapper
- SQL 和数据库方言
- Go 持久化 Model
- Go ORM / Repository
- 事务基础能力
- 数据库错误转换

### 4.3 非目标

- HTTP Router 和 Handler
- 真实 Service 业务实现
- Cache
- Scheduler
- Thread / ThreadPool
- Notify
- 完整业务链路

### 4.4 阶段产物

- Java Mapper 与 Go Repository 映射关系
- Go 持久化 Model
- Go Repository 接口和实现
- SQL 行为和数据库方言差异记录
- 持久化层测试
- 未实现或无法确认的 GAP
- 阶段交接文档

### 4.5 完成条件

- 目标持久化能力已迁移或明确标记 GAP
- Go 持久化层可以独立编译和测试
- 数据字段、查询结果、写入副作用和事务行为已验证
- 没有引入后续阶段的业务实现

## 5. 阶段 2：API 契约层迁移

### 5.1 目标

将 Java Controller 层对外契约迁移为 Go Router 和 Handler，在不实现真实业务逻辑的情况下固定 Go 接口边界。

### 5.2 范围

- Controller 路由
- HTTP Method 和 Path
- Request Parameter
- Response Parameter
- 传输层 DTO
- 传输层 Constant 和 Enum
- Router
- Handler
- Service Interface
- Service Fake / Stub
- API 契约测试

### 5.3 非目标

- 真实 Service 业务实现
- 完整数据库业务链路
- Scheduler 和后台任务
- 完整 Cache 行为
- 完整消息通知行为

### 5.4 边界原则

- 只在本阶段迁移接口契约相关 DTO、Constant 和 Enum
- 业务领域模型和业务规则在阶段 5 迁移
- Service Interface 用于固定 Handler 依赖边界
- Stub 只用于验证接口契约，不作为最终业务实现

### 5.5 阶段产物

- Go Router 和 Handler
- Request / Response 模型
- 接口级 DTO、Constant、Enum
- Service Interface
- Service Fake / Stub
- API 契约测试
- Java/Go 接口映射表
- 阶段交接文档

### 5.6 完成条件

- 目标路由可以完成注册
- Handler 可以通过 Stub 独立运行
- 请求解析、响应格式、状态码和错误格式可以验证
- 没有提前实现真实业务逻辑

## 6. 阶段 3：横切基础设施迁移

### 6.1 目标

迁移被多个业务模块共同依赖的横切能力，为后续 Service 和运行时模型提供稳定接口。

### 6.2 范围

- Interceptor 到 Middleware 的迁移
- 统一错误处理
- 鉴权和 Token 处理
- Cache 抽象及实现
- Cache 失效机制
- Notify 消息通知抽象
- 日志和公共上下文
- 公共配置能力

### 6.3 非目标

- 完整 Scheduler 实现
- Thread / ThreadPool 模型
- 容器完整启动关闭流程
- 具体业务 Service 编排
- 完整端到端业务链路

### 6.4 阶段产物

- Middleware 和横切能力接口
- Cache 接口及必要实现
- Notify 接口及必要实现
- 公共错误和鉴权模型
- 横切基础设施测试
- 阶段交接文档

### 6.5 完成条件

- 横切能力可以独立使用和验证
- 对后续 Service 暴露的接口已经稳定
- 不依赖完整 Service 业务实现

## 7. 阶段 4：并发与生命周期迁移

### 7.1 目标

将 Java 中的调度、线程、线程池、策略模型以及容器生命周期迁移为可控制、可关闭、可验证的 Go 运行时模型。

### 7.2 范围

- Scheduler
- Thread
- ThreadPool
- Worker 和任务队列
- Strategy
- 启动 Loader
- 启动任务顺序
- 关闭任务顺序
- Graceful Shutdown
- Context Cancellation
- 后台任务错误传播
- 运行时资源释放

### 7.3 非目标

- 完整 Service 业务实现
- 全部接口业务链路打通
- 系统级 Java/Go 最终校验

### 7.4 阶段产物

- Go 并发模型
- Worker 和任务调度模型
- Strategy 接口和运行边界
- 应用启动与关闭模型
- 后台任务生命周期管理
- 并发与生命周期测试
- 阶段交接文档

### 7.5 完成条件

- 启动和关闭顺序明确
- 后台任务可以安全停止
- 任务取消和错误传播行为明确
- 不存在已知的任务泄漏和资源泄漏
- Service 层可以稳定依赖这些能力

## 8. 阶段 5：Service 业务层迁移

### 8.1 目标

实现真实 Service 业务逻辑，连接前面阶段已经完成的 API、Repository、基础设施和运行时能力，打通完整业务链路。

### 8.2 范围

- Service Interface 的真实实现
- 业务规则
- 事务编排
- Repository 调用
- Cache 调用
- Notify 调用
- Scheduler / Worker 协作
- Handler 到数据层的完整调用链
- Service Fake / Stub 的替换

### 8.3 非目标

- 无关业务能力扩展
- 未经确认的架构重写
- 为追求理想设计而改变现有对外行为

### 8.4 阶段产物

- Go Service 实现
- 完整业务调用链
- 业务规则测试
- 事务和副作用验证
- 已替换的 Stub 清单
- Java/Go 业务差异记录
- 阶段交接文档

### 8.5 完成条件

- 目标 Stub 已被真实 Service 替换
- 主要业务链路可以完整运行
- 业务规则、事务和副作用已完成阶段性验证
- 阻塞性 GAP 已解决或有明确处理方案

## 9. 阶段 6：系统级功能校验

### 9.1 目标

从系统和外部调用方视角验证 Java 与 Go 的功能和行为是否一致，判断是否具备灰度或替换条件。

### 9.2 范围

- Java/Go 同请求对比
- HTTP Status Code
- Response Body
- Response Header
- 错误行为
- 数据库副作用
- Cache 副作用
- Notify 消息副作用
- 并发行为
- Scheduler 行为
- 启动和关闭行为
- 性能和稳定性

### 9.3 阶段产物

- Java/Go 功能对比报告
- 接口行为差异清单
- 业务链路验证结果
- 性能和稳定性结果
- 剩余 GAP 及风险评估
- 灰度、回滚或替换建议
- 最终交接文档

### 9.4 完成条件

- 所有目标接口完成验证
- 关键业务链路完成验证
- 阻塞性差异已经关闭
- 可接受差异已经记录并获得确认
- 具备明确的灰度、回滚和替换方案

## 10. 阶段门禁

每个阶段进入下一阶段前必须确认：

```text
1. 当前阶段范围内的工作已经完成。
2. 当前阶段测试和验证已经通过。
3. Review 中的阻塞问题已经解决。
4. 未解决问题已经进入 migration-gaps.md。
5. 关键决策已经进入 migration-decisions.md。
6. 已生成当前阶段 handoff.md。
7. 已更新 migration-traceability.md 中的 Java/Go 文件映射表。
8. 本阶段新增的 Go 文件已经标注 Java 来源 contentPath 或 Go-native 说明。
9. 本阶段新增或修改的 Go 关键符号已经标注对应 Java 类、方法、字段或常量来源。
10. Java 基线 commit SHA 已记录。
11. 下一阶段所依赖的接口和产物已经稳定。
```

## 11. 阶段交接关系

```text
持久化层
提供 Repository、数据模型和事务基础
        ↓
API 契约层
提供 Router、Handler、传输模型和 Service Interface
        ↓
横切基础设施
提供 Middleware、Cache、Notify、鉴权和错误能力
        ↓
并发与生命周期
提供 Scheduler、Worker、Strategy 和启动关闭模型
        ↓
Service 业务层
连接并实现完整业务链路
        ↓
系统级功能校验
验证最终 Java/Go 行为一致性
```

## 12. 最终原则

```text
一次只执行一个阶段。
一次只解决当前阶段的问题。
当前阶段验收通过后再进入下一阶段。
每个阶段都留下可验证、可交接、可追踪的成果。
所有阶段都必须维护 Java/Go 文件来源、映射状态和增量变更记录。
```
