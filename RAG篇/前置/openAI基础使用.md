## 一、基础

### 1. 核心前置准备

1. **安装依赖**：首先需要安装 OpenAI 官方 SDK（以 Python 为例）bash运行

```
pip install openai
```

1. **获取 API Key**：从 OpenAI 官网（或国内兼容平台如通义千问、智谱 AI）获取 API Key，这是调用接口的身份凭证。

### 2. 基础使用步骤

**1. 初始化客户端**

通过 API Key 和接口地址配置客户端，基础写法：

```
from openai import OpenAI

# 初始化客户端（OpenAI 官方）
client = OpenAI(
    api_key="你的API Key",  # 替换为真实API Key
    # 国内平台需指定兼容接口地址，如通义千问：base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)
```

**2. 构造对话消息（核心）**

OpenAI API 采用**对话消息列表**的格式传递上下文，每个消息包含 `role`（角色）和 `content`（内容）：

| **角色（role）** | **作用**                      | **示例**                                            |
| ---------------- | ----------------------------- | --------------------------------------------------- |
| system           | 设定 AI 行为准则（可选）      | {"role":"system","content":"你是 Python 专家"}      |
| user             | 用户的提问 / 指令             | {"role":"user","content":"写 1-10 的代码"}          |
| assistant        | AI 历史回复（多轮对话需携带） | {"role":"assistant","content":"print(range(1,11))"} |

基础示例（单轮对话）：

```
messages = [
    {"role": "system", "content": "你是简洁的编程助手，只输出代码不解释"},
    {"role": "user", "content": "用Python输出1到10的数字"}
]
```

**3. 发起请求并获取响应**

调用 `chat.completions.create` 方法（核心接口），指定模型和消息：

```
# 发起请求
response = client.chat.completions.create(
    model="gpt-3.5-turbo",  # 模型名称（如gpt-4、qwen3-max等）
    messages=messages,      # 构造好的对话消息
    temperature=0.7        # 随机性（0-2，越低越固定）
)

# 解析响应（提取AI回复核心内容）
answer = response.choices[0].message.content
print(answer)
```

**4. 核心响应解析**

返回的 `response` 对象包含关键信息：

- `response.choices[0].message.content`：AI 回复的核心文本内容
- `response.usage`：本次调用的 token 消耗（计费依据）

## 二、流式输出

### 1. 核心开启方式

在 `client.chat.completions.create()` 中添加参数：

```
stream=True  # 开启流式输出
```

- 开启后，API 不会等完整回复生成后再返回，而是**分块（chunk）实时推送**内容。
- 原本一次性返回的 `response` 变成了一个**可迭代对象**，需要循环读取每一块数据。

### 2. 流式响应处理逻辑

```
for chunk in response:
    print(
        chunk.choices[0].delta.content,
        end=" ",       # 每段内容之间用空格分隔（避免换行）
        flush=True     # 立刻刷新缓冲区，实现实时打印效果
    )
```

- **遍历 chunk**：循环从 `response` 中读取每一个数据块。
- **提取内容**：通过 `chunk.choices[0].delta.content` 获取当前块的文本内容（注意：不是 `message.content`）。
- **实时输出**：配合 `flush=True` 让内容立即显示在终端，模拟 “打字机” 效果。
- **注意**：原有的一次性解析代码 `print(response.choices[0].message.content)` 已被注释，流式模式下不再适用。

### 3. 流式输出的特点

- ✅ **实时性**：用户无需等待完整回复生成，可立即看到内容逐步输出。
- ✅ **低延迟**：适合对话类场景，提升交互体验。
- ✅ **内存友好**：避免加载超大完整回复，尤其适合长文本生成。
- ⚠️ **处理更复杂**：需要额外处理分块数据，无法直接获取完整回复。
- ⚠️ **接口差异**：响应结构从完整对象变为迭代器，需遍历拼接才能得到最终文本。

### 4. 完整流式流程对比

