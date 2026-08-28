# 备忘 (Memo)

## 定义

在沟通管理领域，**备忘**是指对相关消息进行语义聚类后形成的结构化讨论容器，是沟通从分散到收敛的关键中间形态。备忘具有以下特征：

- **聚类性**：备忘将语义相关的消息聚合在一起，形成主题明确的讨论单元
- **讨论性**：备忘包含讨论区和共识区，支持团队成员围绕主题进行深入交流
- **状态驱动**：备忘具有明确的状态机（草稿 → 开放 → 已共识 → 已归档），驱动沟通流程推进
- **血缘可溯**：备忘记录了从原始消息到最终决策的完整血缘链

在沟通云系统中，备忘是连接消息与正式输出的桥梁，承载着团队讨论、共识形成和决策记录的核心功能。

## 领域模型

### Memo（备忘）

- **id** (String)：备忘的唯一标识符，UUID格式
- **title** (String)：备忘的标题，简要描述讨论主题
- **description** (String, optional)：备忘的详细描述
- **status** (String)：备忘的状态，枚举值：`draft`（草稿）、`open`（开放）、`consensus`（已共识）、`archived`（已归档）
- **messages** (List<String>)：备忘中的消息ID列表
- **created_at** (DateTime)：备忘创建时间
- **updated_at** (DateTime, optional)：备忘最后更新时间

**说明**：Memo 是沟通从分散到收敛的关键中间形态，通过语义聚类将相关消息组织在一起，形成主题明确的讨论单元。备忘假设人机交互，因此不需要 type 字段。

### MemoStatus（备忘状态）

备忘具有明确的状态机，驱动沟通流程推进：

- **draft**：草稿状态，AI生成的初始备忘
- **open**：开放状态，等待团队讨论
- **consensus**：已共识状态，团队已达成一致决策
- **archived**：已归档状态，备忘已完成并归档

状态迁移规则：
```
draft → open → consensus → archived
```

## 领域事件

### MemoCreated（备忘已创建）

- **发生时机**：当AI完成语义聚类，生成新的备忘时
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MemoCreated",
    "timestamp": "datetime",
    "data": {
      "memo_id": "uuid",
      "title": "支付超时处理方案",
      "description": "讨论支付超时的处理方案",
      "status": "draft",
      "messages": ["m0a80101-0000-0000-0000-000000000001"],
      "created_at": "2026-08-28T02:03:00+08:00"
    }
  }
  ```
- **下游影响**：触发备忘列表更新、通知相关用户等操作

### MemoUpdated（备忘已更新）

- **发生时机**：当备忘的元数据（如标题、描述）更新时
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MemoUpdated",
    "timestamp": "datetime",
    "data": {
      "memo_id": "uuid",
      "title": "支付超时处理方案",
      "description": "讨论支付超时的处理方案，包括一键重试功能",
      "updated_at": "2026-08-28T14:30:00+08:00"
    }
  }
  ```
- **下游影响**：触发备忘详情更新、备忘列表刷新等操作

### MemoStatusChanged（备忘状态已变更）

- **发生时机**：当备忘状态发生变更时（如从open变为consensus）
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MemoStatusChanged",
    "timestamp": "datetime",
    "data": {
      "memo_id": "uuid",
      "old_status": "open",
      "new_status": "consensus",
      "changed_at": "2026-08-28T14:23:00+08:00"
    }
  }
  ```
- **下游影响**：触发邮件生成、共识标记等后续操作

### MemoMessageAdded（消息已添加到备忘）

- **发生时机**：当新消息添加到备忘时
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MemoMessageAdded",
    "timestamp": "datetime",
    "data": {
      "memo_id": "uuid",
      "message_id": "uuid",
      "added_at": "2026-08-28T14:21:00+08:00"
    }
  }
  ```
- **下游影响**：触发备忘详情更新、消息列表刷新等操作

### MemoMessageRemoved（消息已从备忘移除）

