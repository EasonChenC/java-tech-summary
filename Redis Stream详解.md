# Redis Stream 详解

> 本文详细介绍 Redis Stream 数据结构及其在项目中的应用

---

## 一、Stream 是什么？

**Redis Stream** 是 Redis 5.0 引入的一种**持久化消息队列**数据结构，专门用于消息传递和事件流处理。

### 核心定位

```
Stream = 日志结构的消息队列
- 类似 Kafka 的消息流
- 支持消费者组
- 消息持久化
- 支持消息确认（ACK）
```

### 与其他数据结构对比

| 特性 | List | Pub/Sub | Stream |
|------|------|---------|--------|
| **持久化** | ✅ | ❌ 内存 | ✅ |
| **消费者组** | ❌ | ❌ | ✅ |
| **消息确认** | ❌ | ❌ | ✅ |
| **历史消息** | ✅ | ❌ | ✅ |
| **多消费者** | ⚠️ 需手动实现 | ✅ | ✅ |
| **消息 ID** | 索引 | ❌ | ✅ 唯一 ID |
| **适用场景** | 简单队列 | 实时推送 | **消息队列** ✅ |

---

## 二、Stream 的基本概念

### 1. 数据结构示意

```
Stream: mystream
├─ 1717488000000-0: {field1: "value1", field2: "value2"}  ← 消息1
├─ 1717488001000-0: {field1: "value3", field2: "value4"}  ← 消息2
├─ 1717488002000-0: {field1: "value5", field2: "value6"}  ← 消息3
└─ 1717488003000-0: {field1: "value7", field2: "value8"}  ← 消息4
     ↑                ↑
   消息ID          消息内容（Hash 结构）
```

**关键特点**：
```
1. 有序性：按 ID 自动排序
2. 持久化：写入磁盘
3. 只追加：消息只能添加，不能修改
4. 唯一 ID：时间戳 + 序列号
```

### 2. 消息 ID 格式

```
消息ID格式：<millisecondsTime>-<sequenceNumber>

示例：
1717488000000-0
  ↑            ↑
  时间戳（毫秒） 序列号

同一毫秒内多条消息：
1717488000000-0
1717488000000-1
1717488000000-2
```

**特点**：
- 自动生成（使用 `*`）
- 全局唯一
- 自然排序
- 可指定（不推荐）

### 3. 消息结构

```
消息 = ID + 多个字段（键值对）

{
  "1717488000000-0": {
    "kbId": "123",
    "content": "文档内容",
    "retryCount": "0"
  }
}
```

**类似于**：
```
Stream 中的每条消息 = Hash（多个字段）
但有全局唯一的 ID 和顺序
```

---

## 三、Stream 核心操作

### 1. 添加消息（XADD）

```bash
# 基础添加
XADD mystream * field1 value1 field2 value2
→ "1717488000000-0"  # 返回消息 ID

# * 表示自动生成 ID
# field1 value1 field2 value2 是键值对

# 示例：添加知识库任务
XADD knowledgebase:vectorize:stream * kbId 123 content "文档内容" retryCount 0
→ "1717488000000-0"
```

**限制长度**：
```bash
# 限制 Stream 最多 1000 条消息
XADD mystream MAXLEN ~ 1000 * field1 value1

# ~ 表示非精确裁剪（性能更好）
# 精确裁剪用 MAXLEN 1000（慢）
```

### 2. 读取消息（XREAD）

```bash
# 读取新消息（从最新开始）
XREAD COUNT 10 STREAMS mystream $
  ↑       ↑            ↑        ↑
 命令   数量限制      Stream名  起始位置

# $ 表示只读取新消息
# 0 表示从头读取所有消息

# 示例
XREAD COUNT 10 STREAMS mystream 0
→ 1) 1) "mystream"
     2) 1) 1) "1717488000000-0"
           2) 1) "field1"
              2) "value1"
        2) 1) "1717488001000-0"
           2) 1) "field1"
              2) "value3"
```

