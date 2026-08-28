# 共识 (Consensus)

## 定义

在沟通管理领域，**共识**是指团队成员通过讨论达成的一致决策或结论，是沟通从分歧到统一的关键产出物。共识具有以下特征：

- **决策性**：共识是经过讨论后形成的明确决策或结论，具有约束力
- **可追溯性**：共识记录了决策的来源、参与人和时间戳，支持决策过程回溯
- **不可篡改性**：共识一旦标记，内容和元数据（时间戳、参与人）不可修改
- **血缘关联**：共识通过关系图与其他共识建立逻辑连接，形成决策网络

在沟通云系统中，共识是沟通流程的核心节点，触发邮件生成和备忘归档等后续动作。

## 领域模型

### Consensus（共识）

- **id** (UUID)：共识的唯一标识符
- **title** (String)：共识的标题，简要描述决策内容
- **description** (String)：共识的详细描述，包含决策的背景、依据和具体内容
- **created_at** (DateTime)：共识创建时间（标记共识的时间戳）
- **updated_at** (DateTime)：共识最后更新时间（通常与 created_at 相同，因为共识不可篡改）

### ConsensusRelation（共识关系）

- **id** (UUID)：关系的唯一标识符
- **from** (UUID)：关系起点的共识 ID
- **to** (UUID)：关系终点的共识 ID
- **relation_type** (String)：关系类型（如：支持、反对、补充、前置条件等）

**说明**：ConsensusRelation 用于建立共识之间的逻辑关联，形成决策图谱。例如，一个共识可能是另一个共识的前置条件，或者两个共识存在支持或反对关系。

### ConsensusGraph（共识图）

- **id** (UUID)：共识图的唯一标识符
- **nodes** (List<Consensus>)：图中的共识节点列表
- **edges** (List<ConsensusRelation>)：图中的关系边列表
- **created_at** (DateTime)：共识图创建时间
- **updated_at** (DateTime)：共识图最后更新时间

**说明**：ConsensusGraph 是多个 Consensus 以 DAG（有向无环图）结构组织的集合，表示共识之间的复杂关联关系。图中的节点是共识，边是共识关系，支持多父节点和多子节点，形成有向无环的决策网络。共识图按时间轴排列，确保因果关系的正确性。

## API 设计

### Consensus API

#### 创建共识
- **POST** `/api/v1/consensuses`
- **请求体**：
  ```json
  {
    "title": "string",
    "description": "string"
  }
  ```
