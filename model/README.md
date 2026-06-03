# MedLLM / NeuroMultiAgentSystem

## 🏥 多智能体深度检索医疗辅助系统

**MedLLM** (NeuroMultiAgentSystem) 是一个面向脑卒中临床场景的智能医疗辅助系统，旨在通过人工智能技术提升临床辅助决策支持能力。系统以大语言模型为核心，融合检索增强生成（RAG）、多智能体推理与医学知识库，实现了从症状输入到辅助分析与建议输出的完整闭环。

与传统通用问答系统不同，本项目并非简单依赖模型生成结果，而是以权威医学文献与临床指南为知识底座，通过"检索—重排—推理—评估"的结构化流程，使每一次回答都具备明确证据来源与逻辑依据。系统强调**"证据先行、过程可解释、结果可验证"**，在保证智能化水平的同时，显著提升了医学场景下的可靠性与安全性。

---

## 🌟 项目亮点与创新

### 1. 证据驱动的医学推理范式与定制 RAG
- **证据前置策略**：将证据获取前置，实现了从"模型主导"向"证据主导"的转变，显著降低幻觉风险
- **混合检索引擎**：基于向量检索（ChromaDB）+ 关键词检索（BM25）的混合策略，优先返回权威指南与最新文献
- **精准信息提取**：强化对脑卒中时间窗、溶栓/取栓指征等关键临床信息的提取能力
- **智能QA生成**：自动从医疗PDF中提炼高质量QA对，提升检索召回率

### 2. 医疗安全三角三层架构
- **外层（流程控制）**：LangGraph定义医疗业务状态图，强制关键节点审批，确保流程完整性
- **中层（多专家协作）**：全科医生、神经专科医生、临床药师协同推理，模拟真实医疗团队
- **后层（双重校验）**：规则引擎 + LLM反思的双重校验机制，确保输出安全可靠

### 3. 多智能体协同的安全推理机制
- **Proposer（生成智能体）**：基于证据生成初步诊疗方案
- **Critic（审查智能体）**：独立执行风险审查，识别临床高风险点（如时间窗陷阱、禁忌症）
- **Integrator（整合反思智能体）**：融合反思生成最终安全结论，模拟"提出方案→上级把关→最终复盘"的真实流程

### 4. 配置化与可扩展架构
- **动态专家配置**：支持灵活配置专家角色、职责和系统提示词
- **规则引擎优化**：禁忌症规则和校验参数可配置，支持规则热更新
- **参数化配置**：所有关键参数和关键词可配置，便于性能调优
- **向后兼容**：提供平滑的升级路径和兼容接口

### 5. 大模型高级QA自建引擎与重排优化
- **批处理扩写**：系统精读医疗PDF并自动提炼生成高质量QA对
- **深度重排**：结合阿里`gte-rerank`进行深度语境打分与证据压缩
- **明确溯源**：在引用中进行文献与页码明确溯源，提升可信度

### 6. 工程化生态与全链路流式推送
- **可见思考过程**：Thinking Step推理展示，让AI思考过程透明化
- **动态多路分发**：支持并发处理和实时响应
- **架构闭环**：结合WebFlux与Redis并发控制的后台以及Vue3流式渲染界面

---

## 🏗️ 医疗安全三角架构详解

### 架构设计理念

本系统采用**"医疗安全三角"**三层架构，在**安全、专业、灵活**上取得最佳平衡，便于通过医疗合规审查。

### 三层架构说明

#### 🔷 外层：LangGraph定义医疗业务状态图
**职责**：定义完整的医疗业务流程和状态流转

**核心功能**：
- 强制关键节点人工/规则审批
- 控制哪些步骤可以自动，哪些必须停等
- 状态机管理，确保流程完整性
- 支持中断点和人工干预

**安全价值**：
- 流程可追溯，每一步都有明确的状态记录
- 关键决策点可强制人工审核
- 支持断点续传和状态恢复
- 符合医疗合规要求

#### 🔷 中层：领域专家多Agent团队
**职责**：通过多专家协作生成初步治疗建议

**核心功能**：
- 全科医生 + 专科医生 + 临床药师协同工作
- 支持Tree-of-Thoughts（疑难分支搜索）模式
- 并行推理，提高效率
- 专家角色可动态配置和扩展

**专业价值**：
- 模拟真实医疗团队协作模式
- 不同专家视角互补，减少盲点
- 支持复杂病例的多维度分析

#### 🔷 后层：Reflection + 规则引擎
**职责**：对所有输出进行医学知识图谱校验和禁忌症检查

