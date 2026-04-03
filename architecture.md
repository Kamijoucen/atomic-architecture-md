# Atomic Architecture

一种将业务逻辑原子化、状态集中管理、调用方与被调用方分离的架构范式。通过 ApplicationContext 集中管理进程级状态，以原子化的纯函数 Logic 处理业务规则，以有状态的 Component 提供业务通用能力，以可动态启停的 Service 提供系统入口，强制显式依赖传递。

## 核心理念

1. **业务逻辑原子化**：Logic 层是原子化的业务函数，不持有生命周期状态
2. **状态集中管理**：所有状态由 ApplicationContext 和 Component 集中管理，避免散落在业务逻辑中
3. **调用方与被调用方分离**：依赖方向明确，禁止循环依赖

## ApplicationContext（应用上下文）

进程级单例，与进程拥有相同生命周期，在应用启动时创建。

- 作为全局访问点，持有 Component 实例
- Logic 通过 ApplicationContext 访问基础设施和其他组件
- 基础设施层出现问题时，通常意味着整体服务无法工作，因此 ApplicationContext 必须保证整个应用期间都可以访问

## Service（主动逻辑入口）

系统的主动逻辑入口，如定时任务、MQ 消息接收器、HTTP 处理器等。

- 生命周期通常与 ApplicationContext 一致，但允许可选启动和中途停止
- 可动态启停以应对负载变化（如高并发时关闭部分服务降低负载）
- 引用 ApplicationContext，但不负责创建 ApplicationContext
- 编排 Logic 的调用

## Logic（原子业务逻辑）

原子化的业务逻辑函数，处理具体业务规则。

- 不持有任何生命周期状态
- 应尽量保持纯计算风格，但允许在受控情况下通过 ApplicationContext 间接访问 Component 和 Infrastructure 能力完成业务处理
- **自演化规则**：当同一逻辑在多个地方重复出现时，自然演化为新的 Logic
- 通过 ApplicationContext 参数访问依赖

## Component（业务组件）

面向对象的内部业务通用功能，具有状态。

- **关键特征**：面向对象设计、有状态（可在进程生命周期内保持状态）
- 可以是基础设施的业务适配层（如基于 Redis 的用户上下文管理器）
- 也可以是纯业务通用能力（如基于内存的任务队列、状态机）
- 被 ApplicationContext 持有和管理

## Infrastructure（基础设施）

底层基础设施资源，如数据库连接池、Redis 客户端、ES 客户端等。

- 生命周期与进程相同
- 被 Component 依赖（可选），不直接暴露给 Logic 和 Service

## 依赖关系

```
Service ──引用──→ ApplicationContext ──持有──→ Component ──依赖（可选）──→ Infrastructure
                              ↑
Logic ──参数──→ ApplicationContext
```

**约束**：
- Logic 只能通过 ApplicationContext 访问依赖，禁止直接依赖 Component 或 Infrastructure
- Component 可依赖 Infrastructure，但不应直接暴露给 Logic
- Service 引用 ApplicationContext，但 ApplicationContext 不依赖 Service

## 跨语言适配指南

> **注意**：ApplicationContext 是本架构的应用上下文概念，与 Go 语言的 `context.Context`（请求上下文）不同。

### Java/Spring
- **ApplicationContext**：Spring ApplicationContext
- **Service**：@Service 或 @Component 注解的类，实现 Lifecycle 接口支持动态启停
- **Logic**：优先使用 @FunctionalInterface 注解的类，或单公开方法的类
- **Component**：@Component 注解的类，可通过 @Autowired 注入

### Go
- **ApplicationContext**：单例 struct，全局可访问（注意：与 Go 的 `context.Context` 不同）
- **Service**：独立的 goroutine，管理生命周期
- **Logic**：package 级函数，ApplicationContext 作为第一个参数
- **Component**：接口实现，被 ApplicationContext 持有