- **发生时机**：当消息从备忘中移除时
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MemoMessageRemoved",
    "timestamp": "datetime",
    "data": {
      "memo_id": "uuid",
      "message_id": "uuid",
      "removed_at": "2026-08-28T14:30:00+08:00"
    }
  }
  ```
- **下游影响**：触发备忘详情更新、消息列表刷新等操作

### MemoDeleted（备忘已删除）

- **发生时机**：当删除错误创建或无效的备忘时
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MemoDeleted",
    "timestamp": "datetime",
    "data": {
      "memo_id": "uuid"
    }
  }
  ```
- **下游影响**：触发备忘列表更新、相关消息清理等操作

## API 规格

### Memo API

#### 创建备忘
- **POST** `/memos`
- **使用场景**：当AI完成语义聚类，生成新的备忘时
- **请求体**：
  ```json
  {
    "title": "支付超时处理方案",
    "description": "讨论支付超时的处理方案",
    "messages": ["m0a80101-0000-0000-0000-000000000001"]
  }
  ```
- **响应**：`201 Created`
  ```json
  {
    "id": "memo0a80101-0000-0000-0000-000000000001",
    "title": "支付超时处理方案",
    "description": "讨论支付超时的处理方案",
    "status": "draft",
    "messages": ["m0a80101-0000-0000-0000-000000000001"],
    "created_at": "2026-08-28T02:03:00+08:00"
  }
  ```

#### 获取备忘详情
- **GET** `/memos/{id}`
- **使用场景**：查看备忘的详细内容、状态和关联消息
- **响应**：`200 OK`
  ```json
  {
    "id": "memo0a80101-0000-0000-0000-000000000001",
    "title": "支付超时处理方案",
    "description": "讨论支付超时的处理方案",
    "status": "open",
    "messages": ["m0a80101-0000-0000-0000-000000000001"],
    "created_at": "2026-08-28T02:03:00+08:00",
    "updated_at": "2026-08-28T14:21:00+08:00"
  }
  ```

#### 更新备忘
- **PUT** `/memos/{id}`
- **使用场景**：更新备忘的标题、描述等元数据
- **请求体**：
  ```json
  {
    "title": "支付超时处理方案（更新）",
    "description": "讨论支付超时的处理方案，包括一键重试功能"
  }
  ```
- **响应**：`200 OK`
  ```json
  {
    "id": "memo0a80101-0000-0000-0000-000000000001",
    "title": "支付超时处理方案（更新）",
    "description": "讨论支付超时的处理方案，包括一键重试功能",
    "status": "open",
    "messages": ["m0a80101-0000-0000-0000-000000000001"],
    "created_at": "2026-08-28T02:03:00+08:00",
    "updated_at": "2026-08-28T14:30:00+08:00"
  }
  ```

#### 删除备忘
- **DELETE** `/memos/{id}`
- **使用场景**：删除错误创建或无效的备忘（需谨慎操作）
- **响应**：`204 No Content`

#### 获取备忘列表
- **GET** `/memos`
- **使用场景**：分页浏览所有备忘，支持按状态筛选
- **查询参数**：
  - `page` (int, optional): 页码，默认 1
  - `page_size` (int, optional): 每页数量，默认 20
  - `sort_by` (string, optional): 排序字段，默认 `created_at`
  - `sort_order` (string, optional): 排序方向，默认 `desc`
  - `status` (string, optional): 按状态筛选
- **响应**：`200 OK`
  ```json
  {
    "items": [
      {
        "id": "memo0a80101-0000-0000-0000-000000000003",
        "title": "异常状态视觉规范",
        "description": "讨论异常状态的视觉规范",
        "status": "open",
        "messages": ["m0a80101-0000-0000-0000-000000000003"],
        "created_at": "2026-08-28T02:05:00+08:00"
      },
      {
        "id": "memo0a80101-0000-0000-0000-000000000002",
        "title": "一键重试功能",
        "description": "讨论一键重试功能的实现方案",
        "status": "open",
        "messages": ["m0a80101-0000-0000-0000-000000000002"],
        "created_at": "2026-08-28T02:04:00+08:00"
      },
      {
        "id": "memo0a80101-0000-0000-0000-000000000001",
        "title": "支付超时处理方案",
        "description": "讨论支付超时的处理方案",
        "status": "consensus",
        "messages": ["m0a80101-0000-0000-0000-000000000001"],
        "created_at": "2026-08-28T02:03:00+08:00"
      }
    ],
    "total": 3,
    "page": 1,
    "page_size": 20
  }
  ```

