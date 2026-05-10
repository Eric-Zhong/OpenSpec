# Event-Driven 从入门到精通

## 前言：为什么需要事件驱动

想象一个电商系统：用户下单后，系统需要扣减库存、发送确认邮件、更新物流、通知推荐系统、记录数据分析……如果这些服务全部通过同步 API 调用，任何一个环节失败都会导致整个链路崩溃。

**事件驱动架构（Event-Driven Architecture, EDA）** 提供了一种不同的思考方式：系统组件不直接互相调用，而是通过发布和消费"事件"来协作。下单服务发布 `OrderPlaced` 事件，库存、邮件、物流等各自独立订阅这个事件并处理。彼此不知道对方的存在，却高效地协同工作。

这不是一本文档，而是一本从入门到精通的完整指南。无论你是第一次接触事件驱动，还是需要深入理解分布式事务和事件版本演进，都能在这里找到答案。

---

## 第一章：认识事件驱动

### 1.1 什么是事件（Event）

**事件是系统中已经发生的、不可改变的事实。**

```
OrderPlaced       ← 订单已创建（过去时态，事实）
PaymentProcessed  ← 支付已处理（过去时态，事实）
UserRegistered    ← 用户已注册（过去时态，事实）
```

事件的核心属性：
- **不可变性（Immutability）**：事件一旦产生，永远不再改变
- **时间性（Temporality）**：事件代表过去发生的动作
- **完整性（Completeness）**：事件携带所有必要的上下文信息
- **自治性（Autonomy）**：事件的消费者可以独立处理，不需要回调生产者

### 1.2 事件与命令的区别

| 特性 | 命令（Command） | 事件（Event） |
|------|----------------|---------------|
| 含义 | 意图、期望 | 事实、结果 |
| 时态 | 现在时：`PlaceOrder` | 过去时：`OrderPlaced` |
| 可拒绝 | 可以（库存不足时拒绝） | 不可以（已经发生了） |
| 数量 | 一个命令最多触发一个对应事件 | 一个事件可触发多个后续动作 |
| 语义 | "我希望..." | "已经发生了..." |

### 1.3 事件驱动架构的组成

```
┌──────────┐     ┌────────────┐     ┌──────────┐
│ Producer │────>│ Event Bus  │────>│Consumer A│
│ (生产者) │     │  (消息总线) │     │ (消费者) │
└──────────┘     └────────────┘     └──────────┘
                       │                  │
                       │     ┌───────────┘
                       │     ▼
                       │     ┌──────────┐
                       └───>│Consumer B│
                            │ (消费者) │
                            └──────────┘
```

- **事件生产者（Producer/Emitter）**：检测业务变化并发布事件的组件
- **事件总线/消息代理（Event Bus/Broker）**：负责任务路由、持久化、分发（Kafka、RabbitMQ、AWS SNS/SQS 等）
- **事件消费者（Consumer/Subscriber）**：订阅并处理事件的组件
- **事件处理器（Event Handler）**：消费者内部处理单个事件的业务逻辑

### 1.4 何时使用事件驱动

**适用场景：**
- 多个服务需要响应同一个业务变化
- 需要解耦服务间的依赖关系
- 实时或准实时处理需求
- 需要审计追踪和数据回放
- 系统规模大，团队需要独立演进各自的服务

**不适用场景：**
- 简单的 CRUD 操作
- 强一致性要求的简单事务
- 需要即时响应的同步操作
- 团队缺乏分布式系统经验

---

## 第二章：核心设计模式

### 2.1 发布-订阅（Pub/Sub）

最基础的事件驱动模式。发布者将事件发送到主题（Topic），订阅者接收并处理。

```yaml
# 一个事件，多个消费者
OrderPlaced → [库存服务, 邮件服务, 推荐引擎, 数据分析]
              ↓         ↓          ↓          ↓
           扣减库存   发送确认   更新推荐   记录埋点
```

**关键设计原则：**
- 生产者不知道消费者的存在
- 新增消费者不影响生产者
- 消费者失败不影响生产者

### 2.2 CQRS：命令查询职责分离

```
写路径：Command → CommandHandler → Aggregate → DomainEvent → EventStore
读路径：DomainEvent → Projection → ReadModel → QueryHandler → Response
```

**为什么分离？**
- 写操作需要业务规则校验，读操作需要高性能查询
- 读写可以使用不同的存储引擎
- 可以独立扩展读写能力

**一致性模型：**
- 读路径最终一致（Eventual Consistency）
- 数据从事件投影到读取模型，通常有秒级延迟

