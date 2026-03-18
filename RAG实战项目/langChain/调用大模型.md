![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1773823478121-960219bf-54de-41f8-a63a-d2069ba94a75.png)

### 大语言模型（LLM Models）

#### 1. 阿里云千问

```
from langchain_community.llm.tongyi import Tongyi
```

- 来源：`langchain_community`（社区维护的第三方集成包）
- 作用：直接封装通义千问系列模型（如 qwen-max、qwen-turbo 等）
- 特点：开箱即用，无需额外兼容层，直接对接阿里云 DashScope 服务

```
# 1. 导入对应模型类
from langchain_community.llm.tongyi import Tongyi

# 2. 初始化模型实例（配置 API Key 和模型名称）
llm = Tongyi(
    model="qwen-max",  # 指定模型版本
    api_key="your-dashscope-api-key",  # 阿里云 API Key
    temperature=0.7    # 随机性参数（可选）
)

# 3. 调用模型（单轮提问）
response = llm.invoke("用Python输出1到10的数字")
print(response)
```

#### 2. Ollama 本地模型

```
from langchain_ollama import OllamaLLM
```

- 来源：`langchain_ollama`（专门为 Ollama 提供的官方集成包）
- 作用：对接本地 Ollama 服务，调用 Llama 3、Qwen、Phi 等开源模型
- 特点：适合本地部署、离线场景，模型完全运行在本地机器

```
from langchain_ollama import OllamaLLM

llm = OllamaLLM(
    model="qwen:7b"  # 指定本地 Ollama 中已下载的模型名
)
response = llm.invoke("用Python输出1到10的数字")
```

### 聊天模型（Chat Models）

- **定位**：专门处理对话交互场景，支持多轮对话上下文，输入输出为**消息格式**（system/user/assistant 角色）。
- **导入方式**：

#### 1. 阿里云千问

```
from langchain_community.chat_models.tongyi import ChatTongyi
```

#### 2. Ollama 本地模型

```
from langchain_ollama import ChatOllama
```

- **调用方法**：


- - `invoke()`：一次性返回完整对话结果
  - `stream()`：逐段流式输出对话内容
  - 支持批量调用（`batch()`）与异步调用（`ainvoke()`/`astream()`）

### 2. 文本嵌入模型（Embedding Models）

- **定位**：将文本转换为**数值向量**，用于语义检索、相似度计算、知识库构建等场景，不生成自然语言回复。
- **导入方式**：

#### 1. 阿里云千问

```
from langchain_community.embeddings import DashScopeEmbeddings
```

#### 2. Ollama 本地模型

```
from langchain_ollama import OllamaEmbeddings
```

- **调用方法**：


- - `embed_query()`：单次将查询文本转换为向量
  - `embed_documents()`：批量将文档文本转换为向量