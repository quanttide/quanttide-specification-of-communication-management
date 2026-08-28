# CHANGELOG

所有显著变更都将记录在此文件中。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)。

版本遵循语义化版本规范：0.0.x（探索期）→ 0.x.y（验证期）→ x.y.z（正式期）

---

## [v0.1.0] - 2026-08-28

### Added

- 增加语境、媒介、内容定义文档
- 增加消息、备忘、邮件定义文档
- 增加共识定义文档，包含 Consensus、ConsensusRelation 和 ConsensusGraph 领域模型
- 增加共识定义文档 API 规格，包括 Consensus、ConsensusRelation 和 ConsensusGraph 的 RESTful API
- 增加领域事件部分，包括9个事件类型
- 增加 AGENTS.md Agent工作指南
- 为所有API添加使用场景说明
- 修改示例JSON文件，以“团队讨论如何建模沟通管理标准”为语境，以“团队定义了共识等概念”为例子

### Changed

- 修改共识定义文档数据结构格式：表格改为列表
- 修改共识定义文档标题：数据结构改为领域模型
- 将 ConsensusChain 替换为 ConsensusGraph
- 修改共识图API命名风格
- 将拓扑排序设为共识图详情API的默认行为
- 修改路径查询API为查询参数方式
- 修改共识定义文档标题：API规范改为API规格

### Fixed

- 无

### Removed

- 无
