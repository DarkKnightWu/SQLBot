# SQLBot 开发者文档

> 本文档为 SQLBot 项目的完整开发文档，旨在帮助开发者快速理解项目架构、进行二次开发与运维部署。

## 文档索引

### 第一部分：产品与需求
- [1.1 产品概述](./01-product-overview.md)
- [1.2 功能需求规格](./02-functional-requirements.md)

### 第二部分：架构设计
- [2.1 系统架构总览](./03-system-architecture.md)
- [2.2 后端架构设计](./04-backend-architecture.md)
- [2.3 前端架构设计](./05-frontend-architecture.md)
- [2.4 AI/LLM 架构设计](./06-ai-architecture.md)
- [2.5 数据库设计](./07-database-design.md)

### 第三部分：详细设计
- [3.1 API 接口文档](./08-api-documentation.md)
- [3.2 核心模块详细设计](./09-core-modules.md)
- [3.3 权限与安全设计](./10-security-design.md)
- [3.4 权限控制实现详解](./23-permission-implementation.md)


### 第四部分：实现指南
- [4.1 后端代码结构](./11-backend-codebase.md)
- [4.2 前端代码结构](./12-frontend-codebase.md)
- [4.3 数据库迁移指南](./13-database-migration.md)
- [4.4 AI 模型接入指南](./14-ai-model-integration.md)
- [4.5 新功能开发模板](./15-development-template.md)

### 第五部分：运维部署
- [5.1 本地开发环境搭建](./16-local-development.md)
- [5.2 Docker 部署指南](./17-docker-deployment.md)
- [5.3 生产环境部署](./18-production-deployment.md)
- [5.4 故障排查手册](./19-troubleshooting.md)

### 第六部分：高级主题
- [6.1 国际化 (i18n) 设计](./20-i18n-design.md)
- [6.2 监控与告警](./21-monitoring-alerting.md)
- [6.3 测试策略](./22-testing-strategy.md)

---

## 快速导航

| 我想... | 推荐阅读 |
| :--- | :--- |
| 快速了解项目 | [产品概述](./01-product-overview.md) |
| 搭建开发环境 | [本地开发环境搭建](./16-local-development.md) |
| 理解技术架构 | [系统架构总览](./03-system-architecture.md) |
| 了解 AI 实现 | [AI/LLM 架构设计](./06-ai-architecture.md) |
| 新增 API 接口 | [新功能开发模板](./15-development-template.md) |
| 接入新的 AI 模型 | [AI 模型接入指南](./14-ai-model-integration.md) |
| 部署到生产环境 | [生产环境部署](./18-production-deployment.md) |
| 配置监控告警 | [监控与告警](./21-monitoring-alerting.md) |
| 排查问题 | [故障排查手册](./19-troubleshooting.md) |

---

## 版本信息

| 项目 | 版本 |
| :--- | :--- |
| SQLBot | v1.5.0 |
| Python | 3.11 |
| FastAPI | 0.115+ |
| Vue | 3.5+ |
| PostgreSQL | 16 (with pgvector) |

---

*文档最后更新时间：2025-12-28*

