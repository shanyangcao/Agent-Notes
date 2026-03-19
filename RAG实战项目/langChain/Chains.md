![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1773924350950-3c9a7fa8-2caa-4b76-93e4-acd9358dd1f1.png)

LangChain 的链（Chain）本质是 `Runnable` 接口组件的流水线，**只要实现** `**Runnable**` **接口（或通过封装适配），任何组件 / 逻辑都能加入**，以下是完整分类总结（重点补充自定义函数）：

### 一、核心基础组件

| **组件类型** | **示例**                               | **作用**                                    |
| ------------ | -------------------------------------- | ------------------------------------------- |
| 提示词模板   | `PromptTemplate`/`ChatPromptTemplate`  | 把字典参数组装成完整提示词（`PromptValue`） |
| 大模型       | `ChatTongyi`/`ChatOllama`/`ChatOpenAI` | 接收提示词生成 `AIMessage` 回复             |
| 输出解析器   | `StrOutputParser`/`JsonOutputParser`   | 把 `AIMessage` 转成字符串 / JSON 等可用格式 |

### 二、进阶功能组件

#### 1. 数据处理 / 逻辑控制类

- `RunnableLambda`：封装**自定义函数**（核心！实现个性化逻辑）

```
# 自定义函数示例：清洗文本
def clean_text(text):
    return text.strip().replace("无效内容", "")
# 封装成Runnable并加入链
clean_runnable = RunnableLambda(clean_text)
chain = prompt | model | StrOutputParser() | clean_runnable
```

- `RunnableParallel`：并行执行多个组件（如同时调用两个模型 / 函数）
- `RunnableBranch`：条件分支（如根据问题类型选择不同模型 / 函数）
- `RunnablePassthrough`：透传输入参数（保留原始输入供后续使用）

#### 2. RAG 核心组件

- 嵌入模型：`DashScopeEmbeddings`/`OllamaEmbeddings`（文本转向量）
- 向量存储：`Chroma`/`FAISS`（向量数据库，存储 / 检索文档向量）
- 检索器：`VectorStoreRetriever`（从向量库中检索相关上下文）

#### 3. 工具调用类

- 自定义工具：`Tool`/`BaseTool`（如计算器、搜索引擎、本地 API 调用）
- 工具调用链：`create_tool_calling_chain`（让模型自动选择调用工具）

#### 4. 对话记忆类

- 记忆组件：`ConversationBufferMemory`/`ConversationSummaryMemory`（存储 / 总结对话历史）

### 三、核心规则

1. **核心前提**：所有加入链的组件（包括自定义函数）需是 `Runnable` 接口实现类，`RunnableLambda` 是自定义函数的 “适配层”，能把普通函数转为 `Runnable`。
2. **格式匹配**：前一个组件的输出必须能作为后一个组件的输入（如自定义函数输入需匹配解析器的输出类型）。
3. **嵌套组合**：链本身也是 `Runnable`，可作为子链嵌套到更大的链中（如 “检索子链 + 生成子链 + 自定义处理函数”）。