**核心功能**：
- 静态规则引擎检查（禁忌症硬规则）
- 动态LLM反思校验（深层次医学逻辑审查）
- 反思修正机制，必要时拉回上一层重试
- 安全警告和风险提示

**安全价值**：
- 双层校验机制（规则引擎 + LLM反思）
- 禁忌症自动检测和拦截
- 支持反思循环，持续优化方案
- 最终报告包含安全警告信息

### 协同工作流程

```
用户输入病例
    ↓
【外层】意图识别
    ↓
【外层】病例分析
    ↓
【外层】证据检索
    ↓
【中层】多专家推理 ←───┐
    ├─ 全科医生建议     │
    ├─ 专科医生建议     │
    ├─ 临床药师建议     │
    └─ 意见综合         │
    ↓
【后层】结果校验        │
    ├─ 规则引擎检查     │
    └─ LLM反思校验      │
    ↓
    校验通过？ ──否──→ 反思循环 ──┘
         ↓是
【外层】报告生成
    ↓
最终临床报告（含安全警告）
```

---

## 🛠️ 技术栈与全链路架构

### 核心技术栈

#### 模型服务核心（Python层 - 本仓库主体）
- **Web框架**：FastAPI - 高性能异步Web框架
- **Agent架构**：LangChain + LangGraph - 智能体编排和状态管理
- **大语言模型**：Qwen-Max（通义千问）- 阿里云大模型服务
- **向量检索**：ChromaDB - 本地向量数据库
- **关键词检索**：BM25 - 经典关键词检索算法
- **重排模型**：阿里gte-rerank - 深度语境重排
- **配置管理**：YAML配置文件 + 动态配置加载器

#### 后端服务层（可选集成）
- **语言**：Java 17
- **框架**：Spring Boot 3 + Spring WebFlux（响应式框架）
- **数据库**：MySQL 8.0
- **缓存**：Redis + Redisson（分布式控制）
- **认证**：JWT鉴权

#### 前端展示层（可选集成）
- **框架**：Vue 3（Composition API）
- **构建工具**：Vite
- **状态管理**：Pinia
- **特性**：流式数据的实时图文响应渲染

### 数据流向管道

```
用户提问 
  → Java鉴权与请求隔离 
  → WebClient异步调用 
  → FastAPI接收请求 
  → Python Agent流式产出(yield) 
  → asyncio.Queue异步队列 
  → Java(Flux持续推送) 
  → Vue(ReadableStream实时渲染)
```

---

## 📂 项目目录结构

```text
D:\pycharmProject\neuro-multi-agent
├── app/                          # 应用主目录
│   ├── agents/                   # 智能体核心模块
│   │   ├── core/                 # 核心定义（状态模式、异常处理）
│   │   │   ├── schema.py         # ClinicalState状态定义
│   │   │   ├── decorators.py     # 装饰器工具
│   │   │   ├── exceptions.py     # 异常定义
│   │   │   └── result.py         # 结果封装
│   │   ├── orchestrators/        # 智能体编排层
│   │   │   ├── clinical_graph.py # 临床推理图构建
│   │   │   ├── qwen_agent.py     # Qwen智能体实现
│   │   │   └── nodes/            # 推理节点
│   │   │       ├── intent_node.py    # 意图识别节点
│   │   │       ├── analysis_node.py  # 病例分析节点
│   │   │       ├── retrieve_node.py  # 证据检索节点
│   │   │       ├── reason_node.py    # 多专家推理节点
│   │   │       ├── validate_node.py  # 校验反思节点
│   │   │       └── report_node.py    # 报告生成节点
│   │   ├── pipelines/            # 处理管道
│   │   │   └── rag_pipeline.py   # RAG检索管道
│   │   ├── services/             # 业务服务
│   │   │   ├── query_service.py      # 查询服务
│   │   │   ├── retrieval_service.py  # 检索服务
│   │   │   └── synthesis_service.py  # 综合服务
│   │   ├── utils/                # 工具函数
│   │   │   ├── json_parser.py    # JSON解析工具
│   │   │   ├── llm_helper.py     # LLM辅助工具
│   │   │   ├── retry.py          # 重试机制
│   │   │   └── text_utils.py     # 文本处理工具
│   │   ├── bailian/              # 百炼模型集成
│   │   │   └── health_risk_analyzer.py  # 健康风险分析
│   │   ├── constants.py          # 常量定义
│   │   └── assistant.py          # 助手主类
│   ├── config/                   # 配置中心
│   │   ├── config_loader.py      # 配置加载器
│   │   ├── expert_config.yaml    # 专家角色配置
│   │   ├── rules_config.yaml     # 校验规则配置
│   │   ├── limits_config.yaml    # 参数限制配置
│   │   ├── prompts.yaml          # 提示词模板
│   │   └── report_templates.yaml # 报告模板
│   ├── rag/                      # RAG模块
│   │   ├── data_loader.py        # 数据加载器
│   │   ├── qa_generator.py       # QA生成器
│   │   ├── retrieve.py           # 检索引擎
│   │   └── retrievers.py         # 检索器实现
│   ├── services/                 # 外部服务
│   │   ├── pubmed_service.py     # PubMed文献搜索
│   │   └── vision_service.py     # 视觉识别服务
│   ├── utils/                    # 通用工具
│   │   ├── context_summary.py    # 上下文摘要
│   │   ├── download_models.py    # 模型下载
│   │   ├── error_codes.py        # 错误码定义
│   │   ├── naming_model.py       # 命名模型
│   │   └── token_aggregator.py   # Token聚合器
│   ├── evaluation/               # 评估模块
│   └── main.py                   # 🚀应用入口
├── data/                         # 数据目录
│   └── documents/                # 医疗文档（PDF指南等）
├── tests/                        # 测试模块
│   ├── test_api_client.py        # API客户端测试
│   ├── test_rag.py               # RAG功能测试
│   ├── test_migration.py         # 迁移测试
│   ├── test_new_architecture.py  # 新架构测试
│   ├── test_simple_migration.py  # 简单迁移测试
│   └── run_search.py             # 搜索测试
├── .env                          # 环境变量配置
├── .env.example                  # 环境变量示例
├── .gitignore                    # Git忽略配置
├── requirements.txt              # Python依赖清单
├── start.bat                     # Windows启动脚本
├── start.sh                      # Linux/Mac启动脚本
├── README.md                     # 📖项目说明文档
├── ARCHITECTURE.md               # 🏗️架构设计文档
└── OPTIMIZATION_SUMMARY.md       # 📊优化总结文档
```

