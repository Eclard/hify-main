# Hify

Hify 是一个面向大语言模型应用开发的 AI Agent 可编排服务系统。项目提供模型服务管理、Agent 配置、流式对话、知识库流程、MCP 工具接入和节点式工作流编排能力，并包含前端管理界面与容器化部署配置。

> 项目源码位于 [`hify-main/`](./hify-main)。

## 功能概览

| 模块 | 当前能力 |
| --- | --- |
| Provider | 管理模型服务与模型配置，支持连接测试；适配 OpenAI、Anthropic、Azure OpenAI、Ollama、OpenAI Compatible/DeepSeek 等类型。 |
| Agent | 配置系统提示词、模型参数、上下文轮数，并关联知识库、MCP 工具和工作流。 |
| Chat | 基于 SSE 的流式多轮对话；会话和消息持久化，Redis 缓存会话上下文。 |
| Knowledge | 知识库与文档管理、异步处理、文本分块和对话上下文注入。 |
| MCP | MCP Server 注册、连通性测试、工具发现、参数 Schema 获取、工具调试与调用。 |
| Workflow | 支持 START、LLM、知识、条件、API 调用和 END 节点的工作流执行及运行记录。 |
| 运维 | Docker Compose、Nginx、Kubernetes 清单、健康检查、Prometheus 指标和结构化日志。 |

## 技术栈

- 后端：Java 17、Spring Boot 3.2.3、MyBatis-Plus
- 前端：Vue 3、Vite、Element Plus
- 数据与缓存：MySQL、Redis；预留 PostgreSQL/pgvector 连接配置
- 可靠性与观测：Resilience4j、Micrometer Prometheus、OpenTelemetry API
- 交付：Docker、Docker Compose、Nginx、Kubernetes

## 目录结构

```text
hify-main/
├── hify-app/        # 应用启动、运行配置和数据库脚本
├── hify-common/     # 通用配置、异常、日志、指标与线程池
├── hify-provider/   # 模型服务与适配器
├── hify-agent/      # Agent 配置与工具绑定
├── hify-chat/       # 会话、消息与 SSE 流式对话
├── hify-knowledge/  # 知识库与文档处理流程
├── hify-mcp/        # MCP Server 与工具调用
├── hify-workflow/   # 工作流定义与执行引擎
├── hify-web/        # Vue 前端
├── deploy/          # Docker、Kubernetes 与环境配置模板
├── docker-compose.yml
└── pom.xml
```

## 快速开始

### 环境要求

- JDK 17
- Maven 3.9+
- Node.js 18+
- MySQL、Redis（完整环境运行时需要）
- Docker 与 Docker Compose（容器化部署时需要）

### 本地 Mock 模式

Mock Profile 用于本地体验基础功能，不依赖真实模型服务。

启动后端：

```bash
cd hify-main
mvn spring-boot:run -pl hify-app -Dspring-boot.run.profiles=mock
```

在另一个终端启动前端：

```bash
cd hify-main/hify-web
npm install
npm run dev
```

### 完整环境与容器部署

1. 参考 `hify-main/deploy/env.template` 创建并填写本地环境变量文件。
2. 如需覆盖应用配置，参考 `hify-main/deploy/application.yml.template`。
3. 在源码目录执行：

```bash
cd hify-main
docker compose up -d --build
```

默认情况下，前端服务监听 `80` 端口，后端服务监听 `8080` 端口。端口可通过环境变量覆盖。

## 构建与验证

后端打包：

```bash
cd hify-main
mvn -q -DskipTests package
```

前端构建：

```bash
cd hify-main/hify-web
npm install
npm run build
```

应用健康检查：

```text
GET http://localhost:8080/api/v1/health
```

## 当前实现边界

知识库模块已实现知识库管理、文档上传、txt/md 内容读取、异步处理、Mock 分块、Mock 检索和对话上下文注入流程。

真实 Embedding、pgvector 相似度检索、PDF 内容解析、Query Rewrite、Multi Query 与 Rerank 尚未在当前源码中实现。项目保留了 pgvector JDBC 依赖和连接配置，相关能力可作为后续演进方向。

## 开发说明

- 不提交 `node_modules`、`target`、`dist`、IDE 配置、本地环境文件和日志；相关规则见 [`.gitignore`](./.gitignore)。
- 前端依赖由 `hify-main/hify-web/package-lock.json` 锁定，克隆项目后请执行 `npm install` 安装。
- 完整部署所需配置请使用 `deploy/` 下模板创建，避免提交真实密码、API Key 或内部服务地址。