- **响应**：`201 Created`
  ```json
  {
    "id": "uuid",
    "title": "string",
    "description": "string",
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 获取共识详情
- **GET** `/api/v1/consensuses/{id}`
- **响应**：`200 OK`
  ```json
  {
    "id": "uuid",
    "title": "string",
    "description": "string",
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 更新共识
- **PUT** `/api/v1/consensuses/{id}`
- **请求体**：
  ```json
  {
    "title": "string",
    "description": "string"
  }
  ```
- **响应**：`200 OK`
  ```json
  {
    "id": "uuid",
    "title": "string",
    "description": "string",
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 删除共识
- **DELETE** `/api/v1/consensuses/{id}`
- **响应**：`204 No Content`

#### 获取共识列表
- **GET** `/api/v1/consensuses`
- **查询参数**：
  - `page` (int, optional): 页码，默认 1
  - `page_size` (int, optional): 每页数量，默认 20
  - `sort_by` (string, optional): 排序字段，默认 `created_at`
  - `sort_order` (string, optional): 排序方向，默认 `desc`
- **响应**：`200 OK`
  ```json
  {
    "items": [
      {
        "id": "uuid",
        "title": "string",
        "description": "string",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "total": "integer",
    "page": "integer",
    "page_size": "integer"
  }
  ```

### ConsensusRelation API

#### 创建共识关系
- **POST** `/api/v1/consensus-relations`
- **请求体**：
  ```json
  {
    "from": "uuid",
    "to": "uuid",
    "relation_type": "string"
  }
  ```
- **响应**：`201 Created`
  ```json
  {
    "id": "uuid",
    "from": "uuid",
    "to": "uuid",
    "relation_type": "string"
  }
  ```

#### 获取共识关系详情
- **GET** `/api/v1/consensus-relations/{id}`
- **响应**：`200 OK`
  ```json
  {
    "id": "uuid",
    "from": "uuid",
    "to": "uuid",
    "relation_type": "string"
  }
  ```

#### 删除共识关系
- **DELETE** `/api/v1/consensus-relations/{id}`
- **响应**：`204 No Content`

#### 获取共识的关系列表
- **GET** `/api/v1/consensuses/{id}/relations`
- **查询参数**：
  - `direction` (string, optional): 关系方向，`outgoing`（出边）、`incoming`（入边）、`all`（所有），默认 `all`
  - `relation_type` (string, optional): 关系类型筛选
- **响应**：`200 OK`
  ```json
  {
    "items": [
      {
        "id": "uuid",
        "from": "uuid",
        "to": "uuid",
        "relation_type": "string"
      }
    ],
    "total": "integer"
  }
  ```

### ConsensusGraph API

#### 创建共识图
- **POST** `/api/v1/consensus-graphs`
- **请求体**：
  ```json
  {
    "name": "string",
    "description": "string"
  }
  ```
- **响应**：`201 Created`
  ```json
  {
    "id": "uuid",
    "name": "string",
    "description": "string",
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 获取共识图详情
- **GET** `/api/v1/consensus-graphs/{id}`
- **响应**：`200 OK`
  ```json
  {
    "id": "uuid",
    "name": "string",
    "description": "string",
    "nodes": [
      {
        "id": "uuid",
        "title": "string",
        "description": "string",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "edges": [
      {
        "id": "uuid",
        "from": "uuid",
        "to": "uuid",
        "relation_type": "string"
      }
    ],
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 向共识图添加节点
- **POST** `/api/v1/consensus-graphs/{id}/nodes`
- **请求体**：
  ```json
  {
    "consensus_id": "uuid"
  }
  ```
- **响应**：`200 OK`
  ```json
  {
    "id": "uuid",
    "name": "string",
    "description": "string",
    "nodes": [
      {
        "id": "uuid",
        "title": "string",
        "description": "string",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "edges": [
      {
        "id": "uuid",
        "from": "uuid",
        "to": "uuid",
        "relation_type": "string"
      }
    ],
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 从共识图移除节点
- **DELETE** `/api/v1/consensus-graphs/{id}/nodes/{consensus_id}`
- **响应**：`200 OK`
  ```json
  {
    "id": "uuid",
    "name": "string",
    "description": "string",
    "nodes": [
      {
        "id": "uuid",
        "title": "string",
        "description": "string",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "edges": [
      {
        "id": "uuid",
        "from": "uuid",
        "to": "uuid",
        "relation_type": "string"
      }
    ],
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 向共识图添加边
- **POST** `/api/v1/consensus-graphs/{id}/edges`
- **请求体**：
  ```json
  {
    "relation_id": "uuid"
  }
  ```
- **响应**：`200 OK`
  ```json
  {
    "id": "uuid",
    "name": "string",
    "description": "string",
    "nodes": [
      {
        "id": "uuid",
        "title": "string",
        "description": "string",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "edges": [
      {
        "id": "uuid",
        "from": "uuid",
        "to": "uuid",
        "relation_type": "string"
      }
    ],
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 从共识图移除边
- **DELETE** `/api/v1/consensus-graphs/{id}/edges/{relation_id}`
- **响应**：`200 OK`
  ```json
  {
    "id": "uuid",
    "name": "string",
    "description": "string",
    "nodes": [
      {
        "id": "uuid",
        "title": "string",
        "description": "string",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "edges": [
      {
        "id": "uuid",
        "from": "uuid",
        "to": "uuid",
        "relation_type": "string"
      }
    ],
    "created_at": "datetime",
    "updated_at": "datetime"
  }
  ```

#### 获取共识图的拓扑排序
- **GET** `/api/v1/consensus-graphs/{id}/topological-sort`
- **响应**：`200 OK`
  ```json
  {
    "sorted_nodes": [
      {
        "id": "uuid",
        "title": "string",
        "description": "string",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ]
  }
  ```

#### 获取共识图的路径
- **GET** `/api/v1/consensus-graphs/{id}/paths`
- **查询参数**：
  - `from` (uuid, required): 起始共识 ID
  - `to` (uuid, required): 目标共识 ID
- **响应**：`200 OK`
  ```json
  {
    "paths": [
      [
        {
          "id": "uuid",
          "title": "string",
          "description": "string",
          "created_at": "datetime",
          "updated_at": "datetime"
        }
      ]
    ]
  }
  ```