---

## 🔄 系统核心链路流程

### 1. 用户提问触发
前台发来病理文本（甚至带图片）。鉴权通过（JWT校验）后建立SSE长连接。

### 2. 意图识别与智能分发
系统评估请求的`question`和`all_info`（病史上下文）。利用意图识别模型判断问题类型：
- **无关问题** → 拒绝处理
- **知识问答** → 直接回答
- **临床问诊** → 进入完整推理流程

### 3. 病例结构化分析
对输入的病例信息进行结构化分析，提取关键临床要素：
- 患者基本信息（年龄、性别等）
- 主诉和现病史
- 既往史和用药史
- 体格检查和辅助检查

### 4. 大混合双重检索（Hybrid RAG）
调用`rag/retrievers.py`中的检索引擎从本地检索与当前体征关联度高的内容：
- **向量检索**：基于语义相似度检索相关段落
- **关键词检索**：基于BM25算法检索精确匹配内容
- **混合重排**：结合两种检索结果，使用gte-rerank进行深度重排

### 5. 多专家协同推理
医学助理Agent汇集所有精准片段作为`background_info`给到Qwen-Max进行多步严谨推演：
- **全科医生**：从整体角度分析病情
- **神经专科医生**：从专业角度提供诊疗建议
- **临床药师**：从用药安全角度审查方案
- **意见综合**：整合各专家意见，形成初步方案

### 6. 双重校验与反思
对生成的初步方案进行双重校验：
- **规则引擎检查**：快速匹配禁忌症规则，硬规则拦截
- **LLM反思校验**：深层次医学逻辑审查，检查指南违反和常识错误
- **反思循环**：校验失败时自动重新推理，最多支持可配置次数的反思

### 7. 报告生成与输出
生成最终的临床报告，包含：
- 诊断结果和依据
- 治疗建议和方案
- 安全警告和注意事项
- 证据来源和参考文献

### 8. 上下文总结更新
流式推送完结论后，启动后台模型总结这次对话重点更新回`all_info`，为用户的多轮就诊做足铺垫。

---

## 🚀 快速接入

### 1. 环境准备与依赖安装

建议通过Anaconda新建虚拟环境屏蔽本机干扰。

```bash
# 创建虚拟环境
conda create -n neuro-model python=3.10
conda activate neuro-model

# 安装依赖
pip install -r requirements.txt
```

### 2. 本地秘钥`.env`配置

在根目录新建或修改`.env`文件，加入阿里云百炼API的密钥和JWT认证种子：