**阻塞读取**：
```bash
# 等待 5000 毫秒，有消息立即返回
XREAD COUNT 10 BLOCK 5000 STREAMS mystream $

# BLOCK 0 表示永久等待
XREAD COUNT 10 BLOCK 0 STREAMS mystream $
```

### 3. 消费者组（Consumer Group）⭐

这是 Stream 最强大的特性！

#### 创建消费者组

```bash
# 创建消费者组（从 0 开始）
XGROUP CREATE mystream mygroup 0

# 创建消费者组（从最新消息开始）
XGROUP CREATE mystream mygroup $

# 如果 Stream 不存在则创建
XGROUP CREATE mystream mygroup 0 MKSTREAM
```

#### 消费者组读取

```bash
# 消费者组读取消息
XREADGROUP GROUP mygroup consumer1 COUNT 10 STREAMS mystream >
              ↑         ↑           ↑                          ↑
          组名      消费者名      数量限制                  只读新消息

# > 表示只读取未分配给任何消费者的消息
# 0 表示读取 Pending 列表中的消息（待重新处理）

# 阻塞读取
XREADGROUP GROUP mygroup consumer1 COUNT 10 BLOCK 1000 STREAMS mystream >
```

**消费者组的特点**：
```
1. 同一组内的消费者不会重复消费
2. 消息分配给消费者后进入 Pending 列表
3. 需要 ACK 确认才会从 Pending 移除
4. 未 ACK 的消息可以重新分配
```

### 4. 消息确认（XACK）

```bash
# 确认消息已处理
XACK mystream mygroup 1717488000000-0

# 批量确认
XACK mystream mygroup 1717488000000-0 1717488001000-0 1717488002000-0
```

**为什么需要 ACK？**
```
1. 标记消息已处理
2. 从 Pending 列表移除
3. 防止重复消费
4. 支持消息重试
```

### 5. 查看 Stream 信息

```bash
# Stream 基本信息
XINFO STREAM mystream
→ length: 100              # 消息数量
  first-entry: ...          # 第一条消息
  last-entry: ...           # 最后一条消息

# 查看消费者组
XINFO GROUPS mystream
→ name: mygroup            # 组名
  consumers: 2             # 消费者数量
  pending: 5               # 待确认消息数

# 查看消费者
XINFO CONSUMERS mystream mygroup
→ name: consumer1          # 消费者名
  pending: 3               # 待确认数
  idle: 1234               # 空闲时间（毫秒）
```

---

## 四、Stream 内部结构

### 底层实现

Redis Stream 使用 **Radix Tree（基数树）** 存储消息：

```
Radix Tree 节点：
├─ 171748800 (前缀)
│  ├─ 0000-0 → {field1: "v1"}
│  ├─ 0000-1 → {field1: "v2"}
│  └─ 1000-0 → {field1: "v3"}
└─ 171748801 (前缀)
   ├─ 0000-0 → {field1: "v4"}
   └─ 0000-1 → {field1: "v5"}
```

**优势**：
- 内存压缩（公共前缀共享）
- 快速范围查询（按 ID）
- 高效插入和删除

### Pending Entries List（PEL）

每个消费者组维护一个 PEL：

```
Consumer Group: mygroup
├─ PEL (Pending Entries List)
│  ├─ 1717488000000-0 → consumer1, idle: 1234ms, deliveryCount: 1
│  ├─ 1717488001000-0 → consumer2, idle: 5678ms, deliveryCount: 2
│  └─ 1717488002000-0 → consumer1, idle: 9999ms, deliveryCount: 1
└─ Last Delivered ID: 1717488003000-0
```

**PEL 作用**：
1. 记录已分配但未确认的消息
2. 跟踪消费者处理进度
3. 支持消息重试
4. 检测消费者故障

---

## 五、消费者组详解

### 消息分配机制

```
Stream: mystream
├─ Msg1
├─ Msg2
├─ Msg3
└─ Msg4

Consumer Group: mygroup
├─ Consumer1 → 读取 → 分配到 Msg1, Msg3
└─ Consumer2 → 读取 → 分配到 Msg2, Msg4

特点：
- 同一组内消息不重复
- 负载均衡（轮询分配）
- 独立处理
```

