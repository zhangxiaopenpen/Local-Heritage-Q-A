# 基于非遗知识库的本地化问答大语言模型系统

面向非物质文化遗产场景的本地知识库问答项目，包含数据采集、知识向量检索、本地大模型推理、后端接口和前端对话页面。

## 项目简介

本项目围绕“非遗知识问答”搭建了一套本地化问答流程：

- 通过爬虫采集非遗项目数据
- 将文本资料切分并向量化
- 使用 FAISS 构建本地向量检索库
- 使用本地部署的大语言模型生成回答
- 通过 FastAPI 暴露问答接口
- 通过 Vue 前端提供图形化聊天界面

项目整体目标是：让用户可以围绕非遗知识进行自然语言提问，并获得结合本地知识库内容的回答，而不是只依赖通用大模型的参数记忆。

## 已实现功能

### 1. 非遗数据采集

位于 `非遗数据爬取/`：

- 调用中国非遗相关公开接口抓取项目数据
- 提取标题、类型、省份、简介等字段
- 保存为 `output_data.xlsx`
- 可作为后续知识库文本整理的数据来源

### 2. 本地知识库构建

位于 `后端/main.py`：

- 读取 `后端/article/` 下的文本资料
- 使用 `RecursiveCharacterTextSplitter` 对文本进行切分
- 使用中文向量模型生成 embedding
- 使用 FAISS 构建并保存向量索引
- 支持相似文本检索，为问答提供上下文

### 3. 本地大模型问答

位于 `后端/main.py`、`后端/web_demo.py`、`后端/cli_demo.py`：

- 封装了本地 Baichuan 模型调用逻辑（底层语言模型替换为Qwen，ChatGLM均可以，但要做逻辑改动，本人已验证过）
- 支持命令行对话
- 支持 Streamlit Web 演示
- 支持结合知识检索结果进行问答生成

### 4. 后端问答接口

位于 `后端/main.py`：

- 基于 FastAPI 提供 `/chat` 接口
- 接收前端问题文本
- 调用“检索 + 大模型生成”流程
- 返回问答结果 JSON

### 5. 前端聊天页面

位于 `前端代码/pythonproject1/`：

- 基于 Vue 3 + Vite 构建
- 提供聊天消息展示
- 通过 Axios 请求后端接口
- 支持回车发送
- 集成浏览器语音识别输入（依赖 Web Speech API）

### 6. 微调脚本

位于 `后端/fine-tune/`：

- 提供基于 Baichuan2 的监督微调脚本
- 支持可选 LoRA 配置
- 可用于后续领域化模型增强

## 系统实现逻辑

项目的核心逻辑可以概括为下面这条链路：

```text
非遗数据采集
    ↓
文本清洗 / 整理为知识文档
    ↓
文本切分
    ↓
向量化（Embedding）
    ↓
FAISS 向量检索
    ↓
用户提问
    ↓
召回相关知识片段
    ↓
本地大模型生成回答
    ↓
FastAPI 返回结果
    ↓
Vue 前端展示回答
```

## 关键工作流程说明

### 数据侧逻辑

1. 使用 `非遗数据爬取/main.py` 抓取非遗条目数据
2. 将数据整理为知识文本，放入 `后端/article/`
3. 后端启动时读取文本并切分
4. 通过 embedding 模型生成向量表示
5. 构建本地 FAISS 索引库

### 问答侧逻辑

1. 前端输入问题
2. 请求发送到 `POST /chat`
3. 后端基于问题从向量库检索相关文本
4. 检索结果交给本地大模型
5. 模型生成最终回答
6. 后端返回 `{ "message": "..." }`
7. 前端展示回答内容

## 项目结构

```text
Local-Q-A/
├─ README.md
├─ 前端代码/
│  └─ pythonproject1/
│     ├─ src/
│     │  ├─ App.vue
│     │  ├─ main.js
│     │  └─ components/
│     │     ├─ ChatBox.vue
│     │     └─ ChatInterface.vue
│     ├─ package.json
│     └─ vite.config.js
├─ 后端/
│  ├─ main.py
│  ├─ web_demo.py
│  ├─ cli_demo.py
│  ├─ OpenAI_api.py
│  ├─ requirements.txt
│  ├─ article/
│  ├─ Faiss_file/
│  ├─ VectorStore/
│  ├─ embedding/
│  └─ fine-tune/
└─ 非遗数据爬取/
   ├─ main.py
   └─ output_data.xlsx
```

## 技术栈

### 前端

- Vue 3
- Vite
- Axios
- Web Speech API

### 后端

- Python
- FastAPI
- Uvicorn
- LangChain
- FAISS
- HuggingFace / Transformers
- SentenceTransformers
- Streamlit

### 模型与检索

- Baichuan 系列本地大模型
- text2vec 中文向量模型
- FAISS 本地向量索引

## 接口说明

### `POST /chat`

请求示例：

```json
{
  "msg": "苗族古歌分布在哪些地方？"
}
```

响应示例：

```json
{
  "message": "这里是模型结合知识库后的回答内容"
}
```
