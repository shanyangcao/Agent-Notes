eAct 是 **Reasoning（推理）+ Acting（行动）** 的缩写，是让大模型像人类一样 “边思考、边行动、看反馈、再调整”，自主解决复杂任务的核心架构。它解决了纯推理易 “幻觉”、纯行动无逻辑的痛点，是当前 AI 智能体开发的主流范式。

### 一、核心概念：“想 - 做 - 看” 的循环闭环

ReAct 的本质是 **“思考→行动→观察”（TAO 循环）** 的多轮迭代，直到任务完成或达到终止条件。

- **思考（Thought）**：模型做 “内心独白”，分析任务目标、现有信息缺口，规划下一步（比如 “需要查实时天气，得调用天气 API”）。
- **行动（Action）**：模型执行具体操作，调用外部工具（搜索、API、计算器、数据库等），格式通常为 `Action: 工具名(参数)`。
- **观察（Observation）**：接收工具返回结果，提取关键信息，更新上下文。
- **循环终止**：信息足够回答问题时，输出最终答案；达到最大迭代次数则停止。

### 二、应用场景

ReAct 适合**依赖外部信息 / 工具、多步骤推理**的场景，核心解决 “需要事实、无法内部计算、必须环境交互” 的任务，常见场景如下：

| **场景**   | **具体例子**                           | **ReAct 价值**                      |
| ---------- | -------------------------------------- | ----------------------------------- |
| 实时问答   | 查今日某城市天气、某航班价格、最新政策 | 避免幻觉，获取准确实时数据          |
| 多步骤任务 | 对比两年会议举办信息、规划旅行行程     | 分步调用工具，逐步补全信息          |
| 工具调用   | 验证接口状态、计算复杂公式、操作数据库 | 明确 “为什么做、做什么”，不盲目执行 |
| 智能助手   | 个人助理安排日程、检索文档、生成报告   | 动态调整策略，适配任务变化          |

### 三、实现方式

ReAct 实现分**手动搭建**和**框架封装**两种，核心是 “定义工具→配置循环→执行迭代”。

#### 1. 手动搭建

核心步骤（以 Python 为例）：

1. 定义工具：封装可调用的外部能力（如天气查询、接口请求）。
2. 初始化上下文：存储用户问题、历史思考 / 行动 / 观察记录。
3. 执行 TAO 循环：

- - 调用大模型生成思考（Thought）；
  - 解析思考，判断是否需要调用工具；
  - 执行工具，获取观察（Observation）；
  - 追加上下文，重复循环；

1. 输出最终答案。

#### 2. 框架封装

主流框架已内置 ReAct 循环，无需手动维护，**Java 开发者常用 2 种**：

- **Spring AI**：Spring 生态首选，注解式定义工具，极简配置即可启用。代码示例：java运行

```
// 1. 定义工具（注解标记）
@Service
public class WeatherService {
    @Tool(description = "获取指定城市的实时天气") // 描述必须清晰，否则模型不调用
    public String getCurrentWeather(String city) {
        // 调用外部天气 API 获取结果
        return WeatherApiClient.get(city);
    }
}

// 2. 构建并执行 ReAct Agent
@SpringBootApplication
public class ReActAgentApp {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(ReActAgentApp.class, args);
        // 注入模型与工具，框架自动处理 ReAct 循环
        ChatClient chatClient = context.getBean(ChatClient.class);
        String result = chatClient.prompt("北京今天天气怎么样？").call().content();
        System.out.println(result);
    }
}
```

- **LangChain4j**：功能全面，适合复杂 Agent 编排，支持自定义内存与工具绑定。代码示例：java运行

```
// 1. 初始化模型与工具
ChatModel model = OllamaChatModel.builder().modelName("llama3").build();
WeatherTool weatherTool = new WeatherTool();

// 2. 构建 ReAct Agent
ReActAssistant agent = AiServices.builder(ReActAssistant.class)
        .chatModel(model)
        .tools(weatherTool) // 绑定工具
        .chatMemory(MessageWindowChatMemory.withMaxMessages(10)) // 配置内存
        .build();

// 3. 执行任务
String result = agent.chat("上海今天天气如何？");
System.out.println(result);
```

### 四、与其他技术的不同

ReAct 常与 CoT、函数调用、Plan-and-Solve 等对比，核心差异在 “推理与行动的结合方式”，以下用表格直观说明：

| **技术**           | **核心逻辑**                     | **优势**                         | **局限**                   | **适用场景**                              |
| ------------------ | -------------------------------- | -------------------------------- | -------------------------- | ----------------------------------------- |
| **ReAct**          | 思考 - 行动 - 观察循环，交替进行 | 可工具交互、动态调整、可解释性强 | 路径不可预测，调试稍难     | 需实时信息、多步骤任务、工具调用          |
| **CoT（思维链）**  | 仅生成推理步骤，不调用工具       | 简单易用，成本低，逻辑清晰       | 无外部交互，易幻觉         | 数学 / 逻辑推理、常识分析（无需实时数据） |
| **函数调用**       | 模型直接输出结构化指令调用工具   | 执行效率高，结构化结果           | 无自主推理，复杂任务易卡壳 | 简单工具调用、固定流程任务                |
| **Plan-and-Solve** | 先全局规划，再分步执行           | 流程可控，适合长周期任务         | 灵活性差，计划外场景难调整 | 需审计追踪、结果可复现的复杂任务          |