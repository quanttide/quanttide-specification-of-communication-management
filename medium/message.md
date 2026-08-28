# 消息 (Message)

## 定义

在沟通管理领域，**消息**是指通过特定媒介传递的离散信息单元，是沟通的基本载体。消息具有以下特征：

- **原子性**：消息是不可分割的最小信息单位，承载单一意图或内容片段
- **可追溯性**：每条消息都有唯一标识、时间戳和来源信息，支持全链路追溯
- **多模态性**：消息可以是文本、语音、图像、视频等多种形式
- **状态可变性**：消息在生命周期中会经历不同状态（如：雨水池 → 备忘区 → 共识区 → 归档）

在沟通云系统中，消息是信息流转的起点，经过AI聚类、人工讨论、共识标记等阶段，最终形成可归档的沟通记录。

## 领域模型

### Message（消息）

- **id** (String)：消息的唯一标识符，UUID格式
- **content** (String)：消息的内容，支持文本、语音、图像等多种形式
- **type** (String)：消息的类型，枚举值：`user`（用户消息）、`agent`（AI代理消息）、`system`（系统消息）
- **created_at** (DateTime)：消息创建时间
- **updated_at** (DateTime, optional)：消息最后更新时间

**说明**：Message 是沟通管理领域的基础数据单元，所有其他概念（如共识、备忘、邮件）都建立在消息之上。消息一旦创建，内容不可篡改，确保沟通记录的完整性。

## 领域事件

### MessageCreated（消息已创建）

- **发生时机**：当用户或系统提交新消息时
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MessageCreated",
    "timestamp": "datetime",
    "data": {
      "message_id": "uuid",
      "content": "string",
      "type": "user",
      "created_at": "datetime"
    }
  }
  ```
- **下游影响**：触发AI聚类、消息分类、备忘生成等后续操作

### MessageUpdated（消息已更新）

- **发生时机**：当消息的元数据（如状态）更新时（内容不可修改）
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MessageUpdated",
    "timestamp": "datetime",
    "data": {
      "message_id": "uuid",
      "updated_at": "datetime"
    }
  }
  ```
- **下游影响**：触发消息列表刷新、状态同步等操作

### MessageDeleted（消息已删除）

- **发生时机**：当删除错误创建或无效的消息时
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MessageDeleted",
    "timestamp": "datetime",
    "data": {
      "message_id": "uuid"
    }
  }
  ```
- **下游影响**：触发消息列表更新、相关关联清理等操作

## API 规格

### Message API

#### 创建消息
- **POST** `/messages`
- **使用场景**：当用户或系统提交新消息时，记录沟通内容
- **请求体**：
  ```json
  {
    "content": "支付失败的时候，用户应该能一键重试...",
    "type": "user"
  }
  ```
- **响应**：`201 Created`
  ```json
  {
    "id": "m0a80101-0000-0000-0000-000000000001",
    "content": "支付失败的时候，用户应该能一键重试...",
    "type": "user",
    "created_at": "2026-08-28T09:17:00+08:00"
  }
  ```

#### 获取消息详情
- **GET** `/messages/{id}`
- **使用场景**：查看消息的详细内容和元数据
- **响应**：`200 OK`
  ```json
  {
    "id": "m0a80101-0000-0000-0000-000000000001",
    "content": "支付失败的时候，用户应该能一键重试...",
    "type": "user",
    "created_at": "2026-08-28T09:17:00+08:00"
  }
  ```

#### 删除消息
- **DELETE** `/messages/{id}`
- **使用场景**：删除错误创建或无效的消息（需谨慎操作）
- **响应**：`204 No Content`

#### 获取消息列表
- **GET** `/messages`
- **使用场景**：分页浏览所有消息，支持按时间排序查看沟通历史
- **查询参数**：
  - `page` (int, optional): 页码，默认 1
  - `page_size` (int, optional): 每页数量，默认 20
  - `sort_by` (string, optional): 排序字段，默认 `created_at`
  - `sort_order` (string, optional): 排序方向，默认 `desc`
  - `type` (string, optional): 按消息类型筛选
- **响应**：`200 OK`
  ```json
  {
    "items": [
      {
        "id": "m0a80101-0000-0000-0000-000000000003",
        "content": "方案一，改动小，风险可控。",
        "type": "user",
        "created_at": "2026-08-28T14:21:00+08:00"
      },
      {
        "id": "m0a80101-0000-0000-0000-000000000002",
        "content": "支付超时的处理方案需要讨论一下。",
        "type": "user",
        "created_at": "2026-08-28T14:20:00+08:00"
      },
      {
        "id": "m0a80101-0000-0000-0000-000000000001",
        "content": "支付失败的时候，用户应该能一键重试...",
        "type": "user",
        "created_at": "2026-08-28T09:17:00+08:00"
      }
    ],
    "total": 3,
    "page": 1,
    "page_size": 20
  }
  ```

## 工程实现

### Go 包实现

消息模型在 Go 包中的定义：

```go
// Message 是消息，沟通的基本载体。
type Message struct {
    ID        string `json:"id"`
    Content   string `json:"content"`
    Type      string `json:"type"`                // "user" / "agent" / "system"
    CreatedAt string `json:"created_at"`
    UpdatedAt string `json:"updated_at,omitempty"`
}
```

导入路径：
```go
import consensus "github.com/quanttide/quanttide-connect-toolkit/packages/go/pkg/consensus"
```

### Python 包实现

消息模型在 Python 包中的定义：

```python
class MessageType(str, Enum):
    user = "user"
    agent = "agent"
    system = "system"

class Message(BaseModel):
    """对话消息，按时间排序。"""
    id: str = Field(default_factory=_new_id)
    content: str
    type: MessageType
    created_at: datetime = Field(default_factory=_utcnow)
    updated_at: datetime | None = None
```

### 数据约束

1. **ID 生成**：使用 UUID 格式，确保全局唯一性
2. **时间格式**：使用 ISO 8601 格式（`2026-08-28T09:17:00+08:00`）
3. **类型枚举**：只允许 `user`、`agent`、`system` 三个值
4. **内容不可变**：消息一旦创建，内容字段不可修改
5. **可选字段**：`updated_at` 字段可省略，使用 `omitempty` 标签