### 2.3 事件溯源（Event Sourcing）

将状态变化作为事件序列存储，而非仅存储当前状态。

```
用户状态的变化：
  UserCreated { name: "张三", email: "zhangsan@example.com" }
  → UserEmailUpdated { email: "new@email.com" }
  → UserProfileUpdated { avatar: "/avatar.jpg" }

当前状态 = 重放所有事件后的结果
```

**优势：**
- 完整审计日志
- 可以"时光旅行"——在任何时间点的状态可还原
- 事件可被重新处理

**挑战：**
- 事件版本管理
- 大量事件时的性能问题（需使用快照机制）

### 2.4 Saga：分布式事务

当 ACID 事务无法跨越多个服务时，Saga 通过补偿操作保证最终一致性。

**编排式（Orchestration）：**
```
OrderService
  ├── 1. 创建订单（Pending）
  ├── 2. → ChargePayment
  ├── 3. → ReserveInventory
  ├── 4. → ShipOrder
  └── 失败时 → RefundPayment → CancelOrder
```

**编舞式（Choreography）：**
```
OrderCreated
  ↓
PaymentService: 收到 OrderCreated → 扣款 → PaymentCompleted
  ↓
InventoryService: 收到 PaymentCompleted → 扣库存 → InventoryReserved
  ↓
ShippingService: 收到 InventoryReserved → 发货 → OrderShipped
```

**补偿事务规则：**
- 每一步必须设计对应的补偿操作
- 补偿操作本身必须幂等
- 补偿通常按逆序执行

---

## 第三章：事件风暴（Event Storming）

### 3.1 什么是事件风暴

事件风暴由 Alberto Brandolini 创建，是一种协作式工作坊方法，用于快速探索复杂的业务领域。它通过物理或虚拟的便利贴，将业务专家、开发者和干系人聚集在一起，共同发现领域事件、理解业务流程、识别系统边界。

### 3.2 颜色编码体系

| 颜色 | 元素类型 | 示例 | 模板 |
|------|---------|------|------|
| 🟠 橙色 | 领域事件 | "Order Placed" | 过去时态的业务事实 |
| 🔵 蓝色 | 命令 | "Place Order" | 动作/意图 |
| 🟡 黄色 | 参与者/用户 | "Customer" | 触发命令的人或系统 |
| 🟣 紫色 | 策略/自动化 | "If stock < 10, reorder" | 触发规则 |
| 🟢 绿色 | 读取模型 | "Order Dashboard" | 数据视图 |
| 🔴 红色 | 热点/问题 | "Payment timeout" | 需要解决的痛点 |
| 🟤 粉色 | 外部系统 | "Stripe API" | 外部依赖 |

### 3.3 工作坊流程

**第一阶段：准备（Workshop Before）**
1. 确定范围和参与者（8-12 人最佳）
2. 邀请领域专家、产品负责人、开发团队成员
3. 准备墙面空间和便利贴

**第二阶段：发现（Discovery）**
1. **生成领域事件**（25-60 分钟）：参与者独立写下所有能想到的事件
2. **按时间排序**（30-60 分钟）：将事件按业务时序排列
3. **添加参与者和外部系统**（30 分钟）：标记谁触发什么
4. **识别命令**（30-45 分钟）：每个事件前添加"为什么发生"
5. **添加策略和读取模型**（30-45 分钟）：补充自动化逻辑

**第三阶段：提炼（Synthesis）**
1. **识别聚合和边界**（45-60 分钟）：分组命令和事件
2. **故事讲述**（30 分钟）：按时间线讲述整个流程
3. **标记热点**：识别问题和风险
4. **文档化**：拍照、整理、分派行动项

### 3.4 事件风暴的产出

事件风暴不只是讨论——它产生具体的、可操作的产出：

1. **领域词汇表**：统一的业务术语，消除沟通歧义
2. **限界上下文（Bounded Contexts）**：自然的系统/服务边界
3. **业务流程全景图**：完整的业务流转可视化
4. **问题和风险清单**：标记的热点和待决策项

---

## 第四章：事件建模（Event Modeling）

### 4.1 从事件风暴到事件建模

事件风暴是发现阶段，事件建模是设计阶段。它将便利贴上的发现转化为结构化的行为流，为后续的规范、设计和 API 文档奠定基础。

### 4.2 泳道图结构