### 多消费者组

```
Stream: mystream
├─ Msg1
└─ Msg2

Group1:
├─ Consumer1 → 处理 Msg1
└─ Consumer2 → 处理 Msg2

Group2:
├─ Consumer3 → 处理 Msg1  ← 同一消息
└─ Consumer4 → 处理 Msg2  ← 不同组可以重复消费

特点：
- 不同组可以消费相同消息
- 类似 Kafka 的 Consumer Group
- 适合多种业务处理同一事件
```

---

## 六、项目中的使用方式

### 1. 发送消息

```java
// RedisService.java:302
public String streamAdd(String streamKey, Map<String, String> message, int maxLen) {
    RStream<String, String> stream = redissonClient.getStream(streamKey, StringCodec.INSTANCE);
    StreamAddArgs<String, String> args = StreamAddArgs.entries(message);
    if (maxLen > 0) {
        args.trimNonStrict().maxLen(maxLen);
    }
    StreamMessageId messageId = stream.add(args);
    return messageId.toString();
}

// 业务代码
redisService.streamAdd(
    "knowledgebase:vectorize:stream",
    Map.of(
        "kbId", "123",
        "content", "文档内容",
        "retryCount", "0"
    ),
    1000  // 最多 1000 条
);

// 对应 Redis 命令
XADD knowledgebase:vectorize:stream MAXLEN ~ 1000 * kbId 123 content "文档内容" retryCount 0
```

### 2. 消费消息

```java
// RedisService.java:231
public boolean streamConsumeMessages(
    String streamKey,
    String groupName,
    String consumerName,
    int count,
    long blockTimeoutMs,
    StreamMessageProcessor processor
) {
    RStream<String, String> stream = redissonClient.getStream(streamKey, StringCodec.INSTANCE);

    Map<StreamMessageId, Map<String, String>> messages = stream.readGroup(
        groupName,
        consumerName,
        StreamReadGroupArgs.neverDelivered()
            .count(count)
            .timeout(Duration.ofMillis(blockTimeoutMs))
    );

    for (Map.Entry<StreamMessageId, Map<String, String>> entry : messages.entrySet()) {
        processor.process(entry.getKey(), entry.getValue());
    }

    return true;
}

// 对应 Redis 命令
XREADGROUP GROUP vectorize-group consumer1 COUNT 10 BLOCK 1000 STREAMS knowledgebase:vectorize:stream >
```

### 3. 完整流程

```
1. 生产者发送消息
   ↓
   XADD knowledgebase:vectorize:stream * kbId 123 content "..." retryCount 0
   ↓
   返回消息ID: 1717488000000-0

2. 消费者读取消息
   ↓
   XREADGROUP GROUP vectorize-group consumer1 COUNT 10 BLOCK 1000 STREAMS knowledgebase:vectorize:stream >
   ↓
   分配消息: {1717488000000-0: {kbId: "123", ...}}
   ↓
   消息进入 PEL（Pending）

3. 处理消息
   ↓
   执行业务逻辑（向量化）
   ↓
   成功/失败

4. 确认消息
   ↓
   XACK knowledgebase:vectorize:stream vectorize-group 1717488000000-0
   ↓
   从 PEL 移除
```

---

## 七、Stream vs 其他消息队列

### 对比表

| 特性 | Redis Stream | Kafka | RabbitMQ | Redis List |
|------|--------------|-------|----------|-----------|
| **持久化** | ✅ AOF/RDB | ✅ 磁盘 | ✅ 磁盘 | ✅ AOF/RDB |
| **消费者组** | ✅ | ✅ | ✅ | ❌ |
| **消息确认** | ✅ XACK | ✅ Commit | ✅ ACK | ❌ |
| **顺序保证** | ✅ 全局 | ✅ 分区内 | ⚠️ Queue 内 | ✅ |
| **历史回溯** | ✅ | ✅ | ❌ | ⚠️ 有限 |
| **吞吐量** | 高 ⚡ | 极高 ⚡⚡ | 中等 | 高 ⚡ |
| **部署复杂度** | 低 ✅ | 高 | 中等 | 低 ✅ |
| **适用场景** | 中小规模异步任务 | 大数据流 | 企业消息总线 | 简单队列 |

