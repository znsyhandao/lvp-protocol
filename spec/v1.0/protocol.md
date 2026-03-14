# Life Validation Protocol (LVP) v1.0

**让AI的记忆变得可信——开放协议，用于智能体记忆验证与冲突裁决**

## 1. 概述

LVP 是一个开放的、去中心化的记忆验证协议，旨在为任何 AI 智能体提供**可信度评分**、**冲突检测**和**裁决溯源**服务。通过 LVP，AI 的记忆不再是黑盒，而是可追溯、可裁决、可信赖的。

## 2. 核心概念

| 概念 | 描述 |
| :--- | :--- |
| **记忆基因 (Memory Gene)** | 一条记忆及其元数据（文本、来源、时间戳、访问频率、情感倾向）。 |
| **结构化记忆 (Structured Memory)** | 记忆的键值对表示，包含实体、属性、值、主题等字段。 |
| **可信度分数 (Confidence Score)** | 0-1之间的数值，表示记忆的可信程度，基于艾宾浩斯遗忘曲线和访问频率计算。 |
| **冲突胶囊 (Conflict Capsule)** | 两条或以上相互矛盾的记忆，及其裁决状态和溯源信息。 |
| **验证者 (Validator)** | 提供LVP服务的节点，通过公开的 `.well-known/validator.json` 宣告自身。 |
| **数据飞轮 (Data Flywheel)** | 通过收集匿名裁决数据，持续优化验证算法，形成网络效应。 |
| **裁决溯源 (Provenance)** | 记录每条裁决的完整历史，包括冲突双方、用户选择、可信度变化等。 |

## 3. 协议发现

任何符合LVP协议的验证者，必须在域名根目录提供以下文件：

### 3.1 `.well-known/validator.json`

```json
{
  "validator_name": "Life Validation Network",
  "version": "1.0.0",
  "description": "Decentralized memory validation service for AI agents",
  "api_endpoint": "https://your-service.com/v1",
  "capabilities": ["confidence_scoring", "conflict_detection", "batch_validation", "resolution_feedback"],
  "fee_model": "freemium",
  "public_key": "pending",
  "docs_url": "https://github.com/your/repo",
  "rate_limits": {
    "free": "100 requests/day",
    "pro": "unlimited"
  }
}

```
4. API 规范
4.1 基础信息
基础路径: /v1

所有请求必须包含 Content-Type: application/json

响应格式统一为 JSON

4.2 可信度验证
POST /v1/validate
验证单条记忆的可信度。

请求体：

json
{
  "text": "我喜欢吃小龙虾",
  "source_type": "user_explicit",  // 可选: user_explicit, user_implicit, ai_inferred, other_ai, web_learned
  "timestamp": "2026-03-14T15:30:00Z",  // 可选，默认为当前时间
  "context": {  // 可选
    "session_id": "sess_123",
    "topic": "food"
  }
}
响应：

json
{
  "confidence": 0.95,
  "source_type": "user_explicit",
  "timestamp": "2026-03-14T15:30:00Z",
  "warnings": ["可信度极高，可作为事实依据"],
  "metadata": {
    "model": "confidence_scorer_v1",
    "half_life_days": 30
  }
}
4.3 批量验证
POST /v1/batch-validate
批量验证多条记忆。

请求体：

json
{
  "memories": [
    {
      "text": "我喜欢吃小龙虾",
      "source_type": "user_explicit"
    },
    {
      "text": "我不爱吃香菜",
      "source_type": "user_implicit"
    }
  ],
  "threshold": 0.6  // 可选，过滤阈值
}
响应：

json
{
  "results": [
    {
      "text": "我喜欢吃小龙虾",
      "confidence": 0.95,
      "source_type": "user_explicit"
    },
    {
      "text": "我不爱吃香菜",
      "confidence": 0.80,
      "source_type": "user_implicit"
    }
  ],
  "average_confidence": 0.875,
  "count": 2
}
4.4 冲突检测
POST /v1/detect-conflict
检测新记忆与现有记忆之间的冲突。

请求体：

json
{
  "new_memories": [
    {
      "text": "我不喜欢吃小龙虾",
      "timestamp": "2026-03-14T15:31:00Z"
    }
  ],
  "existing_memories": [
    {
      "text": "我喜欢吃小龙虾",
      "timestamp": "2026-03-10T10:00:00Z"
    }
  ],
  "threshold": 0.5  // 可选，冲突检测阈值
}
响应：

