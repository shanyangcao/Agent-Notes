LangGraph 是 LangChain 官方推出的**专门用于构建复杂智能体（Agent）的框架**，可以理解为：

LangChain 是构建 LLM 应用的基础框架，而 LangGraph 是 LangChain 生态中「为 Agent 量身定制的进阶工具」，主打**可控的多步决策、循环、分支、并行执行**，完美解决传统 Agent 流程僵化的问题。

简单说：

- LangChain → 适合构建线性流程（如 RAG 链、简单问答链）；
- LangGraph → 适合构建**有状态、有决策、能循环**的复杂 Agent（如智能客服、数据分析助手、多工具协同机器人）

### LangGraph - 复杂流程的指挥家

#### 核心问题：为什么需要LangGraph？

**传统Chain的局限**

想象你在玩"选择你的冒险"游戏书：

\- 传统Chain：每页只有一个"下一页"，线性剧情

\- 现实需求：需要"如果A则翻到X页，如果B则翻到Y页"

**Chain能做的**：

用户输入 → 步骤1 → 步骤2 → 步骤3 → 输出

**Chain做不了的**：

\1. 循环：步骤3不满意 → 回到步骤1重做

\2. 分支：根据结果走不同路径

\3. 并行：同时执行多个任务

\4. 人工介入：某个节点等待人类确认

\5. 动态调整：根据进度改变流程

####  真实场景举例

**场景：AI写文章助手**

传统Chain的问题：

生成文章 → 结束

**问题**：

\- 质量不好怎么办？（需要重写）

\- 需要多轮打磨怎么办？（需要循环）

\- 想让人审核怎么办？（需要等待）

**LangGraph的方案**：

```
生成初稿
  ↓
自我评估质量
  ↓
质量 < 分？
  Yes → 分析问题 → 重写（回到生成初稿）
  No → 人工审核
        ↓
       批准？
        Yes → 发布
        No → 修改建议 → 重写（回到生成初稿）
这就是LangGraph的威力：可以循环、分支、等待。
```

### 核心概念详解

#### **概念一：State**

**通俗理解**：状态就像游戏存档

玩RPG游戏，存档里保存：

- 角色等级、HP、装备
- 当前位置
- 剧情进度
- 背包物品

**LangGraph的状态**：

```
class WritingState(TypedDict):
    draft: str          # 当前草稿
    version: int        # 第几版
    quality_score: float  # 质量评分
    feedback: List[str]  # 修改意见列表
    approved: bool      # 是否批准
    user_input: str     # 用户输入
```

**状态的作用**：

1. **在节点间传递信息**：写作节点生成草稿，评估节点读取草稿
2. **记录流程进度**：现在第3版了，评分6.5分
3. **支持循环和分支**：根据approved决定走哪条路

**状态更新方式**：

**方式一：替换**

```
# 节点返回全新状态，完全覆盖
def node_func(state):
    return {"draft": "新内容", "version": 2}  # 其他字段丢失
```

**方式二：合并（推荐）**

```
# 节点只返回变化的字段，其他保留
def node_func(state):
    return {"quality_score": 8.5}  # draft等字段保持不变
```

#### **概念二：Node**

**通俗理解**：节点就像工厂的工位

汽车制造：

- 焊接工位：负责焊接
- 喷漆工位：负责喷漆
- 组装工位：负责组装

**LangGraph的节点**：

```
def generate_draft(state):
    """生成初稿节点"""
    prompt = f"写一篇关于{state['topic']}的文章"
    draft = llm.invoke(prompt)
    return {"draft": draft, "version": state["version"] + 1}

def evaluate_quality(state):
    """评估质量节点"""
    prompt = f"评估这篇文章质量(1-10分)：{state['draft']}"
    score = llm.invoke(prompt)
    return {"quality_score": float(score)}

def human_review(state):
    """人工审核节点"""
    print(f"请审核文章：{state['draft']}")
    approval = input("是否批准？(y/n): ")
    return {"approved": approval == "y"}
```

**节点可以做任何事**：

- 调用LLM
- 执行工具（搜索、计算）
- 访问数据库
- 调用API
- 等待人工输入
- 纯逻辑计算

#### **概念三：Edge**

**通俗理解**：边就像路标指示牌

**普通边（无条件）**：

```
graph.add_edge("生成初稿", "评估质量")
# 生成初稿做完，一定去评估质量
```

**条件边（带判断）**：