```
┌────────────────────────────────────────────┐
│  触发器（Trigger）                          │
│  用户操作、系统信号、外部事件                │
├────────────────────────────────────────────┤
│  命令（Command）                            │
│  基于意图触发的动作                          │
├────────────────────────────────────────────┤
│  事件（Event）                              │
│  命令执行后产生的事实                        │
├────────────────────────────────────────────┤
│  读取模型（Read Model）                     │
│  从事件投影产生的查询数据                    │
└────────────────────────────────────────────┘
```

### 4.3 四种工作流模式

**命令模式（Command Pattern）：**
```
用户点击"下单" → PlaceOrder 命令 → OrderPlaced 事件 → 库存扣减
```

**查看模式（View Pattern）：**
```
OrderPlaced 事件 → 订单列表读取模型 → 用户在 Dashboard 看到订单
```

**自动化模式（Automation Pattern）：**
```
PaymentExpired 事件 → 读取逾期模型 → 自动触发 SendReminder 命令 → ReminderSent 事件
```

**翻译模式（Translation Pattern）：**
```
外部支付网关回调 → 内部 PaymentReceived 事件 → 更新订单状态
```

### 4.4 实现切片（Implementation Slices）

将复杂流程拆分为可独立实现的最小单元：

1. **状态变更切片**：接口 → 命令 → 事件
2. **状态查看切片**：事件 → 读取模型 → 界面
3. **外部状态导入切片**：导入外部数据
4. **内部状态导出切片**：向外部系统导出数据

每个切片都可以独立开发、测试和部署。

---

## 第五章：AsyncAPI 规范

### 5.1 什么是 AsyncAPI

AsyncAPI 是一个开放规范，用于描述异步 API。就像 OpenAPI 为 REST API 提供标准化文档，AsyncAPI 为事件驱动的 API 提供相同的标准化能力。

### 5.2 核心概念

**通道（Channels）：** 消息交换的地址空间（Kafka Topic、MQTT Topic、AMQP Queue）

```yaml
channels:
  orderPlaced:
    address: orders/placed
    messages:
      OrderPlacedMessage:
        $ref: '#/components/messages/OrderPlaced'
```

**操作（Operations）：** 在通道上执行的动作（发送/接收）

```yaml
operations:
  publishOrderPlaced:
    action: send
    channel:
      $ref: '#/channels/orderPlaced'
```

**消息（Messages）：** 交换的数据结构

```yaml
messages:
  OrderPlaced:
    name: OrderPlaced
    payload:
      $ref: '#/components/schemas/OrderPlacedPayload'
```

**模式（Schemas）：** 数据模型定义

```yaml
schemas:
  OrderPlacedPayload:
    type: object
    required: [orderId, userId, timestamp]
    properties:
      orderId:
        type: string
        format: uuid
      userId:
        type: string
      items:
        type: array
        items:
          $ref: '#/components/schemas/OrderItem'
```

### 5.3 绑定（Bindings）

协议特定的扩展配置：

**Kafka 绑定：**
```yaml
bindings:
  kafka:
    replicas: 3
    partitions: 12
```

**MQTT 绑定：**
```yaml
bindings:
  mqtt:
    qos: 2
    retain: false
```

### 5.4 数据流关系

```
Schema → 被 Message 引用 → 被 Channel 引用 → 被 Operation 引用
```

这是一个自底向上的引用链，确保每个层次都可追溯到数据定义。

---

## 第六章：消息代理选型

### 6.1 消息代理对比

| 代理 | 最佳场景 | 消息模型 | 有序性 | 吞吐量 |
|------|---------|---------|--------|--------|
| **Kafka** | 高吞吐事件流 | 日志式 Pub/Sub | 按分区 | 百万级/秒 |
| **RabbitMQ** | 任务队列、复杂路由 | AMQP | 按队列 | 数万级/秒 |
| **AWS SQS/SNS** | 无服务器、低运维 | 托管队列 | FIFO 可选 | 近乎无限 |
| **Cloud Pub/Sub** | 云原生 Pub/Sub | Topic + Subscription | 按 Key | 百万级/秒 |
| **NATS** | 低延迟微服务 | 最多一次 | 按主题 | 亚毫秒延迟 |

### 6.2 投递语义

**最多一次（At-most-once）：**
- 消息可能丢失但不会重复
- 适用于可丢弃的通知

**至少一次（At-least-once）：**
- 消息不会丢失但可能重复
- **最常用**，需要消费者实现幂等性

**精确一次（Exactly-once）：**
- 消息仅投递和处理一次
- 实现复杂，仍需应用层幂等保护

### 6.3 死信队列（DLQ）

当消息处理失败超过阈值时，消息被路由到死信队列。

```
正常队列 → [处理失败] → [重试 N 次] → [超过阈值] → 死信队列
                                                       ↓
                                               告警 + 人工处理
```