json
{
  "conflicts": [
    {
      "id": "conflict_001",
      "timestamp": "2026-03-14T15:31:00Z",
      "keyword": "小龙虾",
      "old_memory": "我喜欢吃小龙虾",
      "new_memory": "我不喜欢吃小龙虾",
      "severity": "medium",
      "status": "pending",
      "resolve_command": "/resolve conflict_001 <保留旧|保留新|合并>"
    }
  ],
  "count": 1,
  "metadata": {
    "threshold": 0.5,
    "detector_version": "1.0"
  }
}
4.5 裁决反馈
POST /v1/resolve-feedback
提交用户裁决结果（数据飞轮核心）。

请求体：

json
{
  "conflict_id": "conflict_001",
  "new_memory": "我不喜欢吃小龙虾",
  "old_memory": "我喜欢吃小龙虾",
  "decision": "保留新",  // 可选: 保留旧, 保留新, 合并
  "user_hash": "u_8f3a1b7c",  // 匿名用户ID
  "context": {  // 可选，增强数据价值
    "session_id": "sess_123",
    "sentiment_score": -0.3,
    "time_of_day": "afternoon"
  }
}
响应：

json
{
  "status": "received",
  "conflict_id": "conflict_001",
  "message": "感谢你的反馈，这将帮助改进可信度算法"
}
4.6 裁决溯源
GET /v1/provenance/{conflict_id}
获取特定冲突的完整溯源信息。

响应：

json
{
  "conflict_id": "conflict_001",
  "provenance": {
    "resolved_at": "2026-03-14T15:32:00Z",
    "resolved_by": "u_8f3a1b7c",
    "resolution_type": "preference_conflict",
    "confidence_before": [0.80, 0.95],
    "confidence_after": 0.95,
    "confidence_delta": 0.15,
    "supersedes": "我喜欢吃小龙虾",
    "applicability": {
      "entities": ["小龙虾", "喜欢"],
      "topics": ["food"],
      "sentiment": -0.3
    },
    "evolution_signal": {
      "direction": "forward",
      "magnitude": 0.15,
      "stability": 0.85
    }
  }
}
5. 数据结构规范
5.1 记忆基因 (Memory Gene)
json
{
  "memory_id": "mem_001",
  "text": "我不喜欢吃小龙虾",
  "structured": {
    "entity": "user",
    "attribute": "food_preference",
    "value": "negative",
    "topic": "food",
    "source_type": "user_explicit"
  },
  "status": {
    "current": "active",
    "previous": null,
    "change_reason": null
  },
  "confidence": 0.95,
  "timestamp": "2026-03-14T15:30:00Z",
  "access_count": 1,
  "context": {
    "session_id": "sess_123",
    "sentiment_score": -0.3
  }
}
5.2 冲突胶囊 (Conflict Capsule)
json
{
  "conflict_id": "conflict_001",
  "memories": {
    "new": { "memory_id": "mem_002", "text": "我不喜欢吃小龙虾" },
    "old": { "memory_id": "mem_001", "text": "我喜欢吃小龙虾" }
  },
  "detected_at": "2026-03-14T15:31:00Z",
  "severity": "medium",
  "status": "resolved",
  "resolution": {
    "decision": "保留新",
    "resolved_at": "2026-03-14T15:32:00Z",
    "resolved_by": "u_8f3a1b7c"
  }
}
6. 错误处理
所有API在出错时返回标准HTTP状态码和错误信息：

json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input format",
    "details": [
      {
        "field": "source_type",
        "issue": "must be one of: user_explicit, user_implicit, ai_inferred, other_ai, web_learned"
      }
    ]
  }
}
常见状态码：

400: 请求格式错误

401: 未授权（需要API key）

404: 资源不存在

429: 请求超限

500: 服务器内部错误

7. 安全考虑
7.1 数据匿名化
所有用户相关数据必须匿名化处理，user_hash 应为单向哈希值，不可逆推出原始身份。

7.2 请求限制
免费层建议限制为 100次/天，防止滥用。

7.3 HTTPS 强制
所有生产环境下的验证者必须使用 HTTPS。

8. 版本演进
LVP协议遵循语义化版本：

主版本号：不兼容的API变更

次版本号：向下兼容的功能新增

修订号：向下兼容的问题修复

当前版本：v1.0.0

9. 贡献指南
欢迎任何形式的贡献：

⭐ Star 项目

🐛 提交 Issue

📝 完善文档

🔧 提交 PR

详情请见 CONTRIBUTING.md

10. 许可证
MIT © cqs10