```
def route_after_evaluation(state):
    """根据质量分数决定走向"""
    if state["quality_score"] >= 8:
        return "human_review"  # 质量好，去人工审核
    else:
        return "generate_draft"  # 质量差，重新生成

graph.add_conditional_edges(
    "evaluate_quality",  # 从评估节点出来
    route_after_evaluation,  # 用这个函数决定
    {
        "human_review": "human_review",  # 映射到实际节点名
        "generate_draft": "generate_draft"
    }
)
```

**条件边的威力**：

- 实现循环：quality不好 → 回到生成节点
- 实现分支：approved → 发布，不approved → 修改
- 动态路由：根据状态走不同路径

#### **概念四：Graph**

**通俗理解**：图就是整个流程蓝图

```
from langgraph.graph import StateGraph

# 1. 定义图
workflow = StateGraph(WritingState)

# 2. 添加节点
workflow.add_node("generate", generate_draft)
workflow.add_node("evaluate", evaluate_quality)
workflow.add_node("review", human_review)
workflow.add_node("publish", publish_article)

# 3. 连接边
workflow.add_edge("generate", "evaluate")  # 生成 → 评估

# 4. 添加条件边
workflow.add_conditional_edges(
    "evaluate",
    lambda s: "review" if s["quality_score"] >= 8 else "generate",
    {"review": "review", "generate": "generate"}
)

workflow.add_conditional_edges(
    "review",
    lambda s: "publish" if s["approved"] else "generate",
    {"publish": "publish", "generate": "generate"}
)

# 5. 设置入口和出口
workflow.set_entry_point("generate")
workflow.add_edge("publish", END)

# 6. 编译成可执行的
app = workflow.compile()

# 7. 运行
result = app.invoke({"topic": "AI的未来", "version": 0})
```

#### 类比java的理解

- **State** = 方法的**入参 + 出参 + 上下文对象**

你要干活，得有数据；干完活，数据还会变。

- **Node** = 一个单独的**业务方法 / 函数**

每个节点就是一段逻辑：做校验、查库、调接口、计算……

- **Edge** = 方法之间的**调用关系 / 跳转逻辑**

方法 A 执行完，下一步去哪儿？是顺序执行？还是根据结果判断走不同分支？

- **Graph** = 整个**流程引擎 / 工作流类**

把所有方法、跳转规则、数据上下文组装在一起，形成一个可运行的完整业务流程。

### 高级模式实战

#### **模式一：多智能体协作**

**场景**：软件开发团队

**传统方式**：一个AI干所有活，容易混乱

**LangGraph方式**：每个AI是专家

```
class DevelopmentState(TypedDict):
    requirements: str       # 需求文档
    design: str            # 设计方案
    code: str              # 代码
    test_results: str      # 测试结果
    bugs: List[str]        # Bug列表
    current_agent: str     # 当前哪个Agent在工作

# 产品经理Agent
def product_manager(state):
    """理解需求，输出需求文档"""
    requirements = llm.invoke(
        f"将用户需求转化为详细需求文档：{state['user_request']}"
    )
    return {"requirements": requirements, "current_agent": "architect"}

# 架构师Agent
def architect(state):
    """设计技术方案"""
    design = llm.invoke(
        f"根据需求设计架构：{state['requirements']}"
    )
    return {"design": design, "current_agent": "developer"}

# 开发Agent
def developer(state):
    """写代码"""
    code = llm.invoke(
        f"根据设计实现代码：{state['design']}"
    )
    return {"code": code, "current_agent": "tester"}

# 测试Agent
def tester(state):
    """测试并找bug"""
    test_results = run_tests(state["code"])
    bugs = extract_bugs(test_results)
    
    return {
        "test_results": test_results,
        "bugs": bugs,
        "current_agent": "developer" if bugs else "done"
    }

# 路由函数
def route_agent(state):
    return state["current_agent"]

# 构建图
workflow = StateGraph(DevelopmentState)
workflow.add_node("pm", product_manager)
workflow.add_node("architect", architect)
workflow.add_node("developer", developer)
workflow.add_node("tester", tester)

workflow.set_entry_point("pm")
workflow.add_edge("pm", "architect")
workflow.add_edge("architect", "developer")
workflow.add_edge("developer", "tester")

# 测试后的条件边：有bug回到开发，没bug结束
workflow.add_conditional_edges(
    "tester",
    lambda s: "developer" if s["bugs"] else END,
    {"developer": "developer", END: END}
)

app = workflow.compile()
```

