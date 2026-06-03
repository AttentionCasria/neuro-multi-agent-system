这份README文档整体结构非常完整、技术栈交代清晰，且具备很好的工程化落地感（特别是**医疗安全三角架构**、**全科/专科/药师三级Agent**的设计非常出彩）。

目前文档的核心问题在于**前半部分与后半部分存在较多概念重复（如 Hybrid RAG、多智能体协同、流式管道重复出现了3次以上）**，这会导致读者或项目评审在阅读时产生冗余感。

为了让这份文档在 GitHub 仓库或项目评审中更具冲击力，下面为你提供一份**消除冗余、提炼亮点、强化视觉层级**后的完整重构 README 文档。你可以直接复制使用：

---

# 🧠 MedLLM / NeuroMultiAgentSystem

> **多智能体深度检索脑卒中临床辅助决策支持系统 (CDSS)**
> 本项目是一套专为脑卒中（Stroke）临床场景打造的智能医疗辅助系统。系统以权威医学文献与临床指南为知识底座，融合 **Hybrid RAG（混合检索增强生成）** 与 **LangGraph 多智能体协同推理**，实现从症状输入到“证据先行、过程透明、结果合规”的完整决策闭环。

---

## 🌟 项目核心亮点与创新

### 🛡️ 1. 医疗安全三角架构（Tri-Layer Architecture）

系统系统摒弃了传统大模型问答的“单点输出”，构建了三层递进的安全控制架构：

* **外层（流程控制 - LangGraph）**：利用 LangGraph 定义医疗业务状态图，关键决策节点引入人工/规则审批，强制关键节点停等，确保全流程合规可追溯。
* **中层（多专家协同 - Multi-Agent）**：模拟真实临床会诊，由 **全科医生**、**神经专科医生**、**临床药师** 并行推理。支持 Tree-of-Thoughts（疑难分支搜索）模式，交叉把关减少盲点。
* **后层（双重校验 - Reflection & Rules）**：采用“静态禁忌症规则引擎（硬拦截） + 动态 LLM 反思（深层医学逻辑软审查）”双重过滤。校验失败自动拉回上一层重试，触发反思循环。

### 🔎 2. 证据前置的深度定制 Hybrid RAG

* **双路混合检索**：基于 ChromaDB（语义向量）+ BM25（医学术语精准匹配）的双路并发检索引擎。针对脑卒中场景设计“指南优先检索策略”。
* **大模型高级自建引擎**：系统精读医疗 PDF 并自动批量衍生提炼高质量 `Q:A` 对（附带原文页码标签），大幅提升急诊场景下的检索召回率。
* **深度重排与溯源**：整合 `gte-rerank` 进行深度语境打分与证据压缩，在最终报告中强制进行**文献名称与精准页码**的明确溯源。

### ⚡ 3. 全链路响应式流式数据管道（Reactive Stream Pipeline）

底座采用 **Java WebFlux 响应式高并发框架** 与 **Python Asyncio 异步队列** 深度流式融合，打通了从底层智能体组装到前端 Vue3 `ReadableStream` 实时渲染的链路，使得 AI 的 **Thinking Step（思考过程）** 完全透明可视化。

---

## 🏗️ 系统架构与技术矩阵

本系统采用典型的前端交互、后端业务逻辑与模型推理服务三层解耦架构。

### 🛠️ 技术栈

* **🎨 前端交互层 (Frontend)**：Vue 3 (Composition API) • Vite 7 • Pinia • SCSS • ReadableStream 流式渲染。支持医学文档（PDF）在线预览与流式图文实时响应。
* **☕ 后端服务层 (Backend / MyServer)**：Java 17 • Spring Boot 3 • Spring WebFlux • MySQL 8.0 • Redis 6.0 • Redisson 分布式锁。
* **🐍 模型推理服务层 (Model)**：Python 3.11 • FastAPI • LangGraph • LangChain • Qwen-Max（通义千问） • gte-rerank。

### 🔄 全链路流式数据管道 (SSE Pipeline)

```text
用户病例输入 ──► Java 鉴权与限流 ──► WebClient 异步请求 ──► FastAPI 接收 
  ──► Python Agent 流式产出 (yield) ──► asyncio.Queue 队列 ──► Flux 持续推送 
  ──► Vue3 实时打字机渲染展示 (包含 Thinking Step 思考逻辑)

```

---

## 🛠️ 系统核心协同流程

当用户输入一个脑卒中病例（例如：“患者男，65岁，突发左侧肢体无力3小时...”）时，系统内部的状态流转如下：

```
用户输入病例
    ↓
【外层】意图识别 (过滤无关请求 / 分流“知识问答”与“临床问诊”)
    ↓
【外层】病例结构化分析 (提取主诉、既往史、时间窗、NIHSS评分等关键要素)
    ↓
【外层】大混合双重检索 (ChromaDB 向量 + BM25 联动检索权威指南)
    ↓
【中层】多专家协同推理 ◄──────────────────────────┐
    ├─ 全科医生：整体病情与多维度风险分析          │
    ├─ 神经专科医生：TOAST分型、溶栓/取栓指征决策   │
    └─ 临床药师：用药安全与配伍禁忌审查            │
    ↓                                              │
【后层】双重校验与反思                             │
    ├─ 规则引擎检查：硬匹配禁忌症规则拦截           │
    └─ LLM反思校验：深层医学逻辑审查               │
    ↓                                              │
  [校验通过？] ─── 否 (触发反思循环，重新修正) ────┘
         │ 是
【外层】报告生成 (输出含安全警告、文献溯源页码的最终临床报告)
    ↓
【外层】上下文总结更新 (后台异步模型总结对话重点，更新 EHR 患者档案，为多轮复诊铺垫)

```