| **模式** | **调用方式**           | **响应类型**   | **内容提取方式**                                        |
| -------- | ---------------------- | -------------- | ------------------------------------------------------- |
| 普通模式 | `stream=False`（默认） | 完整响应对象   | `response.choices[0].message.content`                   |
| 流式模式 | `stream=True`          | 可迭代响应对象 | 循环遍历 `chunk`，拼接 `chunk.choices[0].delta.content` |

## 三、历史消息

OpenAI 模型本身不存储对话历史，每一次 API 调用都是**无状态**的。要实现多轮对话，需在 `messages` 列表中按「时间顺序」携带所有历史消息，模型会基于这个完整列表生成回复。

### 1. 历史消息的结构与使用

**1. 基础结构（角色顺序）**

`messages` 列表严格遵循「时间顺序」排列，核心角色组合：

```
# 初始化历史消息列表（包含系统指令）
messages = [
    {"role": "system", "content": "你是一个简洁的Python编程助手，只输出代码不解释"},  # 系统指令（可选，仅需初始化时加一次）
    {"role": "user", "content": "写一段代码输出1到10"},  # 第一轮用户提问
    {"role": "assistant", "content": "for i in range(1,11):\n    print(i)"},  # 第一轮AI回复（历史）
    {"role": "user", "content": "修改代码，让数字倒序输出"},  # 第二轮用户提问（基于上一轮的追问）
]
```

- **核心规则**：`user` 和 `assistant` 角色需**交替出现**，严格对应「用户问→AI 答→用户再问」的对话流程。
- **system 角色**：仅需在列表开头定义一次，无需重复添加（除非需要修改系统指令）。

**2. 多轮对话完整示例**

```
from openai import OpenAI

# 1. 初始化客户端
client = OpenAI(api_key="你的API Key")

# 2. 初始化历史消息（包含系统指令）
messages = [{"role": "system", "content": "你是Python编程助手，仅用代码回答"}]

# 3. 第一轮对话
user_msg1 = "写代码输出1到10"
messages.append({"role": "user", "content": user_msg1})  # 添加用户提问到历史
response1 = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=messages  # 携带当前所有历史
)
assistant_msg1 = response1.choices[0].message.content
messages.append({"role": "assistant", "content": assistant_msg1})  # 保存AI回复到历史
print("第一轮AI回复：", assistant_msg1)

# 4. 第二轮对话（基于上一轮追问）
user_msg2 = "修改代码，倒序输出"
messages.append({"role": "user", "content": user_msg2})  # 追加新提问
response2 = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=messages  # 携带两轮完整历史
)
assistant_msg2 = response2.choices[0].message.content
messages.append({"role": "assistant", "content": assistant_msg2})  # 保存新回复
print("第二轮AI回复：", assistant_msg2)
```

### 2. 关键注意事项

**1. 历史消息的管理**

- **追加而非覆盖**：每次对话后，需将「用户新提问」和「AI 新回复」**追加**到 `messages` 列表末尾，而非重新创建列表。
- **避免冗余**：无需重复添加 `system` 角色，仅需在初始化时定义一次。
- **长度控制**：模型有 token 上限（如 gpt-3.5-turbo 默认为 4096 token），历史消息过长会触发报错，需按需截断早期对话（保留核心上下文即可）。

**2. 常见错误与规避**

- ❌ 错误：仅传递当前提问，未携带历史消息 → 模型无法理解上下文（如追问 “修改代码” 时，模型不知道要修改哪段）。
- ❌ 错误：角色顺序混乱（如连续两个 `user` 角色）→ API 会返回格式错误。
- ✅ 正确：严格遵循「system → user → assistant → user → assistant...」的顺序。

**3. 流式输出中的历史消息**

流式输出仅改变「回复接收方式」，历史消息的管理逻辑与普通模式完全一致：

```
# 流式输出+历史消息示例
messages.append({"role": "user", "content": "再添加注释"})
response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=messages,
    stream=True  # 开启流式
)
# 接收流式回复并拼接
assistant_msg = ""
for chunk in response:
    content = chunk.choices[0].delta.content or ""
    assistant_msg += content
    print(content, end="", flush=True)
# 保存流式回复到历史
messages.append({"role": "assistant", "content": assistant_msg})
```