**这个流程的特点**：

- ✅ 每个Agent专精一个领域
- ✅ 有bug自动循环修复
- ✅ 流程清晰可追踪
- ✅ 可以随时暂停查看状态

#### **模式二：人机协作流程**

**场景**：内容审核发布

**关键点**：某些节点需要等待人类输入

```
class ContentState(TypedDict):
    topic: str
    draft: str
    human_feedback: str
    final_version: str
    approved: bool

def generate_content(state):
    draft = llm.invoke(f"写一篇关于{state['topic']}的文章")
    return {"draft": draft}

def wait_for_human_review(state):
    """这个节点会暂停，等待人类"""
    # 在实际应用中，这里会：
    # 1. 把草稿发给审核人员
    # 2. 存储状态到数据库
    # 3. 返回特殊标记表示等待
    print(f"=== 请审核以下内容 ===\n{state['draft']}\n")
    
    # 模拟等待（实际中会真的暂停）
    feedback = input("请输入反馈（直接回车表示批准）：")
    
    if feedback:
        return {"human_feedback": feedback, "approved": False}
    else:
        return {"approved": True, "final_version": state['draft']}

def revise_content(state):
    revised = llm.invoke(
        f"根据反馈修改文章：\n原文：{state['draft']}\n反馈：{state['human_feedback']}"
    )
    return {"draft": revised}

# 构建图
workflow = StateGraph(ContentState)
workflow.add_node("generate", generate_content)
workflow.add_node("review", wait_for_human_review)
workflow.add_node("revise", revise_content)
workflow.add_node("publish", lambda s: {"final_version": s['draft']})

workflow.set_entry_point("generate")
workflow.add_edge("generate", "review")

workflow.add_conditional_edges(
    "review",
    lambda s: "publish" if s["approved"] else "revise",
    {"publish": "publish", "revise": "revise"}
)

workflow.add_edge("revise", "review")  # 修改后再审核
workflow.add_edge("publish", END)

app = workflow.compile()
```

**实际生产中的实现**：

```
# 状态持久化到Redis
def wait_for_human_review(state):
    # 保存状态
    redis.set(f"review:{state['id']}", json.dumps(state))
    
    # 发送通知给审核人员
    send_notification(state['reviewer_email'], state['draft'])
    
    # 返回WAIT特殊标记
    return {"status": "WAITING_FOR_HUMAN"}

# 人类审核后恢复
def resume_after_review(review_id, feedback):
    # 从Redis读取状态
    state = json.loads(redis.get(f"review:{review_id}"))
    state['human_feedback'] = feedback
    
    # 继续执行workflow
    app.invoke(state, starting_node="revise")
```

#### **模式三：自我反思循环**

**场景**：AI写代码并自我检查

```
class CodingState(TypedDict):
    task: str
    code: str
    test_results: str
    self_review: str
    iteration: int
    max_iterations: int

def write_code(state):
    """写代码"""
    prompt = f"任务：{state['task']}"
    if state.get('self_review'):
        prompt += f"\n上次问题：{state['self_review']}\n请改进"
    
    code = llm.invoke(prompt)
    return {"code": code, "iteration": state["iteration"] + 1}

def test_code(state):
    """运行测试"""
    results = execute_code(state['code'])
    return {"test_results": results}

def self_critique(state):
    """自我评估"""
    review = llm.invoke(
        f"评估这段代码：\n{state['code']}\n测试结果：{state['test_results']}\n找出问题"
    )
    return {"self_review": review}

def should_continue(state):
    """决定是继续还是结束"""
    # 达到最大迭代次数 → 结束
    if state["iteration"] >= state["max_iterations"]:
        return "end"
    
    # 测试全通过 → 结束
    if "PASS" in state["test_results"] and "FAIL" not in state["test_results"]:
        return "end"
    
    # 否则继续改进
    return "continue"

# 构建图
workflow = StateGraph(CodingState)
workflow.add_node("write", write_code)
workflow.add_node("test", test_code)
workflow.add_node("critique", self_critique)

workflow.set_entry_point("write")
workflow.add_edge("write", "test")
workflow.add_edge("test", "critique")

workflow.add_conditional_edges(
    "critique",
    should_continue,
    {
        "continue": "write",  # 回到写代码（循环）
        "end": END
    }
)

app = workflow.compile()

# 运行
result = app.invoke({
    "task": "写一个快速排序函数",
    "iteration": 0,
    "max_iterations": 5
})
```