```env
# 阿里云百炼API密钥
DASHSCOPE_API_KEY="sk-您自己在阿里云百炼平台申请的秘钥"

# JWT认证密钥
SECRET_KEY="您自定义防止用户越权访问后端的随机字符串"

# 其他可选配置
# LOG_LEVEL="INFO"
# MAX_RETRIES=3
# TIMEOUT=30
```

### 3. 数据知识库建设（RAG核心底座）

把医疗领域的临床指南等PDF文件统一放入`data/documents/`文件夹。

**启动时的大模型打底入库过程**：

当您第一次启动系统或更换了新一批文档时，系统会执行以下操作：

1. **自动递归分块（Recursive Chunking）**
   - 采用512字长配128字重叠的规则
   - 跨层级用段落、句号作为自然分割符
   - 将几十个PDF`split_documents`切成上千条长块

2. **大模型QA衍生（AI Batch QA Generation）**
   - 如果启用了`"enable_qa_generation": True`
   - 系统会将文本块每10条打包发送给Qwen-Turbo
   - 用模型独有的归纳能力从里头"反向做题"
   - 提取出几百条`Q: ...? A: ...`并打上原文页码标签

3. **混合双索引编织（Dual-Indexing）**
   - 将"原生块"+"造出的QA"进行向量化放进ChromaDB
   - 在内存挂载高频词组的BM25索引
   - 构建庞大的知识底座

### 4. 配置文件说明

系统支持通过YAML配置文件灵活调整各项参数：

#### `config/expert_config.yaml` - 专家角色配置
```yaml
experts:
  general_practitioner:
    name: "全科医生"
    role: "从整体角度分析病情"
    system_prompt: "..."
    enabled: true
  
  neurologist:
    name: "神经专科医生"
    role: "从专业角度提供诊疗建议"
    system_prompt: "..."
    enabled: true
  
  clinical_pharmacist:
    name: "临床药师"
    role: "从用药安全角度审查方案"
    system_prompt: "..."
    enabled: true

synthesis:
  method: "weighted_voting"
  weights:
    general_practitioner: 0.3
    neurologist: 0.5
    clinical_pharmacist: 0.2
```

#### `config/rules_config.yaml` - 校验规则配置
```yaml
validation:
  max_reflection_count: 3
  enable_rule_engine: true
  enable_llm_reflection: true

contraindications:
  thrombolysis:
    - condition: "active_bleeding"
      severity: "critical"
      message: "活动性出血是溶栓治疗的绝对禁忌症"
  
  anticoagulation:
    - condition: "severe_hypertension"
      severity: "high"
      message: "严重高血压是抗凝治疗的相对禁忌症"
```

#### `config/limits_config.yaml` - 参数限制配置
```yaml
limits:
  max_sub_questions: 5
  max_evidence_chars: 2000
  max_proposal_chars: 1500
  max_critique_chars: 1000

keywords:
  diagnosis:
    - "诊断"
    - "确诊"
    - "判断"
  
  treatment:
    - "治疗"
    - "方案"
    - "用药"
```

### 5. 启动与测试

#### 启动服务
```bash
# Windows
start.bat

# Linux/Mac
bash start.sh

# 或直接运行
python app/main.py
```

服务会默认监听`0.0.0.0:8000`

#### 测试功能

**测试核心多智能体和临床指南RAG推测的流式问诊能力**
```bash
python tests/test_api_client.py
```

**测试本地向量数据库召回水平**
```bash
python tests/test_rag.py
```

**测试新架构功能**
```bash
python tests/test_new_architecture.py
```

---

## 📝 核心 API

### 1. 主要推理流：`/model/get_result`

**特性**：结合多智能体与RAG对重症、急诊情况分析

**协议**：SSE（Server-Sent Events）

**请求参数**：
```json
{
  "question": "患者男，65岁，突发左侧肢体无力3小时，既往有高血压病史。",
  "all_info": "既往史：高血压10年，糖尿病5年",
  "token": "your-jwt-token",
  "report_mode": "emergency",
  "show_thinking": true
}
```

**响应格式**：流式输出，包含思考过程和最终报告

### 2. 独立风险归纳：`/ai/analyze`

**特性**：不需要进行检索，专注于独立分析大段病历评估急用风险

**请求参数**：
```json
{
  "case_text": "患者男，65岁，突发左侧肢体无力3小时...",
  "token": "your-jwt-token"
}
```

**响应格式**：
```json
{
  "riskLevel": "high",
  "suggestion": "建议立即进行影像学检查",
  "analysisDetails": "..."
}
```

### 3. 外部补充抓取：`/model/pubmed/search`

**特性**：连接国家生化信息中心的API直接根据症状抓取相关最新外文文章列表

