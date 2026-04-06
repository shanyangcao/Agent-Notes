### 一、三者关系

```
ChatModel   = 你调用AI的工具（相当于“连接器”）
Prompt      = 你给AI说的话（相当于“提问”）
ChatResponse= AI 给你的回答（相当于“回复”）
```

**流程固定死：**

```
你构造 Prompt → 交给 ChatModel → ChatModel 调用大模型 → 返回 ChatResponse
```

### 二、ChatModel 详细解释

#### 是什么？

**ChatModel 是 Spring AI 提供的统一接口**不管你用 豆包、通义、文心、OpenAI、Llama**一套代码通用，不用改逻辑！**

#### 作用？

- 帮你发送请求
- 帮你处理响应
- 帮你统一异常、配置、调用方式
- 你不用关心底层模型是谁

#### 代码 + 逐行解释

```
// 1. 自动注入 Spring AI 提供的聊天模型（自动根据配置匹配你用的AI）@Autowiredprivate ChatModel chatModel;// 底层真实实现可能是：// OpenAiChatModel / DouBaoChatModel / TongYiChatModel// 但你不用管！统一用 ChatModel 就行
```

#### 通俗类比

**ChatModel = 万能翻译官**你说中文，它自动帮你转成：

- 豆包能懂的话
- OpenAI 能懂的话
- 通义能懂的话

### 三、Prompt 详细解释

#### 是什么？

**Prompt = 你给大模型的指令 / 问题 / 对话内容**

在 Spring AI 里，它是一个**对象**，用来封装：

- 系统提示词
- 用户问题
- 历史对话

#### 作用？

告诉 AI：

- 你是谁
- 要做什么
- 怎么回答

#### 代码 + 逐行解释

```
// 1. 创建一个简单 Prompt（只传用户问题）Prompt prompt = new Prompt("帮我解释 Kafka 增量聚合是什么");// 2. 更高级的 Prompt（带角色设定）Prompt prompt = new Prompt(List.of(// 系统角色：设定AI身份new SystemMessage("你是一个Java技术专家，回答要通俗简洁"),// 用户问题new UserMessage("SpringAI 的 ChatModel 是什么？")));
```

#### 通俗类比

**Prompt = 你对 AI 说的话**没有 Prompt，AI 不知道你要干嘛。

### 四、ChatResponse 详细解释

#### 是什么？

**ChatResponse = AI 返回给你的结构化结果**

它不是一段字符串，而是一个对象，里面包含：

- 回答内容
- 用了多少 token
- 模型信息
- 结束原因
- 候选回答

#### 代码 + 逐行解释

```
// 1. 把 Prompt 传给 ChatModel，得到响应ChatResponse response = chatModel.call(prompt);// 2. 从 response 里取出 AI 回答的文字String answer = response.getResult().getOutput().getContent();// 3. 输出结果System.out.println(answer);
```

#### 通俗类比

**ChatResponse = AI 递回来的 “答案包裹”**你要自己拆开（getContent）才能拿到文字。

### 五、三者完整运行代码

#### 引入依赖（pom.xml）

```
<dependency><groupId>org.springframework.ai</groupId><artifactId>spring-ai-openai-spring-boot-starter</artifactId></dependency>
```

#### 配置文件（application.yml）

```
spring:ai:openai:api-key: 你的key
      base-url: 你的地址
```

#### 真实业务代码（最经典写法）

```
@Servicepublic class AIService {// 注入 Spring AI 统一聊天模型@Autowiredprivate ChatModel chatModel;public String askAI(String question) {// ======================// 1. 构造 Prompt（你的问题）// ======================Prompt prompt = new Prompt(List.of(new SystemMessage("你是一个温柔的技术助手"),new UserMessage(question)));// ======================// 2. 调用 AI（ChatModel）// ======================ChatResponse response = chatModel.call(prompt);// ======================// 3. 从返回结果里拿答案// ======================return response.getResult().getOutput().getContent();}}
```

#### 逐行解释这段代码

```
ChatModel chatModel
← 注入SpringAI提供的统一AI调用工具

Prompt prompt
← 把你的问题包装成AI能识别的格式

chatModel.call(prompt)
← 把问题发给AI，等待回复

ChatResponse response
← AI 把回答包装成一个对象给你

response.getResult().getOutput().getContent()
← 拆开对象，拿到真正的文字回答
```

### 六、终极总结

```
ChatModel   = 发送请求的工具（你用它调用AI）
Prompt      = 你的问题（必须包装成这个对象）
ChatResponse= AI 的回复（包装好的结果，要拆开拿内容）
```

**Spring AI 固定公式：**

```
ChatModel.call(Prompt) → ChatResponse
```