### 项目选择 Stream 的原因

```
✅ 已使用 Redis（限流、缓存）
✅ 无需额外部署消息队列
✅ 支持消费者组（负载均衡）
✅ 支持消息确认（可靠性）
✅ 满足中小规模需求（知识库、简历、面试评估）
✅ 简化架构（减少组件）
```

---

## 八、Stream 的优势与限制

### 优势

```
1. 内置在 Redis：无需额外部署
2. 消费者组：负载均衡 + 故障转移
3. 消息确认：可靠投递
4. 持久化：数据不丢失
5. 阻塞读取：高效消费
6. 历史回溯：重新消费旧消息
7. 简单易用：API 清晰
```

### 限制

```
1. 单机性能：受限于 Redis 单线程
2. 集群支持：跨 Slot 操作困难
3. 消息大小：不适合大消息（建议 < 1MB）
4. 顺序保证：仅全局有序（无分区概念）
5. 查询能力：按 ID 范围查询，不支持复杂查询
```

---

## 九、实际应用场景

### 1. 异步任务队列（项目中）

```java
// 知识库向量化
生产者 → XADD knowledgebase:vectorize:stream * kbId 123 content "..."
消费者 → XREADGROUP → 调用 AI 向量化 → XACK

// 简历分析
生产者 → XADD resume:analyze:stream * resumeId 456 content "..."
消费者 → XREADGROUP → AI 评分 → XACK

// 面试评估
生产者 → XADD interview:evaluate:stream * sessionId 789 answer "..."
消费者 → XREADGROUP → AI 评估 → XACK

// 语音面试评估
生产者 → XADD voice:evaluate:stream * voiceSessionId 101 audio "..."
消费者 → XREADGROUP → 语音识别 + AI 评估 → XACK
```

### 2. 事件流处理

```bash
# 用户行为事件流
XADD user:events * userId 123 action "login" timestamp 1717488000000
XADD user:events * userId 123 action "view_page" page "/home"
XADD user:events * userId 123 action "click_button" button "submit"

# 多个消费者组处理
Group1: 实时推荐系统
Group2: 数据分析
Group3: 审计日志
```

### 3. 日志收集

```bash
# 应用日志
XADD app:logs * level "ERROR" message "Database connection failed" timestamp 1717488000000

# 消费者实时处理
- 告警系统：监控 ERROR 级别日志
- 日志存储：写入 Elasticsearch
- 指标统计：计算错误率
```

### 4. 消息通知

```bash
# 用户通知
XADD notifications * userId 123 type "comment" content "Someone commented on your post"

# 推送服务消费
- WebSocket 推送
- 邮件发送
- 短信通知
```

---

## 十、最佳实践

### 1. 消息大小控制

```java
// ❌ 不推荐：大消息
Map<String, String> message = Map.of(
    "content", largeDocument  // 10MB 文档
);

// ✅ 推荐：存储引用
Map<String, String> message = Map.of(
    "documentId", "doc123",
    "s3Key", "documents/doc123.pdf"
);
```

### 2. 设置最大长度

```java
// 防止 Stream 无限增长
redisService.streamAdd(streamKey, message, 1000);  // 最多 1000 条

// 对应命令
XADD mystream MAXLEN ~ 1000 * field value
```

### 3. 消费者命名

```java
// ✅ 推荐：唯一标识
String consumerName = consumerPrefix + UUID.randomUUID().toString().substring(0, 8);
// "vectorize-consumer-a1b2c3d4"

// 便于追踪和调试
```

### 4. 错误处理

```java
try {
    processBusiness(payload);
    markCompleted(payload);
    ackMessage(messageId);
} catch (Exception e) {
    if (retryCount < MAX_RETRY_COUNT) {
        retryMessage(payload, retryCount + 1);  // 重试
    } else {
        markFailed(payload, error);  // 标记失败
    }
    ackMessage(messageId);  // 无论成功失败都要 ACK
}
```