**这个模式的威力**：

- 第1次：写出基本版本
- 第2次：发现边界情况bug，修复
- 第3次：优化性能
- 第4次：添加错误处理
- 第5次：完善文档

比单次生成质量高很多！

### 状态管理深入技巧

#### **技巧一：状态快照**

记录每个节点执行后的状态，用于调试和回溯。

```
class StatefulGraph:
    def __init__(self):
        self.snapshots = []  # 保存每次状态
    
    def run_node(self, node_func, state):
        # 执行节点
        new_state = node_func(state)
        
        # 保存快照
        self.snapshots.append({
            "node": node_func.__name__,
            "state_before": copy.deepcopy(state),
            "state_after": copy.deepcopy(new_state),
            "timestamp": datetime.now()
        })
        
        return new_state
    
    def replay_to_step(self, step_num):
        """回到某个步骤的状态"""
        return self.snapshots[step_num]["state_after"]
```

**用途**：

- 调试：看到每步状态变化
- 回溯：发现错误，回到之前某步
- 审计：记录完整执行历史

#### **技巧二：并发状态合并**

当多个节点并行执行时，如何合并状态？

```
def parallel_research(state):
    """并行搜索多个数据源"""
    
    # 同时启动3个搜索
    results = await asyncio.gather(
        search_academic_papers(state['query']),
        search_news(state['query']),
        search_company_docs(state['query'])
    )
    
    # 合并结果
    return {
        "academic_results": results[0],
        "news_results": results[1],
        "docs_results": results[2],
        "all_results": results[0] + results[1] + results[2]
    }
```

**冲突处理**：

```
def merge_states(state1, state2):
    """两个并发节点的状态合并"""
    merged = state1.copy()
    
    for key, value in state2.items():
        if key not in merged:
            merged[key] = value
        elif isinstance(value, list):
            # 列表合并
            merged[key].extend(value)
        elif isinstance(value, dict):
            # 字典合并
            merged[key].update(value)
        else:
            # 冲突：取最新的
            merged[key] = value
    
    return merged
```

### 实际生产部署考虑

#### **考虑一：长时间运行**

复杂流程可能运行几小时，怎么办？

**解决方案：检查点Checkpoint**

```
class CheckpointedGraph:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def run(self, workflow_id, initial_state):
        state = initial_state
        
        for node in self.nodes:
            # 执行节点
            state = node(state)
            
            # 保存检查点
            self.redis.set(
                f"checkpoint:{workflow_id}",
                json.dumps({
                    "current_node": node.name,
                    "state": state,
                    "timestamp": time.time()
                }),
                ex=86400  # 24小时过期
            )
        
        return state
    
    def resume(self, workflow_id):
        """从检查点恢复"""
        checkpoint = json.loads(self.redis.get(f"checkpoint:{workflow_id}"))
        
        # 从中断的节点继续
        start_node = checkpoint["current_node"]
        state = checkpoint["state"]
        
        return self.run_from_node(start_node, state)
```

#### **考虑二：错误处理**

节点执行失败怎么办？

**策略一：重试**

```
def retry_node(node_func, state, max_retries=3):
    for attempt in range(max_retries):
        try:
            return node_func(state)
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            time.sleep(2  attempt)  # 指数退避
```

**策略二：降级**

```
def node_with_fallback(state):
    try:
        # 尝试主要方法（调用GPT-4）
        return expensive_llm_call(state)
    except RateLimitError:
        # 降级到便宜的模型
        return cheap_llm_call(state)
    except Exception:
        # 完全失败，返回默认
        return {"result": "暂时无法处理，请稍后重试"}
```

**策略三：人工介入**

```
def node_with_human_fallback(state):
    try:
        return ai_process(state)
    except Exception as e:
        # AI失败，转人工
        return {
            "status": "ESCALATED_TO_HUMAN",
            "error": str(e),
            "state": state
        }
```

#### **考虑三：监控和可观测性**

**指标收集**：

```
def instrumented_node(node_func):
    def wrapper(state):
        start_time = time.time()
        
        try:
            result = node_func(state)
            
            # 记录成功指标
            metrics.counter("node.success", tags=[f"node:{node_func.__name__}"])
            metrics.histogram("node.duration", time.time() - start_time)
            
            return result
        
        except Exception as e:
            # 记录失败指标
            metrics.counter("node.error", tags=[
                f"node:{node_func.__name__}",
                f"error:{type(e).__name__}"
            ])
            
            raise
    
    return wrapper
```