#### 变更备忘状态
- **PATCH** `/memos/{id}/status`
- **使用场景**：变更备忘状态（如从open变为consensus）
- **请求体**：
  ```json
  {
    "status": "consensus"
  }
  ```
- **响应**：`200 OK`
  ```json
  {
    "id": "memo0a80101-0000-0000-0000-000000000001",
    "title": "支付超时处理方案",
    "description": "讨论支付超时的处理方案",
    "status": "consensus",
    "messages": ["m0a80101-0000-0000-0000-000000000001"],
    "created_at": "2026-08-28T02:03:00+08:00",
    "updated_at": "2026-08-28T14:23:00+08:00"
  }
  ```

#### 添加消息到备忘
- **POST** `/memos/{id}/messages`
- **使用场景**：将新消息添加到备忘中
- **请求体**：
  ```json
  {
    "message_id": "m0a80101-0000-0000-0000-000000000002"
  }
  ```
- **响应**：`200 OK`
  ```json
  {
    "id": "memo0a80101-0000-0000-0000-000000000001",
    "title": "支付超时处理方案",
    "description": "讨论支付超时的处理方案",
    "status": "open",
    "messages": ["m0a80101-0000-0000-0000-000000000001", "m0a80101-0000-0000-0000-000000000002"],
    "created_at": "2026-08-28T02:03:00+08:00",
    "updated_at": "2026-08-28T14:21:00+08:00"
  }
  ```

#### 从备忘移除消息
- **DELETE** `/memos/{id}/messages/{message_id}`
- **使用场景**：从备忘中移除消息
- **响应**：`200 OK`
  ```json
  {
    "id": "memo0a80101-0000-0000-0000-000000000001",
    "title": "支付超时处理方案",
    "description": "讨论支付超时的处理方案",
    "status": "open",
    "messages": ["m0a80101-0000-0000-0000-000000000001"],
    "created_at": "2026-08-28T02:03:00+08:00",
    "updated_at": "2026-08-28T14:30:00+08:00"
  }
  ```

## 工程实现

### Go 包实现

备忘模型在 Go 包中的定义：

```go
// Memo 是备忘，对相关消息进行语义聚类后形成的结构化讨论容器。
type Memo struct {
    ID          string   `json:"id"`
    Title       string   `json:"title"`
    Description string   `json:"description,omitempty"`
    Status      string   `json:"status"`                // "draft" / "open" / "consensus" / "archived"
    Messages    []string `json:"messages"`
    CreatedAt   string   `json:"created_at"`
    UpdatedAt   string   `json:"updated_at,omitempty"`
}
```

导入路径：
```go
import consensus "github.com/quanttide/quanttide-connect-toolkit/packages/go/pkg/consensus"
```

### Python 包实现

备忘模型在 Python 包中的定义：

```python
class MemoStatus(str, Enum):
    draft = "draft"
    open = "open"
    consensus = "consensus"
    archived = "archived"

class Memo(BaseModel):
    """备忘，对相关消息进行语义聚类后形成的结构化讨论容器。"""
    id: str = Field(default_factory=_new_id)
    title: str
    description: str | None = None
    status: MemoStatus = MemoStatus.draft
    messages: list[str] = []
    created_at: datetime = Field(default_factory=_utcnow)
    updated_at: datetime | None = None
```

### 数据约束

1. **ID 生成**：使用 UUID 格式，确保全局唯一性
2. **时间格式**：使用 ISO 8601 格式（`2026-08-28T02:03:00+08:00`）
3. **状态枚举**：只允许 `draft`、`open`、`consensus`、`archived` 四个值
4. **消息关联**：`messages` 字段存储消息ID列表，支持多对多关系
5. **可选字段**：`description` 和 `updated_at` 字段可省略，使用 `omitempty` 标签

### 与消息的关系

备忘与消息是多对多关系：
- 一个备忘可以包含多个消息
- 一个消息可以属于多个备忘（理论上，但实际中通常只属于一个）

备忘假设人机交互，因此不需要 `type` 字段，所有消息都来自用户或AI代理。
