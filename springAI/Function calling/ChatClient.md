## 一、ChatClient：Spring AI 的核心聊天入口

**ChatClient** 是 Spring AI 提供的**新一代对话客户端**，用来替代早期的 `ChatModel`，它采用流式 Builder 设计，让你更简洁、更直观地和大模型交互。

- **核心作用**：统一封装对话请求、参数配置、工具调用、结果解析等逻辑，让你用一套代码就能对接不同大模型（OpenAI、豆包、通义千问等）。
- **典型用法**：

```
String answer = chatClient.prompt().user("请解释Spring AI的ChatClient是什么").call().content();
```

- **一句话理解**：`ChatClient` 就是你和 AI 对话的「专属聊天窗口」，所有交互都从这里发起。

## 二、default：给 ChatClient 预设默认行为

`default` 是 `ChatClient.Builder` 上的配置方法，用来**提前设置全局默认值**，避免在每个请求里重复写相同配置。

- **常见默认配置**：


- - 默认系统提示词（`defaultSystem`）
  - 默认模型参数（`defaultOptions`）
  - 默认工具 / Advisor（`defaultTools`/`defaultAdvisors`）


- **代码示例**：

```
@Beanpublic ChatClient chatClient(ChatClient.Builder builder) {return builder
        .defaultSystem("你是专业Java后端工程师，回答简洁通俗").defaultOptions(ChatOptions.builder().temperature(0.7).build()).build();}
```

- **一句话理解**：`default` 就是给聊天窗口「预设模板」，之后所有对话自动继承这些配置，不用每次都重新写。

## 三、Options：精细控制 AI 生成行为

`Options`（通常是 `ChatOptions`）是**控制****大模型****输出风格和行为的参数集合**，相当于 AI 的「控制面板」。

- **常用参数**：


- - `temperature`：控制随机性（0 = 严谨保守，1 = 创意发散）
  - `maxTokens`：限制单次回答的最大 token 数
  - `model`：指定底层使用的模型名称
  - `topP`：控制采样多样性


- **代码示例**：

```
String answer = chatClient.prompt().user("写一首关于春天的诗").options(ChatOptions.builder().temperature(0.9).maxTokens(500).build()).call().content();
```

- **一句话理解**：`Options` 让你精准调教 AI，想让它严谨就降温，想让它放飞就升温。

## 四、Functions & Tools：让 AI 调用外部工具

`Functions`（旧版）和 `Tools`（新版）是 Spring AI 实现 ** 工具调用（Function Calling）** 的核心，让大模型能调用你写的 Java 方法，连接真实世界。

- **核心作用**：AI 可以根据用户问题，自动判断是否需要调用外部接口（比如查天气、查订单、算数学），并把结果整理成自然语言回答。
- **代码示例**：

```
// 1. 定义工具方法@Componentpublic class WeatherService {public String getWeather(String city) {return city + "今天晴，25℃";}}// 2. 注册为 Function Bean@Beanpublic Function<WeatherRequest, String> weatherFunction(WeatherService service) {return req -> service.getWeather(req.city());}// 3. 让 AI 使用工具String answer = chatClient.prompt().user("北京今天天气怎么样？").tools(weatherFunction).call().content();
```

- **一句话理解**：`Functions/Tools` 给 AI 装上「插件」，让它从 “纸上谈兵” 变成能干活的助手。

## 五、System&User：对话的核心角色与内容

`System` 和 `User` 是对话消息的两种核心角色，构成了 Prompt 的基础结构：

- **System 消息**：给 AI 设定身份、规则、语气（比如「你是温柔的英语老师」）
- **User 消息**：用户的实际问题或指令
- **代码示例**：

```
String answer = chatClient.prompt().system("你是严厉的英语老师，专门纠正语法错误").user("I am go to school. 这句话对吗？").call().content();
```

- **一句话理解**：`System` 给 AI 定人设，`User` 提问题，两者共同构成完整对话上下文。

## 六、Advisors：给对话加「中间件」能力

`Advisors` 是 Spring AI 的**高级扩展机制**，相当于对话流程的「AOP 拦截器」，可以在请求前后插入自定义逻辑。

- **典型场景**：


- - **RAG 检索**：自动从知识库查相关内容，注入 Prompt
  - **内容审核**：检查用户输入 / AI 输出是否合规
  - **日志监控**：记录对话耗时、token 用量
  - **结果后处理**：格式化、过滤回答内容


- **代码示例（RAG）**：

```
@Beanpublic Advisor ragAdvisor(VectorStore vectorStore) {return QuestionAnswerAdvisor.builder().vectorStore(vectorStore).build();}String answer = chatClient.prompt().user("Spring AI的ChatClient是什么？").advisors(ragAdvisor).call().content();
```

- **一句话理解**：`Advisors` 给对话加「智能助理」，自动帮你做检索、审核、日志等脏活累活。

## 七、完整调用流程总结

一次典型的 Spring AI 对话流程是：

1. 用 `ChatClient.Builder` + `default` 预设全局配置
2. 发起 `prompt()` 流式调用
3. 用 `system()`/`user()` 设定角色和问题
4. 用 `options()` 精细控制 AI 行为
5. 用 `tools()` 给 AI 安装插件
6. 用 `advisors()` 加中间件能力
7. 调用 `call()` 发送请求，用 `content()` 提取最终回答

```
String answer = chatClient.prompt().system("你是专业Java顾问").user("请解释Kafka增量聚合").options(ChatOptions.builder().temperature(0.6).build()).tools(weatherFunction).advisors(ragAdvisor).call().content();
```

## 八、核心记忆口诀

**ChatClient 是入口，default 设模板；**

**Options 调参数，System&User 定对话；**

**Tools 装插件，Advisors 加扩展。**