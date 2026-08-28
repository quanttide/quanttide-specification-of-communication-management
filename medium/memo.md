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
- **status** (String)：备忘的状态，枚举值：`initial`（初始）、`active`（活跃）、`consensual`（已共识）、`archived`（已归档）
- **created_at** (DateTime)：备忘创建时间
- **updated_at** (DateTime, optional)：备忘最后更新时间

**说明**：Memo 是沟通从分散到收敛的关键中间形态，通过语义聚类将相关消息组织在一起，形成主题明确的讨论单元。备忘假设人机交互，因此不需要 type 字段。

### MemoStatus（备忘状态）

备忘具有明确的状态机，驱动沟通流程推进：

- **initial**：初始状态，AI生成的初始备忘
- **active**：活跃状态，等待团队讨论
- **consensual**：已共识状态，团队已达成一致决策
- **archived**：已归档状态，备忘已完成并归档

状态迁移规则：
```
initial → active → consensual → archived
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
      "status": "initial",
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

- **发生时机**：当备忘状态发生变更时（如从active变为consensual）
- **事件载荷**：
  ```json
  {
    "event_id": "uuid",
    "event_type": "MemoStatusChanged",
    "timestamp": "datetime",
    "data": {
      "memo_id": "uuid",
      "old_status": "active",
      "new_status": "consensual",
      "changed_at": "2026-08-28T14:23:00+08:00"
    }
  }
  ```
- **下游影响**：触发邮件生成、共识标记等后续操作

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
    "description": "讨论支付超时的处理方案"
  }
  ```
- **响应**：`201 Created`
  ```json
  {
    "id": "memo0a80101-0000-0000-0000-000000000001",
    "title": "支付超时处理方案",
    "description": "讨论支付超时的处理方案",
    "status": "initial",
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
    "status": "active",
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
    "status": "active",
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
        "status": "active",
        "created_at": "2026-08-28T02:05:00+08:00"
      },
      {
        "id": "memo0a80101-0000-0000-0000-000000000002",
        "title": "一键重试功能",
        "description": "讨论一键重试功能的实现方案",
        "status": "active",
        "created_at": "2026-08-28T02:04:00+08:00"
      },
      {
        "id": "memo0a80101-0000-0000-0000-000000000001",
        "title": "支付超时处理方案",
        "description": "讨论支付超时的处理方案",
        "status": "consensual",
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
- **使用场景**：变更备忘状态（如从active变为consensual）
- **请求体**：
  ```json
  {
    "status": "consensual"
  }
  ```
- **响应**：`200 OK`
  ```json
  {
    "id": "memo0a80101-0000-0000-0000-000000000001",
    "title": "支付超时处理方案",
    "description": "讨论支付超时的处理方案",
    "status": "consensual",
    "messages": ["m0a80101-0000-0000-0000-000000000001"],
    "created_at": "2026-08-28T02:03:00+08:00",
    "updated_at": "2026-08-28T14:23:00+08:00"
  }
  ```



## 工程实现

### Go 包实现

备忘模型在 Go 包中的定义：

```go
// Memo 是备忘，对相关消息进行语义聚类后形成的结构化讨论容器。
type Memo struct {
    ID          string `json:"id"`
    Title       string `json:"title"`
    Description string `json:"description,omitempty"`
    Status      string `json:"status"`                // "initial" / "active" / "consensual" / "archived"
    CreatedAt   string `json:"created_at"`
    UpdatedAt   string `json:"updated_at,omitempty"`
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
    initial = "initial"
    active = "active"
    consensual = "consensual"
    archived = "archived"

class Memo(BaseModel):
    """备忘，对相关消息进行语义聚类后形成的结构化讨论容器。"""
    id: str = Field(default_factory=_new_id)
    title: str
    description: str | None = None
    status: MemoStatus = MemoStatus.initial
    created_at: datetime = Field(default_factory=_utcnow)
    updated_at: datetime | None = None
```

### 数据约束

1. **ID 生成**：使用 UUID 格式，确保全局唯一性
2. **时间格式**：使用 ISO 8601 格式（`2026-08-28T02:03:00+08:00`）
3. **状态枚举**：只允许 `initial`、`active`、`consensual`、`archived` 四个值
4. **可选字段**：`description` 和 `updated_at` 字段可省略，使用 `omitempty` 标签

### 与消息的关系

备忘暂时不关联消息，专注于讨论容器的核心功能。

备忘假设人机交互，因此不需要 `type` 字段，所有消息都来自用户或AI代理。
