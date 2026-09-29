# 装配质量分析 Agent 实践指南

本指南配合方案文章使用，提供本地复算和已有 AWS 平台中的操作步骤。资料包包含示例数据、保存的预测、文章与配图；完整平台源码和部署包需另行获取。

## 本地复算

只希望查看测试数据的读者，可下载[测试集与说明包](blog-ml-agent-test-dataset.zip)。其中 `data/test.csv` 为1600条有标签测试样本，包含14个特征和1个标签；`data/test-arrivals-*.csv` 为1660台完整到达队列，另含60台未知结果对象。包内 README 说明数据来源、字段、时间划分与两种测试口径，并提供原实验保存的预测和复算脚本。

解压 `blog-ml-agent-dataset.zip`，进入生成的 `blog-ml-agent-dataset` 目录。请保留原文件，使用 Python 3.11 或更高版本运行以下命令。

```bash
python3 scripts/verify_package.py
python3 scripts/replay_policy.py
```

两个脚本都使用 Python 标准库，不需要安装 AutoGluon、访问 AWS 或配置凭据。前者核对包内文件哈希，后者读取已保存的预测与固定策略进行复算。复算时不要重新生成输入文件，也不要将新模型的预测混入原结果。

原始输入包括 `data/train.csv`、`data/validation.csv` 和 `data/test.csv`，分别包含4800、1600和1600条有标签记录。模型根据装配记录预测首次下线测试通过或不通过，并输出不通过的风险分数。`test-arrivals-features.csv` 及对应的 `test-arrivals-index.csv` 包含测试期全部1660台发动机，用于回放建议人工检查清单的筛选过程。清单列明发动机编号及风险分数，现场检查由人员实施。特征与索引文件逐行对应，不应分别排序。

预期筛选结果为：模型策略将146台发动机列入建议人工检查清单，扭矩规则筛选143台；两种方法均覆盖22台已知首次失败，对83台已知失败的覆盖率均为26.5%。模型清单另含3台结果未知的发动机。这些数字来自模拟数据的筛选回放，不表示已在现场完成相应数量的人工检查。字段、标签及数据生成方法见数据包中的 `README.md` 和 `data-dictionary.csv`。

## 运行环境准备

AgentCore Runtime 与 Gateway 已支持中国北京和宁夏区域。中国区域部署可保留本文的 MCP 工具接入和 SageMaker AI 计算架构，使用当地账户、服务端点、数据存储和容器镜像，并配置可在目标环境访问且支持工具调用的基础模型服务。

