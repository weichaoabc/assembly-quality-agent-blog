# 装配质量分析 Agent 文章与配套材料

打开 blog-ml-agent-final.html 阅读完整文章。blog-ml-agent-paper.docx 为论文式排版的 Word，包含摘要、关键词、图表、完整工具和 Skill 附录及参考文献。

文章《基于 Amazon Bedrock AgentCore 和 Amazon SageMaker AI 构建装配质量分析 Agent》介绍客户场景、架构设计、业务使用与部署实践，包含中国区域部署说明。示例用装配记录训练通过或不通过的二分类模型，在首次下线测试前输出风险分数，供质量团队筛选建议人工检查的发动机。Agent 协助调用建模工具和解释结果，现场检查由人员执行。

## 实践资料

blog-ml-agent-practice-guide.html 与对应 Markdown 面向实施团队，提供工具接入、环境配置及操作步骤。本资料包不包含完整平台源码。

blog-ml-agent-dataset.zip 包含模拟装配记录、测试标签、字段说明、预测结果及配套脚本。详细方法与结果见 blog-ml-agent-validation.html。

blog-ml-agent-test-dataset.zip 是从原数据包中逐字节提取的测试资料，含1600条有标签测试样本、1660台完整到达队列、字段字典、保存预测与本地复算脚本；新增 README 介绍来源、生成假设、字段、划分、用途及结果口径。数据来自项目模拟程序，并非公开工业基准或客户生产记录。

Word 的图片已嵌入。数据包、实践指南及验证记录使用相对链接，请将这些文件与 Word 保持在同一目录。Markdown 和网页版本也使用相对资源路径，分享时请一起保留 blog-ml-agent-assets 文件夹。

## 图像与视频

quality-workflow、solution-architecture、evaluation-traceability 和 model-lifecycle 提供 PNG、SVG、DOT 和 draw.io 文件。架构图说明 Agent 经 MCP 调用 ML 工具及其背后的 AutoGluon 计算，标明 Lambda 与 SageMaker 的执行位置，基础模型服务按部署区域选择。评估追溯图说明模型、数据与评估记录的关系。

ui-chat、ui-training、ui-evaluation、ui-governance 使用现有门户回放保存的实际调用与实验记录。正文图5使用评估追溯流程图；ui-evaluation保留完整预测指标，可从验证记录页打开。资源地址已脱敏，业务指标保持原值。临时验证端点已回收。

ml-workflow-replay.mp4 为保存记录的工作台操作回放，带中文字幕与章节导航。网页提供播放器；Word 不包含视频或视频补充材料条目。review-outcomes 为详细实验记录使用的对照图。