**日志追踪**：

```
import uuid

def run_with_tracing(workflow, state):
    trace_id = str(uuid.uuid4())
    
    logger.info(f"[{trace_id}] Workflow started", extra={
        "trace_id": trace_id,
        "initial_state": state
    })
    
    for node in workflow.nodes:
        logger.info(f"[{trace_id}] Executing node: {node.name}")
        
        state = node(state)
        
        logger.info(f"[{trace_id}] Node complete", extra={
            "node": node.name,
            "state": state
        })
    
    return state
​```

### 适用场景总结

#### ✅ 强烈推荐用LangGraph的场景

1. 需要反复迭代
- 内容创作（写-评估-改-再评估）
- 代码生成（写-测试-修复-再测试）
- 研究分析（搜索-分析-深入搜索-综合）

2. 需要人工介入
- 审核流程（AI生成-人审核-修改-再审核）
- 决策辅助（AI建议-人决策-执行-反馈）

3. 多Agent协作
- 虚拟团队（产品+设计+开发+测试）
- 专家系统（法律+财务+技术多角度分析）

4. 复杂决策树
- 客服路由（识别问题类型-分配专家-解决-回访）
- 故障诊断（症状-检查-诊断-处理-验证）

#### ❌ 不需要LangGraph的场景

1. 简单单次调用
- 问答："什么是AI？"（直接调API）
- 翻译："把这段翻译成英文"（一次搞定）

2. 固定线性流程
- 数据处理：读取-清洗-分析-输出（没有分支）
- 报告生成：收集数据-格式化-输出（不需要循环）

3. 性能要求极高
- 实时响应（<100ms）场景
- 图调度有开销，不如直接写代码

---

## 🎯 三者对比与选型指南

### 核心定位对比

| 维度 | LangChain | LlamaIndex | LangGraph |
|------|-----------|------------|-----------|
| 核心能力 | 全栈AI应用框架 | 数据检索专家 | 流程编排大师 |
| 主攻方向 | 快速原型 | RAG系统 | 复杂工作流 |
| 学习曲线 | ⭐⭐⭐ 中等 | ⭐⭐⭐⭐ 较陡 | ⭐⭐⭐⭐⭐ 最陡 |
| 适合人群 | 初学者、全栈开发 | 数据工程师 | 架构师、高级开发 |
| 生态成熟度 | ⭐⭐⭐⭐⭐ 最成熟 | ⭐⭐⭐⭐ 成熟 | ⭐⭐⭐ 新兴 |

### 技术架构对比

抽象程度：
​```
LangChain：高抽象（封装最多，灵活性中等）
    ↓
LlamaIndex：中抽象（专注检索，该领域很灵活）
    ↓
LangGraph：低抽象（接近底层，最灵活但最复杂）
​```

性能对比（相同任务）：
​```
直接调用API：100ms ⚡
LangGraph：150ms ⚡⚡（图调度开销小）
LlamaIndex：200ms ⚡⚡⚡（检索计算量大）
LangChain：300ms ⚡⚡⚡⚡（抽象层最多）
​```

### 组合使用策略

#### 策略一：检索+编排

场景：智能客服系统
​```
LangGraph（整体流程）
    ├─ 理解用户意图
    ├─ LlamaIndex（检索知识库）
    ├─ 生成回复
    ├─ 质量检查
    └─ 如果不满意，循环
```

**代码概念**：

```
# LlamaIndex构建知识库
from llama_index import VectorStoreIndex

docs = load_documents()
index = VectorStoreIndex.from_documents(docs)
query_engine = index.as_query_engine()

# LangGraph编排流程
def retrieve_node(state):
    """用LlamaIndex检索"""
    results = query_engine.query(state['user_question'])
    return {"retrieved_info": results}

def generate_node(state):
    """生成回复"""
    answer = llm.invoke(
        f"根据信息：{state['retrieved_info']}\n回答：{state['user_question']}"
    )
    return {"answer": answer}

# 构建图
workflow = StateGraph(...)
workflow.add_node("retrieve", retrieve_node)
workflow.add_node("generate", generate_node)
...
​```

#### 策略二：全家桶组合