中国区域目前不提供 AgentCore Memory；会话历史和任务记录可保存在 DynamoDB，跨会话历史检索按应用需求实现。Gateway 入站访问应使用 AWS IAM 或符合 OIDC 标准的身份提供方。具体差异见[中国区域 AgentCore 官方说明](https://docs.amazonaws.cn/en_us/aws/latest/userguide/bedrock-agentcore.html)。

配套计算记录使用 `us-east-1` 和 AutoGluon 1.5.0。实施团队应按选定区域确认实例配额、训练及推理镜像，并记录模型与镜像版本。

完整部署依赖主平台和 ML 模块的匹配版本。获得源码后，主项目的 `docs/installation.md` 是平台安装入口，`ml-agent-lambda/README.md` 说明 ML 模块构建、配置及协同上线。主安装入口为 `scripts/install.sh`；ML 模块的 `deploy.sh --prepare` 仅准备本地材料，不等于完整部署。新环境须按两份说明完成接入和独立验收。

平台侧需要门户、Agent Runtime、Gateway、会话存储及身份配置。ML 模块需要输入与模型存储、状态表、登记表、授权密钥、训练和推理角色、工具 Lambda，以及任务和端点状态核对机制。主平台安装完成后，还需连接 ML 模块、配置 Gateway 的 `ml` 目标，并向 Agent 挂载工具和业务 Skill。

主要配置包括 `ML_AGENT_BUCKET`、`JOBS_TABLE`、`MODEL_REGISTRY_TABLE`、`SAGEMAKER_ROLE_ARN`、`SAGEMAKER_SERVING_ROLE_ARN`、`TRAINING_SCRIPT_URI`、`ML_WORKSPACE_ID`、`ML_CONTEXT_KEY_ARN` 和 `ML_AUTH_MODE=required`。门户与 Runtime 使用一致的工作区和可信授权配置。部署者应根据实际账户设置这些值，并核对目标资源及权限。

Gateway 使用发布工具生成的 `gateway-tool-schema.json`；工具 Lambda 使用本地完整参数定义执行严格校验。将函数注册为 `ml` target 后，确认 `tools/list` 返回预期工具，Agent 配置包含正文附录中的 ML 工具和业务 Skill。

本项目的业务审核、签名授权、模型登记及临时资源回收均需要对应模块配置生效。当前 ML 工作台仅对平台管理员开放。Microsoft Entra ID 是企业接入设计，需另行完成应用注册、OIDC 配置、用户映射和端到端登录验证。

## 在已配置平台中运行样例

### 上传数据并核对策略

将数据上传到为本工作区配置的 S3 桶和前缀，保留训练、验证与测试文件的区分。记录实际地址、文件哈希和标签 `first_eol_fail`，不要使用本文中未提供的示例账户或资源标识。

先查询 `get_ml_policy`，确认允许的实例、训练时长、并发和端点限制。由数据人员核对发动机标识、字段含义、时间与标签，再调用 `inspect_dataset` 检查文件结构和标签分布。

### 提交训练

在聊天中提供训练文件地址、标签、资源要求及验证文件地址。每次新的实验生成一个 `request_id`；因网络问题重试原实验时，复用相同编号和配置。

`train_model` 使用 `s3_path`、`label`、`request_id`、`time_limit`、`presets` 等参数。训练时长参数表示拟合预算。下列 JSON 仅展示业务参数格式，其中数据地址和请求编号必须替换为实际值；授权信息由可信门户或 Runtime 附加。

```json
{
  "s3_path": "s3://YOUR_CONFIGURED_BUCKET/quality/train.csv",
  "label": "first_eol_fail",
  "request_id": "YOUR_NEW_EXPERIMENT_ID",
  "time_limit": 120,
  "presets": "medium_quality",
  "instance_type": "ml.m5.xlarge",
  "use_spot": false
}
```

实例必须在工作区允许列表中并具备配额。调用成功后保存实际返回的任务与模型编号，使用 `get_training_status` 查询，直到作业和模型登记均完成。训练结束但登记失败时，应处理原记录，不能把提交成功当作模型已可用。

### 评估与预测

用 `evaluate_model` 指定 `test_s3_path`，并设置 `dataset_role=validation` 完成外部验证。确定模型与策略后，再对保留测试集使用 `dataset_role=test`。比较模型时，保持目标、指标与外部验证数据一致。

`predict` 使用实际 `model_id`、`data_s3_path`，必要时设置 `include_probabilities=true`。人工检查清单的筛选回放需要完整到达队列及其索引，不能用只包含已知标签的 `test.csv` 代替。新的训练可能产生不同预测，应单独保存为新实验。

需要在线验证时，再创建 `purpose=validation` 的临时端点。等待端点可用后，用预先确定的样本调用 `invoke_endpoint`，与同一模型的离线输出比较。记录响应、模型版本和时间，不将一次调用解释为性能压测。

### 清理与保留

验证完成后，通过项目的 `delete_endpoint` 清理所属端点、端点配置和服务模型，并查询确认 `cleanup_complete=true`。若还有训练任务运行，根据任务用途决定保留或停止，停止请求需等待后台确认。

临时端点回收不代表整个方案不再计费。存储、日志、备份及保留的平台资源仍需按用途管理。删除模型或实验输入前，先确认是否需要保留用于复查与审核；完整环境拆除应遵循相应基础设施栈的保留策略。

## 试点记录

为每次验证记录基础模型、Agent 配置、工具和 Skill 版本、容器镜像、输入哈希、资源参数及任务编号。统计任务完成率时，保留全部任务和失败记录，并预先定义成功条件。

质量侧分别记录清单筛选数量、实际人工检查数量、发现的问题、采取的措施和后续测试结果。平台侧记录到可审查结果的耗时、人工介入、重复提交、错误恢复、端点回收和实际费用。可以先生成清单进行影子运行，再根据业务批准开展有限范围的现场人工检查。

## 配套资料

[返回方案文章](blog-ml-agent-final.html)

[完整示例数据](blog-ml-agent-dataset.zip)

[测试集与说明](blog-ml-agent-test-dataset.zip)

[详细实验记录](blog-ml-agent-validation.html)