**请求参数**：
```json
{
  "query": "acute ischemic stroke thrombolysis",
  "max_results": 10
}
```

**响应格式**：包含文献标题、作者、摘要、链接等信息

---

## 🎯 使用示例

### 示例1：急性缺血性卒中诊疗

**输入**：
```json
{
  "question": "患者男，65岁，突发左侧肢体无力3小时，NIHSS评分12分，既往有高血压病史。CT排除脑出血。如何处理？",
  "report_mode": "emergency",
  "show_thinking": true
}
```

**输出**：
- 诊断结果：急性缺血性卒中
- 治疗建议：评估溶栓适应症，考虑静脉溶栓治疗
- 安全警告：注意溶栓禁忌症，监测出血风险
- 证据来源：引用相关指南和文献

### 示例2：术后并发症处理

**输入**：
```json
{
  "question": "患者术后次日复查头颅CT显示梗死灶内出现点状高密度影，无占位效应，患者症状稳定。这最可能是什么？是否需要特殊处理？",
  "report_mode": "routine",
  "show_thinking": true
}
```

**输出**：
- 诊断结果：出血转化（症状性）
- 处理建议：继续监测，调整抗凝方案
- 安全警告：密切观察神经功能变化
- 证据来源：引用出血转化诊治共识

---

## 🔧 配置化与扩展

### 添加新的专家角色

编辑`config/expert_config.yaml`：

```yaml
experts:
  radiologist:
    name: "影像科医生"
    role: "从影像学角度提供诊断建议"
    system_prompt: "你是影像科专家，专注于CT/MRI影像分析..."
    enabled: true
```

### 添加新的禁忌症规则

编辑`config/rules_config.yaml`：

```yaml
contraindications:
  dual_antiplatelet:
    - condition: "bleeding_risk_high"
      severity: "high"
      message: "高出血风险是双抗治疗的相对禁忌症"
```

### 调整反思次数

编辑`config/rules_config.yaml`：

```yaml
validation:
  max_reflection_count: 5  # 增加反思次数
```

---

## 📊 性能优化建议

### 1. 向量数据库优化
- 定期清理无效向量
- 调整chunk大小和重叠度
- 使用更高效的embedding模型

### 2. 检索优化
- 调整向量检索和关键词检索的权重
- 优化重排模型的阈值
- 实现检索结果缓存

### 3. 推理优化
- 调整专家数量和权重
- 优化prompt模板
- 实现推理结果缓存

---

## 🛡️ 安全与合规

### 安全机制
- **多层防护**：外层流程控制 + 后层双重校验
- **可追溯性**：每一步都有明确的状态记录
- **人工干预**：关键节点支持人工审核
- **合规友好**：符合医疗行业监管要求

### 数据隐私
- 所有敏感数据加密存储
- 支持数据脱敏处理
- 符合GDPR和HIPAA要求

### 责任声明
本系统仅供临床辅助决策参考，不能替代专业医生的判断。最终诊疗决策应由专业医生根据患者具体情况做出。

---

## 🤝 贡献指南

欢迎贡献代码、报告问题或提出建议！

### 贡献流程
1. Fork本仓库
2. 创建特性分支（`git checkout -b feature/AmazingFeature`）
3. 提交更改（`git commit -m 'Add some AmazingFeature'`）
4. 推送到分支（`git push origin feature/AmazingFeature`）
5. 开启Pull Request

### 代码规范
- 遵循PEP 8编码规范
- 添加必要的注释和文档
- 编写单元测试
- 确保代码通过lint检查

---

## 📄 许可证

本项目采用MIT许可证 - 详见LICENSE文件

---

## 📞 联系方式

- 项目主页：[GitHub Repository]
- 问题反馈：[Issues]
- 邮件联系：[your-email@example.com]

---

## 🙏 致谢

感谢以下开源项目和工具的支持：
- LangChain & LangGraph
- FastAPI
- ChromaDB
- Qwen（通义千问）
- 阿里云百炼平台

---

## 📚 参考文档

- [ARCHITECTURE.md](ARCHITECTURE.md) - 详细架构设计文档
- [OPTIMIZATION_SUMMARY.md](OPTIMIZATION_SUMMARY.md) - 优化总结和医疗安全三角架构
- [LangGraph文档](https://langchain-ai.github.io/langgraph/)
- [FastAPI文档](https://fastapi.tiangolo.com/)

---

**⚠️ 免责声明**：本系统仅供临床辅助决策参考，不能替代专业医生的判断。最终诊疗决策应由专业医生根据患者具体情况做出。