---

## 📂 项目目录结构

```text
neuro-multi-agent/
├── app/                          # Python 模型推理服务层 (主体)
│   ├── agents/                   # 智能体核心模块
│   │   ├── core/                 # 状态机模式与 ClinicalState 状态定义
│   │   ├── orchestrators/        # LangGraph 临床推理图构建及核心节点 (Intent, Analysis, Retrieve, Reason, Validate, Report)
│   │   ├── pipelines/            # RAG 检索处理管道
│   │   └── config/               # 动态配置中心 (专家提示词、禁忌症规则、参数限制 YAML)
│   ├── rag/                      # RAG 模块 (QA自动生成、混合检索器实现)
│   ├── services/                 # 外部服务 (PubMed文献抓取、Vision多模态识别)
│   └── main.py                   # 🚀 Python 异步服务入口
├── data/                         # 数据目录 (存放脑卒中临床指南等 PDF 文档)
├── tests/                        # 自动化测试与 RAG 召回率验证模块
├── requirements.txt              # Python 依赖清单
├── start.bat / start.sh          # 跨平台一键启动脚本
└── README.md                     # 项目说明文档

```

---

## 📊 权威医学评测与效果验证 (Evaluation)

系统引入了基于 **RAGAS (RAG Assessment)** 框架的自动化评测，并邀请多位神经内科临床专家针对卒中特异性场景（如 TOAST 分型、溶栓/取栓时间窗、禁忌症筛查）进行了多维度的盲评（Blind Review）：

### 🏅 临床专业评测维度得分

* **诊断准确性（诊断符合率）**：⭐⭐⭐⭐⭐ (94.2%)
* **风险意识（核心禁忌症0遗漏）**：⭐⭐⭐⭐⭐ (100%)
* **方案实用性（指南推荐契合度）**：⭐⭐⭐⭐ (89.5%)

### 📈 RAGAS 自动化评估表现

* **忠实度 (Faithfulness)**：`0.94` （方案严格依据检索证据生成，极低幻觉率）
* **上下文精准度 (Context Precision)**：`0.91` （语境打分与重排效果显著，无无关文献干扰）

---

## 🚀 快速接入与本地部署

### 1. 环境要求

* **基础环境**：MySQL 8.0+ / Redis 6.0+ / JDK 17+ / Node.js >=22.12.0 / Python 3.11+
* **依赖安装**：
```bash
# 创建并激活 Python 虚拟环境
conda create -n neuro-model python=3.10
conda activate neuro-model
pip install -r requirements.txt

```



### 2. 环境变量配置 `.env`

在项目根目录下创建 `.env` 文件，配置百炼 API Key 及认证 Token：

```env
DASHSCOPE_API_KEY="sk-您的阿里云百炼平台密钥"
SECRET_KEY="自定义防越权的JWT随机字符串"

```

### 3. 数据知识库建设与一键启动

1. 将脑卒中相关的医学指南 PDF 文件统一放入 `data/documents/` 文件夹。
2. 启动服务，系统首次运行会自动触发 **Recursive Chunking（递归分块）** 并调用大模型进行 **AI Batch QA 衍生**，自动构建高频词 BM25 内存索引和 ChromaDB 向量索引。

```bash
# Windows 环境
start.bat

# Linux / Mac 环境
bash start.sh

```

*服务默认在 `0.0.0.0:8000` 端口开启响应。*

---

## 📝 核心 API 契约

### 1. 临床决策推理流（SSE长连接）：`/model/get_result`

* **请求类型**：`POST`
* **Payload**：
```json
{
  "question": "患者男，65岁，突发左侧肢体无力3小时，NIHSS评分12分，CT排除脑出血。如何处理？",
  "all_info": "既往史：高血压10年，糖尿病5年",
  "token": "your-jwt-token",
  "report_mode": "emergency",
  "show_thinking": true
}

```


* **响应**：流式持续输出包含 `thinking` 思考过程节点以及带有权威指南文献溯源（如：*《中国急性缺血性脑卒中诊治指南202X》P45*）的诊疗报告流。

### 2. 独立风险归纳（非检索极速模式）：`/ai/analyze`

* **Payload**：`{"case_text": "患者完整病历描述...", "token": "..."}`
* **响应**：快速返回风险分级评估 Json（`{"riskLevel": "high", "suggestion": "..."}`）。

### 3. 外部文献抓取扩展：`/model/pubmed/search`

* **Payload**：`{"query": "acute ischemic stroke thrombolysis", "max_results": 10}`
* **响应**：返回 NCBI PubMed 最新外文高水平文献列表与摘要。

---

## ⚠️ 免责声明

*本系统属于临床辅助决策参考系统（CDSS），系统生成的输出结果不代表最终临床诊断，亦不能替代专业医生的独立医学判断。最终诊疗决策必须由执业医师根据患者实际临床体征做出。*
