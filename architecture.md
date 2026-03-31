# Atomic Architecture

一种将状态管理与业务执行严格分离的微函数式架构范式。通过 Infrastructure 与 Context 集中承载进程级与请求级状态，以原子化的纯函数 Logic 处理业务规则，以渐进提取的 Service 提供无状态计算，强制显式依赖传递，禁止隐式全局访问。

## Infrastructure

系统级基础设施，生命周期与进程相同，包含数据库连接池、缓存客户端、配置中心等重资源。不持有任何请求级状态。

## Context

请求级或会话级上下文，生命周期明确且短暂，如一次 HTTP 请求、一个事务或一个定时任务执行周期。它包含当前用户、请求ID、事务句柄、临时状态等，持有Infrastructure实例，其他模块通常也是通过Context访问到Infrastructure

## Component

主动逻辑实体，具有生命周期回调，如定时任务调度器、消息监听器、第三方 SDK 包装器。它持有 Infrastructure 的引用，负责在触发时创建 Context，并编排 Logic 的调用。

## Logic

无状态的原子业务逻辑，以显式参数传递依赖和上下文，不持有生命周期状态。Logic 应尽量保持纯计算风格，但允许在受控情况下通过 Context 间接访问 Infrastructure 能力完成业务处理。

## Service

无状态的通用工具，纯函数，禁止通过Infrastructure产生状态，禁止接收 Context 参数，禁止访问 Infrastructure，仅通过纯数据类型进行计算、转换、验证等无副作用操作，且必须是静态方法或模块级函数，禁止实例化。Service 禁止预先设计，只允许渐进式增加，当同一能力在三个或以上 Logic 中重复出现时，才允许提取为 Service。