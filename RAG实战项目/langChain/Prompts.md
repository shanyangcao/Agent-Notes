### ![img](https://cdn.nlark.com/yuque/0/2026/png/56763514/1773881746366-e18a7ff9-dbb2-40b7-84ad-c42737f4b8a6.png)

### 一、PromptTemplate

**通用提示词模板**

**1. 是什么**

最基础的**变量替换模板**，把固定话术和可变参数组合起来，通过 `{变量名}` 占位符动态填充内容。

**2. 核心结构**

```
from langchain.prompts import PromptTemplate

prompt = PromptTemplate(
    input_variables=["topic", "difficulty"],  # 定义需要传入的变量
    template="请解释{topic}的概念，难度设定为{difficulty}级别"  # 模板文本，包含占位符
)

# 传入参数生成完整提示词
full_prompt = prompt.format(topic="Python装饰器", difficulty="入门")
# 输出："请解释Python装饰器的概念，难度设定为入门级别"
```

**3. 适用场景**

- 简单的单轮提问、固定格式输出
- 批量生成相似提示词（比如批量生成不同主题的解释文案）
- 不需要示例、只需要变量替换的场景

### 二、FewShotPromptTemplate

**少样本提示词模板**

**1. 是什么**

在通用模板基础上，**加入了 “示例（Few-Shot Examples）”**，让模型先看几个 “问题 - 答案” 范例，再生成更规范、更符合预期的输出，属于**小样本学习**在提示词里的应用。

**2. 核心结构**

```
from langchain.prompts import FewShotPromptTemplate, PromptTemplate

# 1. 准备示例（问题-答案对）
examples = [
    {"question": "1+1等于几", "answer": "2"},
    {"question": "3+5等于几", "answer": "8"}
]

# 2. 定义示例的模板
example_prompt = PromptTemplate(
    input_variables=["question", "answer"],
    template="问题：{question}\n答案：{answer}"
)

# 3. 构建少样本模板
few_shot_prompt = FewShotPromptTemplate(
    examples=examples,  # 传入示例列表
    example_prompt=example_prompt,  # 示例的格式模板
    prefix="请按照以下示例回答数学问题：",  # 开头引导语
    suffix="问题：{input}\n答案：",  # 结尾+用户新问题占位符
    input_variables=["input"]  # 最终需要传入的变量
)

# 生成完整提示词
full_prompt = few_shot_prompt.format(input="7+9等于几")
```

生成后的完整提示词会是：

```
请按照以下示例回答数学问题：
问题：1+1等于几
答案：2
问题：3+5等于几
答案：8
问题：7+9等于几
答案：
```

**3. 适用场景**

- 要求输出格式严格统一（比如 JSON、特定结构的代码）
- 模型需要学习特定回答风格 / 逻辑
- 复杂任务下提升模型准确率（比如文本分类、信息抽取）

### 三、核心对比

| **维度**       | **PromptTemplate（通用模板）** | **FewShotPromptTemplate（少样本模板）**     |
| -------------- | ------------------------------ | ------------------------------------------- |
| **核心能力**   | 变量替换，生成固定格式提示词   | 变量替换 + 示例注入，引导模型按范例输出     |
| **结构复杂度** | 低，仅需定义变量和模板文本     | 高，需要额外准备示例、示例模板、前缀 / 后缀 |
| **输出可控性** | 一般，依赖模型本身理解能力     | 高，通过示例规范输出格式和逻辑              |
| **Token 消耗** | 较少                           | 较多（示例会占用额外 Token）                |
| **典型场景**   | 简单提问、批量生成提示词       | 格式要求高、复杂任务、小样本学习场景        |