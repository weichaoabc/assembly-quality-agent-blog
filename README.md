# 装配质量分析 Agent：文章与配套资料

本仓库汇集“基于 Amazon Bedrock AgentCore 和 Amazon SageMaker AI 构建装配质量分析 Agent”系列文章及配套资料，方便读者理解方案、查看架构和使用示例数据。

## 下载与阅读

| 资料 | 入口 | 用途 |
| --- | --- | --- |
| 第一篇：完整资料包 | [下载 ZIP](https://github.com/weichaoabc/assembly-quality-agent-blog/releases/download/v2026.09.29/blog-ml-agent-customer-final.zip) | 包含 Word、网页版、配图、示例数据、实践指南及演示视频 |
| 第一篇：在线阅读 | [阅读文章](blog-ml-agent-final.md) | 了解业务问题、方案架构、Agent 工具调用流程和中国区域部署安排 |
| 第一篇：Word 文档 | [下载 Word](https://github.com/weichaoabc/assembly-quality-agent-blog/releases/download/v2026.09.29/blog-ml-agent-paper.docx) | 审阅、编辑文章；配套附件请下载完整资料包 |
| 示例数据与字段说明 | [下载 ZIP](https://github.com/weichaoabc/assembly-quality-agent-blog/releases/download/v2026.09.29/blog-ml-agent-dataset.zip) | 查看装配记录、测试标签、字段说明和预测结果 |
| 模型评估示例数据 | [下载 ZIP](https://github.com/weichaoabc/assembly-quality-agent-blog/releases/download/v2026.09.29/blog-ml-agent-test-dataset.zip) | 了解模型评估方法与结果 |
| 部署与操作指南 | [阅读指南](blog-ml-agent-practice-guide.md) | 了解工具接入、环境配置和主要操作流程 |
| 数据与结果说明 | [阅读说明](blog-ml-agent-validation.md) | 查看示例数据的构造方法及结果口径 |

[查看全部版本](https://github.com/weichaoabc/assembly-quality-agent-blog/releases)。每次更新会保留版本记录，便于在博客中引用固定版本的下载地址。

## 方案简介

工程师通过对话提出质量分析需求，Agent 经 MCP 调用封装后的 AutoGluon 工具，完成数据检查、模型训练、评估和预测。AgentCore Runtime 承载 Agent，AgentCore Gateway 提供工具接入；Lambda 接收工具调用并提交 SageMaker AI 训练任务，AutoGluon 在训练容器内完成建模，在线预测由 SageMaker AI 端点承担。

示例使用装配记录和历史下线测试结果训练二分类模型，在发动机首次下线测试前给出不通过的风险分数，帮助质量团队安排额外人工检查。文章包含中国北京和宁夏区域的部署说明。

## 使用资料包

下载并解压完整资料包后，打开 `blog-ml-agent-final.html` 阅读带配图的文章，也可使用 Word 版本。请保留目录结构，以便文章访问配图、数据与指南。可编辑架构图位于 `blog-ml-agent-assets`，提供 draw.io、SVG、PNG 和 DOT 格式。

详细文件说明见 [资料包目录说明](PACKAGE-CONTENTS.md)。

## 资料范围

示例数据由模拟程序生成，用于介绍建模方法与业务流程，不是客户生产数据。本仓库提供文章及配套实践资料，不包含完整 Agent 平台源码。数据、图表与文中的结果口径保持一致。

后续系列文章将在完成整理后加入本仓库。