场景：企业级研究助手
​```
LangGraph（最外层流程控制）
    │
    ├─ 任务规划节点
    │   └─ LangChain（Chain把大任务拆小）
    │
    ├─ 检索节点
    │   └─ LlamaIndex（查内部文档）
    │
    ├─ 搜索节点
    │   └─ LangChain（调用搜索工具）
    │
    ├─ 分析节点
    │   └─ LangChain（多步推理Chain）
    │
    └─ 汇总节点
        └─ LlamaIndex（树形合成）
​```

### 选型决策树
​```
你的项目需求
    │
    ├─ 主要是查询文档？
    │   ├─ Yes → LlamaIndex ✓
    │   └─ No → 继续
    │
    ├─ 需要复杂流程控制（循环/分支/并行）？
    │   ├─ Yes → LangGraph ✓
    │   └─ No → 继续
    │
    ├─ 快速原型验证，标准场景？
    │   ├─ Yes → LangChain ✓
    │   └─ No → 继续
    │
    └─ 高度定制，性能优先？
        └─ 都不用，直接调API ✓
​```

### 实际案例分析

#### 案例一：法律咨询AI

需求分析：
- 查询海量法律文书（✓ 需要LlamaIndex）
- 多轮对话，根据回答深入（✓ 需要LangGraph）
- 调用法规数据库（✓ LangChain工具集成）

架构设计：
​```
LangGraph（对话流程管理）
    │
    ├─ 理解问题节点
    │   └─ LangChain（意图识别Chain）
    │
    ├─ 检索法律文档节点
    │   └─ LlamaIndex（向量索引+关键词混合检索）
    │
    ├─ 查询法规数据库节点
    │   └─ LangChain（SQL工具）
    │
    ├─ 生成法律意见节点
    │   └─ 直接调用LLM
    │
    ├─ 人工律师审核节点
    │   └─ LangGraph（等待人工输入）
    │
    └─ 根据审核决定循环或输出
​```

#### 案例二：内容创作平台

需求：
- AI写文章
- 多轮优化迭代
- 参考网络资料
- 人工审核发布

选型：
- 主框架：LangGraph（迭代流程）
- 辅助1：LangChain（调用搜索API）
- 辅助2：LlamaIndex（查询写作素材库）
​```
初始化：用户输入主题
    ↓
【LangChain搜索节点】搜索相关资料
    ↓
【LlamaIndex检索节点】查内部素材库
    ↓
【生成初稿】
    ↓
【自我评估】
    ↓
质量<8? → Yes → 【分析不足】→ 回到生成初稿
    ↓ No
【人工审核】
    ↓
批准? → Yes → 发布
    ↓ No
【根据反馈修改】→ 回到生成初稿
​```

---

## 🚀 深入专题

### 专题一：Prompt工程的决定性作用

无论用什么框架，Prompt质量 = 80%的效果差异。

#### 黄金原则

1. 角色设定清晰

❌ 差的prompt：
​```
"帮我写个销售报告"
​```

✅ 好的prompt：
​```
"你是一位有10年经验的销售分析师，擅长用数据讲故事。
请为CEO撰写Q4销售报告，重点突出：
1. 同比增长趋势
2. 主要驱动因素
3. 风险预警
语气：专业但不失人性化，数据用可视化描述"
​```

效果差异：好prompt输出直接可用，差prompt要改5遍

2. 给出示例（Few-shot）

❌ 没示例：
​```
"提取这段话的关键信息"
​```

✅ 有示例：
​```
"提取关键信息，格式如下：

示例输入：'苹果公司今天发布了iPhone 15，售价999美元，将于9月22日开售'
示例输出：
- 公司：苹果
- 产品：iPhone 15
- 价格：$999
- 发售日期：2024-09-22

现在处理：{实际文本}"
​```

效果差异：准确率从60%提升到95%

3. 思维链Chain-of-Thought

❌ 直接问：
​```
"23 * 47 = ?"
​```

✅ 让它展示思考：
​```
"计算23 * 47，请一步步展示计算过程：
1. 先算...
2. 再算...
3. 最后..."
​```

效果差异：复杂推理任务准确率提升30-50%

#### 针对不同模型调整

GPT-4特点：
- 强项：复杂推理、创意、理解隐含意图
- 弱项：啰嗦、有时过度解释
- Prompt策略：简洁指令，信任它的理解力

Claude特点：
- 强项：精炼、遵循格式、安全意识强
- 弱项：需要明确指令，不太擅长猜测
- Prompt策略：详细指令，明确格式要求