**设计原则：**
- 每个生产队列必须配置 DLQ
- DLQ 增长需触发告警
- 自动化重处理机制

---

## 第七章：高级主题

### 7.1 事件版本管理

当事件结构需要变更时，如何保证向后兼容？

**策略对比：**

| 策略 | 描述 | 适用场景 |
|------|------|---------|
| 事件名称版本化 | `user.created.v1`, `user.created.v2` | 消费者需要明确选择版本 |
| 原地演进 | 只增加字段，不删除/重命名 | 小幅改动，消费者忽略未知字段 |
| 独立流 | 不同版本使用独立主题 | 大的不兼容变更 |
| Schema 注册表 | 集中管理（Confluent Schema Registry） | 企业级管理 |
| 向上转换 | 读取时将旧事件转换为最新格式 | 消费者只需处理最新版本 |

**兼容性黄金法则：**
1. 添加可选字段且有默认值 — 永远安全
2. 永远不要删除必填字段 — 先弃用
3. 永远不要更改已有字段的数据类型
4. 结构变更和语义变更分开处理

### 7.2 幂等性设计

幂等性保证同一事件被多次处理不会产生副作用。

```
处理 OrderPlaced 事件：
  1. 检查 orderId 是否已处理
  2. 如果已处理，返回成功
  3. 如果未处理，执行业务逻辑
  4. 标记 orderId 为已处理
```

**实现模式：**
- **处理记录表**：记录已处理的事件 ID
- **数据库唯一约束**：利用数据库天然幂等性
- **乐观锁**：版本号机制防止并发冲突
- **去重窗口**：基于时间窗口的重复检测

### 7.3 分布式追踪

事件驱动系统中的可观测性挑战：

```
Trace ID: 追踪整个请求链路
Correlation ID: 关联相关事件
Event ID: 唯一标识每个事件
```

**最佳实践：**
- 每个事件携带 Trace ID 和 Correlation ID
- 结构化日志记录所有事件处理
- 使用 OpenTelemetry 等标准协议
- 可视化追踪链路

### 7.4 反模式与陷阱

**蜂群（Swarm of Gnats）：** 发送过多细小、低价值的事件，导致系统噪声和耦合。

**把事件当命令：** 用事件做异步 RPC 调用（如 `ValidateUser`, `SendEmail`），违反事件的"事实"语义。

**缺少幂等性：** 未保护重复事件处理，导致数据不一致。

**忽略 Schema 演进：** 随意修改事件结构，破坏消费者。

**没有 DLQ：** 失败事件消失，无恢复机制。

**缺乏可观测性：** 异步事件链中无法追踪问题根源。

---

## 第八章：实际案例研究

### 8.1 麦当劳：全球事件驱动平台

- **挑战**：全球数百万订单需要统一平台
- **方案**：AWS MSK（托管 Kafka），基于领域的分片策略，自定义 SDK 含 Schema 验证
- **成果**：统一全球订单系统，处理百万级日订单

### 8.2 Zalando：时尚电商的事件驱动转型

- **挑战**：产品数据架构需要支持 350+ 工程团队
- **方案**：CQRS 架构 + Caffeine 缓存 + DynamoDB
- **成果**：P99 延迟 < 10ms，大促期间事件延迟从 30 分钟降至秒级

### 8.3 LEGO：Black Friday 后的无服务器事件驱动架构

- **挑战**：Black Friday 系统宕机 2 小时
- **方案**：迁移到 AWS 无服务器事件驱动架构
- **成果**：可处理 200 倍交易峰值，9.5 倍流量峰值

---

## 第九章：总结与学习路径

### 9.1 知识体系回顾

```
入门层          核心概念、事件 vs 命令、Pub/Sub
│
中进阶          CQRS、事件溯源、Saga、消息代理选型
│
高级            事件风暴、事件建模、AsyncAPI
│
专家级          Schema 演进、幂等性、分布式追踪、反模式
```

### 9.2 推荐学习资源

- **书籍**：《Domain-Driven Design》（Eric Evans）、《Implementing DDD》（Vaughn Vernon）
- **规范**：AsyncAPI 官方文档 https://www.asyncapi.com/
- **方法**：Event Storming 官方指南 https://www.eventstorming.com/
- **实践**：Confluent 事件建模课程 https://developer.confluent.io/courses/event-modeling/

### 9.3 下一步

理解理论只是第一步。下一章你将学习如何使用 OpenSpec 的 Event-Driven Schema 将这些理论转化为可执行的工程实践。