### 5. 监控指标

```bash
# 监控 Stream 长度
XLEN mystream

# 监控 Pending 消息数
XINFO GROUPS mystream

# 监控消费者状态
XINFO CONSUMERS mystream mygroup

# 告警规则
- Stream 长度 > 10000 → 消费者处理慢
- Pending 数量 > 100 → 消息堆积
- Consumer idle > 60000ms → 消费者可能卡死
```

---

## 十一、可视化总结

### Stream 完整架构图

```
┌─────────────────────────────────────────────────┐
│            Redis Stream: mystream               │
├─────────────────────────────────────────────────┤
│ 1717488000000-0 → {field1: "v1", field2: "v2"} │ ← 消息1
│ 1717488001000-0 → {field1: "v3", field2: "v4"} │ ← 消息2
│ 1717488002000-0 → {field1: "v5", field2: "v6"} │ ← 消息3
│ 1717488003000-0 → {field1: "v7", field2: "v8"} │ ← 消息4
└─────────────────────────────────────────────────┘
              ↓ XREADGROUP
┌─────────────────────────────────────────────────┐
│        Consumer Group: mygroup                  │
├─────────────────────────────────────────────────┤
│ Last Delivered ID: 1717488003000-0              │
│                                                 │
│ PEL (Pending):                                  │
│ ├─ 1717488000000-0 → consumer1, idle: 1234ms   │
│ ├─ 1717488001000-0 → consumer2, idle: 5678ms   │
│ └─ 1717488002000-0 → consumer1, idle: 9999ms   │
└─────────────────────────────────────────────────┘
         ↓ XACK                    ↓ XACK
    Consumer1                  Consumer2
```

### 项目中的 Stream 应用

```
业务层
  ↓
AbstractStreamProducer
  ↓ streamAdd()
┌─────────────────────────────────────────────┐
│     Redis Stream                            │
│  - knowledgebase:vectorize:stream           │
│  - resume:analyze:stream                    │
│  - interview:evaluate:stream                │
│  - voice:evaluate:stream                    │
└─────────────────────────────────────────────┘
  ↓ streamConsumeMessages()
AbstractStreamConsumer
  ↓ processMessage()
业务处理
  ↓ markCompleted() / markFailed()
数据库状态更新
```

---

## 总结

### Redis Stream 核心特点

1. **有序消息流**：按 ID 自动排序，只追加
2. **消费者组**：负载均衡，消息不重复
3. **消息确认**：XACK 机制，可靠投递
4. **持久化**：AOF/RDB 持久化
5. **阻塞读取**：高效消费，实时响应
6. **历史回溯**：可重新消费旧消息

### 适用场景

```
✅ 异步任务队列（项目中：向量化、简历分析、面试评估）
✅ 事件流处理
✅ 日志收集
✅ 消息通知
✅ 数据同步
```

### 在项目中的角色

```
Stream 是项目异步任务处理的核心基础设施：
- 知识库向量化：AI 模型处理耗时，异步执行
- 简历分析：解析 + AI 评分，异步执行
- 面试评估：AI 评估答案，异步执行
- 语音面试评估：语音处理 + AI 评估，异步执行
```

### 关键命令速查

| 命令 | 说明 | 示例 |
|------|------|------|
| `XADD` | 添加消息 | `XADD stream * field value` |
| `XREAD` | 读取消息 | `XREAD COUNT 10 STREAMS stream $` |
| `XREADGROUP` | 消费者组读取 | `XREADGROUP GROUP group consumer COUNT 10 STREAMS stream >` |
| `XACK` | 确认消息 | `XACK stream group messageId` |
| `XGROUP CREATE` | 创建消费者组 | `XGROUP CREATE stream group 0` |
| `XINFO STREAM` | 查看 Stream 信息 | `XINFO STREAM stream` |
| `XINFO GROUPS` | 查看消费者组 | `XINFO GROUPS stream` |
| `XLEN` | 获取长度 | `XLEN stream` |

Redis Stream 提供了**轻量级、高性能、易用**的消息队列能力，在中小规模项目中比较适用