开源模型（Llama/Mistral）：
- 强项：便宜、可控
- 弱项：理解力弱，需要更多引导
- Prompt策略：更多示例，更详细的分步指令

### 专题二：评估体系建设

没有评估 = 瞎折腾

#### 评估维度

1. 检索系统评估

指标一：MRR（Mean Reciprocal Rank）
​```
查询："如何提高转化率"
返回10个文档，正确答案在第3个

MRR = 1/3 = 0.33

如果正确答案在第1个，MRR = 1（完美）
如果前10个都没有正确答案，MRR = 0（最差）
​```

指标二：Recall@K（召回率）
​```
一共有5个相关文档
检索返回前10个文档，包含其中3个相关的

Recall@10 = 3/5 = 0.6（60%召回率）
​```

指标三：NDCG（归一化折损累积增益）
​```
考虑排序质量：
- 相关文档排在前面，得分高
- 相关文档排在后面，得分打折

复杂公式略，工具库有现成实现
​```

2. 生成质量评估

传统方法：人工打分
​```
维度1：准确性（0-10分）
维度2：相关性（0-10分）
维度3：流畅性（0-10分）
维度4：完整性（0-10分）

缺点：贵、慢、不scalable
```

**现代方法：LLM-as-Judge**

```
def llm_evaluate(query, answer, ground_truth):
    eval_prompt = f"""
    评估AI回答质量（1-10分）：
    
    问题：{query}
    AI回答：{answer}
    标准答案：{ground_truth}
    
    评分标准：
    - 准确性（40%权重）
    - 完整性（30%权重）
    - 表达清晰度（30%权重）
    
    请给出分数和理由。
    """
    
    result = gpt4.invoke(eval_prompt)
    return parse_score(result)

# 对100个测试用例批量评估
scores = [llm_evaluate(q, a, gt) for q, a, gt in test_cases]
avg_score = sum(scores) / len(scores)
​```

优点：快、便宜、可复现
缺点：评估者本身可能有偏差

#### 建立测试集

步骤一：收集真实查询
​```
从用户日志中抽取：
- 高频问题
- 失败case
- 边界case
- 各类型问题都要覆盖
​```

步骤二：人工标注答案
​```
每个问题标注：
- 正确答案是什么
- 应该检索哪些文档
- 回答重点是什么

投入：至少100个问题，理想500+
```

**步骤三：定期评估**

```
# 每次改动后跑评估
results = []
for test_case in test_set:
    answer = system.query(test_case.question)
    score = evaluate(answer, test_case.ground_truth)
    results.append(score)

print(f"平均分：{mean(results)}")
print(f"及格率（>7分）：{sum(s >= 7 for s in results) / len(results)}")

# 跟踪指标变化
plot_metrics_over_time(results, version="v2.3")
​```

### 专题三：成本优化实战

AI应用的成本主要在LLM调用，必须重视。

#### 成本拆解

典型RAG应用的成本构成：
​```
Embedding（向量化） - 10%
检索计算 - 5%
LLM调用 - 80% ← 重点优化
其他 - 5%
```

#### **优化策略**

**策略一：智能缓存**

```
import hashlib
from functools import lru_cache

class SemanticCache:
    def __init__(self, similarity_threshold=0.95):
        self.cache = {}  # {query_hash: (query_embedding, answer)}
        self.threshold = similarity_threshold
    
    def get(self, query):
        query_emb = embed(query)
        
        # 查找相似查询
        for cached_emb, answer in self.cache.values():
            similarity = cosine_similarity(query_emb, cached_emb)
            if similarity > self.threshold:
                return answer  # 命中缓存！
        
        return None
    
    def set(self, query, answer):
        query_emb = embed(query)
        query_hash = hashlib.md5(query.encode()).hexdigest()
        self.cache[query_hash] = (query_emb, answer)

# 使用
cache = SemanticCache()

def query_with_cache(question):
    # 先查缓存
    cached = cache.get(question)
    if cached:
        print("缓存命中！节省成本")
        return cached
    
    # 缓存未命中，调用LLM
    answer = expensive_llm_call(question)
    cache.set(question, answer)
    return answer
```

**效果：缓存命中率30-50%，直接节省30-50%成本**

**策略二：模型分级**

```
class TieredLLM:
    def __init__(self):
        self.cheap_model = "gpt-3.5-turbo"  # $0.001/1K tokens
        self.expensive_model = "gpt-4"      # $0.03/1K tokens
    
    def route_query(self, query):
        """判断查询复杂度"""
        # 简单规则
        if len(query) < 50 and "?" in query:
            return "simple"
        
        # 用便宜模型判断
        complexity = cheap_llm.invoke(
            f"这个问题复杂吗（简单/中等/复杂）：{query}"
        )
        
        if "简单" in complexity or "中等" in complexity:
            return "simple"
        else:
            return "complex"
    
    def query(self, question):
        complexity = self.route_query(question)
        
        if complexity == "simple":
            print("使用GPT-3.5（便宜）")
            return call_model(self.cheap_model, question)
        else:
            print("使用GPT-4（贵但准）")
            return call_model(self.expensive_model, question)

# 效果：
# 假设80%问题是简单的
# 成本 = 0.8 * $0.001 + 0.2 * $0.03 = $0.0068
# 比全用GPT-4便宜77%！
```

**策略三：Prompt压缩**

```
def compress_prompt(verbose_prompt):
    """去除冗余，压缩prompt"""
    
    # 1. 去除示例中的重复模式
    # 2. 用更简洁的表达
    # 3. 合并相似指令
    
    compressed = llm.invoke(
        f"把这个prompt压缩到一半长度，保持关键信息：\n{verbose_prompt}"
    )
    
    return compressed

# 示例
原prompt = """
你是一个非常专业的客服助手，有着丰富的经验和知识。
你需要帮助用户解决他们遇到的各种各样的问题和困难。
当用户向你提问时，你要仔细倾听和理解他们的需求...
（500字）
"""

压缩后 = """
专业客服助手，解决用户问题。
要求：仔细理解需求，给出清晰准确的答案。
（50字）
"""

# 节省90% token！
```

**策略四：批处理**

```
# ❌ 低效：逐个处理
for item in items:
    result = llm.invoke(f"处理：{item}")
    results.append(result)
# 调用100次

# ✅ 高效：批量处理
batch_prompt = "批量处理以下项目（每行一个）：\n"
for i, item in enumerate(items):
    batch_prompt += f"{i+1}. {item}\n"

results = llm.invoke(batch_prompt)
# 调用1次！
```

**策略五：输出长度控制**

```
# ❌ 不限制
answer = llm.invoke(query)  # 可能生成1000字废话

# ✅ 限制
answer = llm.invoke(
    query,
    max_tokens=200  # 最多200 tokens
)

# 或在prompt里限制
prompt = f"{query}\n请用不超过100字回答。"
```

#### **成本监控仪表盘**

```
class CostTracker:
    def __init__(self):
        self.costs = []
    
    def log_call(self, model, input_tokens, output_tokens):
        cost = self.calculate_cost(model, input_tokens, output_tokens)
        self.costs.append({
            "timestamp": datetime.now(),
            "model": model,
            "tokens": input_tokens + output_tokens,
            "cost": cost
        })
    
    def daily_report(self):
        today = datetime.now().date()
        today_costs = [c for c in self.costs if c["timestamp"].date() == today]
        
        total = sum(c["cost"] for c in today_costs)
        by_model = {}
        for c in today_costs:
            by_model[c["model"]] = by_model.get(c["model"], 0) + c["cost"]
        
        print(f"今日总成本：${total:.2f}")
        for model, cost in by_model.items():
            print(f"  {model}: ${cost:.2f}")
        
        return total

# 设置预警
if tracker.daily_report() > 100:
    send_alert("日成本超过$100！")
```

### 专题四：生产部署checklist

#### **上线前必查清单**

**1. 性能测试 ✓**

-  压力测试：并发100用户，响应时间<3秒
-  长时间运行：24小时稳定性测试
-  异常处理：各种错误都能优雅处理

**2. 成本预估 ✓**

-  每个用户会话平均成本
-  日均用户量 × 平均成本 = 日成本
-  设置预算上限和预警

**3. 安全措施 ✓**

-  Prompt注入防护
-  敏感信息过滤
-  访问权限控制
-  审计日志完整

**4. 监控告警 ✓**

-  成功率监控（>95%）
-  延迟监控（P99 < 5s）
-  错误率监控（<1%）
-  成本监控（日预算）

**5. 灾备方案 ✓**

-  API失败降级策略
-  数据备份恢复
-  回滚方案
-  应急联系人