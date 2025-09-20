# LangChain学习


<!--more-->



## 介绍

LangChain 旨在简化 AI 应用开发，通过标准化接口将 LLM 与外部系统（数据源、工具、向量数据库等）无缝连接，帮助开发者构建可扩展、模块化的智能应用。



**主要功能如下：**

- Prompt templates：Prompt templates 是不同类型提示的模板。例如“ chatbot ”样式模板、ELI5 问答等
- LLMs：像 GPT-3、BLOOM 等大型语言模型
- Agents：Agents 使用 LLMs 决定应采取的操作。可以使用诸如网络搜索或计算器之类的工具，并将所有工具包装成一个逻辑循环的操作。
- Memory：短期记忆、长期记忆。



**生态系统：**

- LangSmith：用于监控、调试和评估 LLM 应用性能（如跟踪 Agent 轨迹）。
- LangGraph：构建长期记忆和复杂状态控制的 Agent 工作流（被 LinkedIn、Uber 等企业采用）。
- LangGraph Platform：可视化部署和扩展 Agent 的云平台。





这里以Deepseek为例

## 安装环境

```bash
pip install -U langchain  # 安装最新版
pip install -U langchain-core langchain-community langchain-openai # 安装核心库
pip install python-dotenv
```

> 版本提示：建议使用 langchain-core ≥0.1.0 和 langchain-openai ≥0.0.5

- langchain_core：包含框架基础类和接口
- langchain_openai：提供与 OpenAI 兼容的接口（可适配 DeepSeek）
- langchain：集成高级功能如内存管理





## 快速开始

不同调用大模型的方式：



**1.【init_chat_model】方式**

```python
# 1.调用DeepSeek大模型-【init_chat_model】方式



from langchain.chat_models import init_chat_model
# 获取环境变量、操作文件路径等，这里主要用它来获取环境变量(硬编码容易泄露)
import os
# 从环境变量中读取API KEY
api_key = os.getenv('DEEPSEEK_API_KEY')
base_url = "https://api.deepseek.com/"


model = init_chat_model(
    model='deepseek-chat',  # 指定使用的模型名称，需与平台支持的模型名匹配
    # 以下为传入 DeepSeek 模型需要的额外参数，比如 api_key、base_url 等
    api_key=api_key,		# api_key DeepSeek API 密钥 应从环境变量读取，避免硬编码
    base_url=base_url,	 # base_url API 端点（如果用的是第三方的，url换成对应的即可） 必须包含 /v1 路径
    temperature=0.8,  # 控制生成文本的随机性
    max_tokens=512,  # 控制生成文本的最大 token 数
)
print("mode:", type(model))
# 调用
responses = model.invoke("中国的新能源汽车品牌有哪些？举三个最著名的，不用介绍")
print(responses.content)
```



**2.【ChatOpenAI】方式**

> ChatOpenAI默认调用openai大模型，但是可以指定调用模型，如果指定deepSeek，即可调用deepSeek模型。

```python
from langchain_openai import ChatOpenAI
import os

api_key = os.getenv('DEEPSEEK_API_KEY')
base_url = "https://api.deepseek.com/"
ll1 = ChatOpenAI(
    model="deepseek-chat",
    api_key=api_key,
    base_url=base_url
)
print("type:", type(ll1))
responses = ll1.invoke("北京最著名的景点有哪些，举三个？不用具体介绍")
print(responses.content)
```





**3.【ChatDeepSeek】方式**

> langchain框架 为调用DeepSeek专门写的API，需要安装对应环境即：pip3 install langchain-deepseek

```python
import os
from langchain_deepseek import ChatDeepSeek

api_key = os.getenv('DEEPSEEK_API_KEY')
llm1 = ChatDeepSeek(model="deepseek-chat")

# 调用
print("type:", type(llm1))
result = llm1.invoke("北京最著名的CBD在哪，具体是哪个？不需要介绍")
print(result.content)

```







## 对话链构建

###  提示词模板

```python
prompt = ChatPromptTemplate.from_template("{input}")

```

进阶使用：

```python
from langchain_core.prompts import (
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate
)

prompt = ChatPromptTemplate.from_messages([
    SystemMessagePromptTemplate.from_template("你是一个{role}"),
    HumanMessagePromptTemplate.from_template("{input}")
])
```



### 链式组合

```python
chain = prompt | llm	# 通过管道符，将prompt的输出作为输入给llm
```



```python
import os

from langchain_core.prompts import ChatPromptTemplate, PromptTemplate
from langchain_openai import ChatOpenAI  # 虽然叫 OpenAI，但可兼容 DeepSeek


# 创建提示模板
prompt = PromptTemplate(
    input_variables=["product"],
    template="为{product}写一个创意广告文案:",
)

# 注意：这里使用 ChatOpenAI 但指向 DeepSeek 的 API
llm = ChatOpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",  # 注意 /v1 路径
    model="deepseek-chat"
)

# 新版链式调用
# prompt = ChatPromptTemplate.from_template("{input}")
chain = prompt | llm  # 使用管道操作符替代旧版 LLMChain

resp = chain.invoke({"product": "堡垒机"})
print(resp.content)

```



`ChatPromptTemplate`是提示词模板：

```python
import os

from langchain_core.prompts import ChatPromptTemplate, PromptTemplate
from langchain_openai import ChatOpenAI  # 虽然叫 OpenAI，但可兼容 DeepSeek



# 注意：这里使用 ChatOpenAI 但指向 DeepSeek 的 API
llm = ChatOpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",  # 注意 /v1 路径
    model="deepseek-chat"
)

# 新版链式调用
prompt = ChatPromptTemplate.from_messages([
    ("system", "为{product}写一个创意广告文案"),    # 系统提示词
    # ("user", "请用 2-3 句解释 {topic}，并给出应用场景。")
])



chain = prompt | llm  # 使用管道操作符替代旧版 LLMChain

resp = chain.invoke({"product": "堡垒机"})
print(resp.content)

```



```python
import os

from langchain_core.prompts import ChatPromptTemplate, PromptTemplate
from langchain_openai import ChatOpenAI  # 虽然叫 OpenAI，但可兼容 DeepSeek



# 注意：这里使用 ChatOpenAI 但指向 DeepSeek 的 API
llm = ChatOpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",  # 注意 /v1 路径
    model="deepseek-chat"
)
# 步骤1：摘要
prompt1 = ChatPromptTemplate.from_template("用一句话总结：{text}")
chain1 = prompt1 | llm

# 步骤2：翻译
prompt2 = ChatPromptTemplate.from_template("请把下面的句子翻译成英文：{summary}")
chain2 = prompt2 | llm

# 简单串联（伪代码风格）
res1 = chain1.invoke({"text": "哈哈哈哈，你是谁你是谁哈哈哈"})
summary = res1.content if hasattr(res1,'content') else res1['text']
res2 = chain2.invoke({"summary": summary})
print(res2.content)

```

> 提示：LangChain 的 `|` 操作会把前一个组件的输出自动传入下一个组件。不同版本输出字段可能是 `.content` 或 `.text`，注意检查。





扩展链示例（带记忆）：

```python
from langchain_core.runnables import RunnablePassthrough

memory = ConversationBufferMemory()
chain = (
    RunnablePassthrough.assign(
        history=memory.load_memory_variables
    ) 
    | prompt 
    | llm
)
```





## 提示词工程

核心知识点:

- system role 用来设定“身份/约束/输出格式”
- user role 放用户内容/问题
- template 支持占位符（如 `{topic}`）
- few-shot：可以把示例对话放到 template 帮助模型模仿风格

```python

from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
import os

# 构建OpenAI客户端
llm = ChatOpenAI(
	model="deepseek-chat",
	openai_api_key=os.getenv("DEEPSEEK_API_KEY"),
	openai_api_base="https://api.deepseek.com/",
)

# 提示词
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个精确、简洁的技术助理，回答时只给结果，不要多余话。"),    # 系统提示词
    ("user", "请用 2-3 句解释 {topic}，并给出应用场景。")
])

chain = prompt | llm    # 链式结构

resp = chain.invoke({"topic": "企业蓝军"})  # 调用
print(resp.content)

```







```python
import datetime
from langchain.tools import tool
from langchain_openai import ChatOpenAI
# 构建OpenAI客户端
llm = ChatOpenAI(
	model="deepseek-v3",
	openai_api_key="sk-xxxxxx",
	openai_api_base="https://api.deepseek.com/"
)


### 1.简单调用
# 定义⼯具 注意要添加注释
# @tool
# def add(a: int, b: int) -> int:
# 	"""Adds a and b."""
# 	return a + b
# print(add.invoke({"a":11,"b":21}))



### 2.LangChain调用工具
from langchain.tools.render import render_text_description
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser

# 定义⼯具 注意要添加注释
@tool
def add(a: int, b: int) -> int:
    """Adds a and b."""
    return a + b


@tool
def get_current_date():
    """获取今天⽇期"""
    return datetime.datetime.today().strftime("%Y-%m-%d")


tools = [add, get_current_date]

# 构建工具条件和调用描述
rendered_tools = render_text_description(tools)
print("rendered_tools:", rendered_tools)

# 构建系统提示词
system_prompt = f"""
You are an assistant that has access to the following set of tools. Here are the names and descriptions for each tool:
{rendered_tools}
Given the user input, return the name and input of the tool to use. Return your response as a JSON blob with 'name' and 'arguments' keys."""

# 构建输入提示词模板
prompt = ChatPromptTemplate.from_messages(
    [("system", system_prompt), ("user", "{input}")]
)

# 构建chain以及最后格式化输出
chain = prompt | llm
print(chain.invoke({"input": "what's 11 + 12"}))  # 输出所有结果
tool_map = {tool.name: tool for tool in tools}


def tools_call(model_output):
    chosen_tool = tool_map[model_output["name"]]
    return chosen_tool.invoke(model_output["arguments"])


# 在链条后面添加JsonOutputParser()和tools_call用来限制输出内容，只输出工具的结果
# 链式组合
chain = prompt | llm | JsonOutputParser() | tools_call
print(chain.invoke({"input": "what's 11 + 33"}))
print(chain.invoke({"input": "what's the date today?"}))

```







强制输出结构：用 请返回 JSON：{...} 并配合输出解析器







## 对话与 Memory

\- 让模型记住对话上下文
\- 使用 ConversationBufferMemory
\- 构建智能助手



```python
import os

from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferMemory
from langchain_openai import ChatOpenAI


# 构建OpenAI客户端
llm = ChatOpenAI(
	model="deepseek-chat",
	openai_api_key=os.getenv("DEEPSEEK_API_KEY"),
	openai_api_base="https://api.deepseek.com/"
)

# 创建对话记忆
memory = ConversationBufferMemory()

# 创建对话链，将模型和记忆关联
conversation = ConversationChain(llm=llm, memory=memory)

# 进行多轮对话
print("=== 对话过程 ===")
response1 = conversation.invoke("你好")
print(f"AI: {response1['response']}\n")

response2 = conversation.invoke("我叫小明")
print(f"AI: {response2['response']}\n")

response3 = conversation.invoke("我叫什么名字？")	# # 应该能回忆出“小明”
print(f"AI: {response3['response']}\n")

# 查看记忆中存储的对话历史
print("=== 记忆中的对话历史 ===")
print(memory.load_memory_variables({}))

```



SummaryMemory（用于长会话）：当内存变大，用摘要策略把历史压缩成短摘要，从而节省 token。









## 工具(tools)



### 方法一：`@tool` 装饰器（同步函数）



定义工具：

```python
from langchain.tools import tool

@tool
def add(a: int, b: int) -> int:
    """加法运算"""
    return a + b

tools = [add]

```





绑定LLM和工具：

```python
llm_with_tools = llm.bind_tools(tools)

resp = llm_with_tools.invoke("请帮我算一下 123 + 456")
print(resp)

```



Agent（智能体，自动决策调用工具）:

```python
from langchain.agents import initialize_agent, AgentType

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

resp = agent.invoke("帮我计算 12 + 35")
print(resp["output"])

```





```python
"""
阶段 5：工具调用（Tools + Agent）

目标：了解如何把外部函数注册为工具（tool），并让 model 决定调用。

知识点：
- 什么是工具（外部函数）
- @tool 装饰器注册函数
- Agent 根据需求调用工具
- 让 LLM 自动决定调用
"""
import os
from langchain_openai import ChatOpenAI
from langchain.agents import initialize_agent, AgentType, tool


# 构建OpenAI客户端
llm = ChatOpenAI(
	model="deepseek-chat",
	openai_api_key=os.getenv("DEEPSEEK_API_KEY"),
	openai_api_base="https://api.deepseek.com/v1"
)

# 定义工具
# 方法一：@tool 装饰器（同步函数）
@tool
def get_weather(city: str) -> str:
    """查询天气demo"""
    return f"{city}明天晴，25度"

@tool
def get_assets(domain: str) -> str:
    """查询域名资产demo"""
    import socket
    try:
        ip = socket.gethostbyname(domain)
        return f"资产信息：{domain} -> {ip}, 1.1.1.1"
    except Exception as e:
        return f"解析失败: {e}"


tools = [get_weather, get_assets]

# 注意：initialize_agent 会把工具的 docstring / description 传给模型，让模型学会如何调用。
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

# 让模型自己决定调用哪个工具
resp = agent.invoke("帮我查询baidu.com的资产")
print(resp["output"])


resp2 = agent.invoke("帮我查询北京天气")
print(resp2["output"])

```







### 方法二：异步工具（适用于 aiohttp / async IO）

如果需要做网络 IO，最好提供 async 函数并使用 MCP或 langchain 的 async agent 支持。



```python
import asyncio
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain.agents import tool
from langchain_core.output_parsers import JsonOutputParser


# 1. 定义异步工具
@tool
async def get_weather(location: str) -> str:
    """根据地点获取天气"""
    await asyncio.sleep(1)  # 模拟耗时IO
    return f"{location} 今天天气晴，26℃"


@tool
async def get_time(city: str) -> str:
    """根据城市获取时间"""
    await asyncio.sleep(1)  # 模拟耗时IO
    return f"{city} 当前时间：2025-09-20 14:30"


# 2. 初始化模型和 prompt
api_key = "sk-xxxxx"
base_url = "https://api.deepseek.com/"
llm = ChatOpenAI(
    model="deepseek-chat",
    api_key=api_key,
    base_url=base_url,
    temperature=0
)


prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个智能助手。输出 JSON，包含需要调用的工具及参数。"),
    ("human", "{question}")
])

parser = JsonOutputParser()

# 3. 构建 chain (让模型输出工具调用参数)
chain = prompt | llm | parser       # 只让大模型输出一个 JSON，说明需要调用哪些工具（例如 "tool": "get_weather", "args": {...}）。


# 4. 异步调用
async def main():
    question = "请告诉我北京的天气和上海的时间"

    # 先让 LLM 决定要调用什么工具
    tool_plan = await chain.ainvoke({"question": question})
    print("模型输出:", tool_plan)

    # 解析工具调用并执行
    results = []
    for call in tool_plan.get("calls", []):
        tool_name = call["tool"]
        args = call["args"]

        if tool_name == "get_weather":
            result = await get_weather.ainvoke(args["location"])    # 调用，工具的执行是在这里显式写的，而不是在 chain 里面自动完成的。
        elif tool_name == "get_time":
            result = await get_time.ainvoke(args["city"])
        else:
            result = f"未知工具: {tool_name}"

        results.append({tool_name: result})

    print("工具执行结果:", results)


if __name__ == "__main__":
    asyncio.run(main())

```



使用agent自动调用工具：
```python
import asyncio
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain.agents import tool
from langchain_core.output_parsers import JsonOutputParser

import os
import asyncio
from langchain_openai import ChatOpenAI
from langchain.tools import tool
from langchain.agents import initialize_agent, AgentType

# ========== 1. 配置 DeepSeek ==========
api_key = "sk-a338af1dfb72401c98ad5c7c7cfdf760"
base_url = "https://api.deepseek.com/"
llm = ChatOpenAI(
    model="deepseek-chat",
    api_key=api_key,
    base_url=base_url
)

# ========== 2. 定义工具 ==========
# 定义异步工具
@tool
async def get_weather(city: str) -> str:
    """查询指定城市的天气（异步）"""
    await asyncio.sleep(1)  # 模拟IO操作
    return f"{city} 明天是晴天，25度。"

@tool
async def get_assets(domain: str) -> str:
    """查询域名的资产信息（异步）"""
    await asyncio.sleep(1)  # 模拟IO操作
    return f"{domain} 共有 5 个子域名，部署在 3 台服务器上。"

tools = [get_weather, get_assets]

# ========== 3. 初始化 Agent ==========
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.OPENAI_FUNCTIONS,   # ✅ 使用函数调用式 Agent
    verbose=True
)

# ========== 4. 运行示例 ==========
async def main():
    print(">>> 询问天气：")
    result1 = await agent.ainvoke({"input": "查询北京明天的天气"})
    print(result1["output"])

    print("\n>>> 资产信息：")
    result2 = await agent.ainvoke({"input": "帮我查询 baidu.com 的资产"})
    print(result2["output"])


if __name__ == "__main__":
    asyncio.run(main())


```





## MCP

`

mcp_server.py

```python
# math_server.py
from mcp.server.fastmcp import FastMCP
import logging

# 配置日志记录器
logging.basicConfig(
    level=logging.INFO,  # 设置日志级别为 INFO
    format="%(asctime)s - %(levelname)s - %(message)s"  # 日志格式
)
logger = logging.getLogger(__name__)

# 创建 FastMCP 实例
mcp = FastMCP("Math")


@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    logger.info("The add method is called: a=%d, b=%d", a, b)  # 记录加法调用日志
    return a + b


@mcp.tool()
def multiply(a: int, b: int) -> int:
    """Multiply two numbers"""
    logger.info("The multiply method is called: a=%d, b=%d", a, b)  # 记录乘法调用日志
    return a * b


if __name__ == "__main__":
    logger.info("Start math server through MCP")  # 记录服务启动日志
    mcp.run(transport="stdio")  # 启动服务并使用标准输入输出通信
```



mcp_client.py

```python
import asyncio
from mcp import ClientSession, StdioServerParameters
from mcp.client.stdio import stdio_client
from langchain_mcp_adapters.tools import load_mcp_tools
from langgraph.prebuilt import create_react_agent
from langchain_deepseek import ChatDeepSeek

# 初始化 DeepSeek 大模型客户端
llm = ChatDeepSeek(
    model="deepseek-chat",  # 指定 DeepSeek 的模型名称
    api_key="sk-xxxxxx"  # 替换为您自己的 DeepSeek API 密钥
)


# 解析并输出结果
def print_optimized_result(agent_response):
    """
    解析代理响应并输出优化后的结果。
    :param agent_response: 代理返回的完整响应
    """
    messages = agent_response.get("messages", [])
    steps = []  # 用于记录计算步骤
    final_answer = None  # 最终答案

    for message in messages:
        if hasattr(message, "additional_kwargs") and "tool_calls" in message.additional_kwargs:
            # 提取工具调用信息
            tool_calls = message.additional_kwargs["tool_calls"]
            for tool_call in tool_calls:
                tool_name = tool_call["function"]["name"]
                tool_args = tool_call["function"]["arguments"]
                steps.append(f"调用工具: {tool_name}({tool_args})")
        elif message.type == "tool":
            # 提取工具执行结果
            tool_name = message.name
            tool_result = message.content
            steps.append(f"{tool_name} 的结果是: {tool_result}")
        elif message.type == "ai":
            # 提取最终答案
            final_answer = message.content

    # 打印优化后的结果
    print("\n计算过程:")
    for step in steps:
        print(f"- {step}")
    if final_answer:
        print(f"\n最终答案: {final_answer}")


# 定义异步主函数
async def main():
    # 创建服务器参数
    server_params = StdioServerParameters(
        command="python",
        # 确保更新为 math_server.py 文件路径
        args=["./math_server.py"],
    )

    # 使用 stdio_client 进行连接
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(read, write) as session:
            # 初始化连接
            await session.initialize()
            print("成功连接到 Math 服务")

            # 加载工具
            tools = await load_mcp_tools(session)
            print("加载工具完成: ", [tool.name for tool in tools])

            # 创建代理
            agent = create_react_agent(llm, tools)

            # 循环接收用户输入
            while True:
                try:
                    # 提示用户输入问题
                    user_input = input("\n请输入您的问题（或输入 'exit' 退出）：")
                    if user_input.lower() == "exit":
                        print("感谢使用！再见！")
                        break

                    # 调用代理处理问题
                    agent_response = await agent.ainvoke({"messages": user_input})

                    # 打印完整响应（调试用）
                    # print("\n完整响应:", agent_response)

                    # 调用抽取的方法处理输出结果
                    print_optimized_result(agent_response)

                except Exception as e:
                    print(f"发生错误：{e}")
                    continue


# 使用 asyncio 运行异步主函数
if __name__ == "__main__":
    asyncio.run(main())
```







## 检索增强生成（RAG）



**基本流程：**

1. 文档分段（chunk） → 2. 生成 embedding → 3. 建立向量索引并持久化 → 4. 检索 top-k → 5. 将检索结果拼到 prompt 给 LLM



```python
# rag_deepseek.py
import os
from langchain_community.document_loaders import TextLoader, PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.embeddings import HuggingFaceEmbeddings
from langchain_community.vectorstores import FAISS
from langchain.prompts import PromptTemplate
from langchain.chains import RetrievalQA

# ============ 配置 API ============
os.environ["OPENAI_API_KEY"] = os.getenv("DEEPSEEK_API_KEY")
os.environ["OPENAI_API_BASE"] = "https://api.deepseek.com/v1"  # DeepSeek 兼容 OpenAI API 格式


# ============ 1. 加载文档 ============
def load_documents(doc_dir="./files"):
    docs = []
    for filename in os.listdir(doc_dir):
        filepath = os.path.join(doc_dir, filename)
        if filename.endswith(".txt") or filename.endswith(".md"):
            loader = TextLoader(filepath, encoding="utf-8")
        elif filename.endswith(".pdf"):
            loader = PyPDFLoader(filepath)
        else:
            print(f"跳过不支持的文件: {filename}")
            continue
        docs.extend(loader.load())
    return docs


# ============ 2. 文本切分 ============
def split_documents(docs):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=500,
        chunk_overlap=100,
        separators=["\n\n", "\n", "。", " ", ""]
    )
    return splitter.split_documents(docs)


# ============ 3. 向量化并存储 ============
def build_vectorstore(chunks):
    # embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
    # vectorstore = FAISS.from_documents(chunks, embeddings)
    # return vectorstore
    # 使用 HuggingFace embedding 模型
    # HuggingFace embeddings 默认会把文本转成向量，本地运行，不需要 API key，也不会产生费用。
    # 流程：1.embedding → 本地 HuggingFace 模型（免费，快）
    #      2.RAG 生成 → DeepSeek 模型（API 调用）。
    embeddings = HuggingFaceEmbeddings(model_name="sentence-transformers/all-MiniLM-L6-v2")
    vectorstore = FAISS.from_documents(chunks, embeddings)
    return vectorstore


# ============ 4. 构建 RAG 问答链 ============
def build_qa_chain(vectorstore):
    retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": 3})

    prompt_template = """
    你是一个专业的问答助手。
    根据以下已知信息回答问题，如果无法从中找到答案，请说“我不知道”。
    已知信息:
    {context}

    问题:
    {question}

    答案:"""

    prompt = PromptTemplate(template=prompt_template, input_variables=["context", "question"])

    llm = ChatOpenAI(
        model="deepseek-chat",
        temperature=0,  # 调低，更准确
    )

    # RetrievalQA 是 LangChain 提供的 RAG 问答封装。
    """
    它的工作流程大概是：
    1.拿到用户问题
    2.用 retriever 从向量数据库里找相关文档
    3.把文档塞进大模型（llm）
    4.用 prompt 模板指导模型生成答案
    """
    chain = RetrievalQA.from_chain_type(
        llm=llm,
        retriever=retriever,
        chain_type="stuff",
        chain_type_kwargs={"prompt": prompt}
    )
    return chain


# ============ 主程序 ============
if __name__ == "__main__":
    print(">>> 加载文档中...")
    docs = load_documents("./files")

    print(">>> 文本切分中...")
    chunks = split_documents(docs)

    print(">>> 构建向量数据库...")
    vectorstore = build_vectorstore(chunks)

    print(">>> 构建 RAG 问答系统...")
    qa_chain = build_qa_chain(vectorstore)

    print(">>> 系统已启动，可以提问啦！输入 exit 退出。\n")
    while True:
        query = input("用户: ")
        if query.lower() in ["exit", "quit", "q"]:
            break
        result = qa_chain.invoke({"query": query})
        print("助手:", result["result"])

```







注意：
Chunk size：尽量不要过大（以 embedding 模型和 downstream LLM 的上下文窗口为准）。
Embedding 模型选择影响检索质量：小模型快、便宜；大模型通常更准。
若要做会话式检索（Conversation + RAG），可使用 ConversationalRetrievalChain 并把 conversation memory 作为 context。



## LangGraph和LangChain区别



|     | LangChain | LangGraph |
| --- | --- | --- |
| **主要目的** | 提供构建LLM应用的标准化组件和链式流程 | 专门用于构建复杂、有状态的LLM工作流 |
| **抽象级别** | 高级API，简化常见任务 | 低级控制，**灵活编排复杂逻辑** |
| **典型用例** | 快速搭建RAG、简单对话系统 | 多角色协作、**复杂决策流程** |





架构设计对比

**LangChain**：

* 基于"链(Chain)"的概念
* 线性执行：A → B → C
* 内置标准化组件：
  
    ```python
    chain = prompt | llm | output_parser
    ```
    

**LangGraph**：

* 基于"图(Graph)"的概念
* 支持任意拓扑结构
  
* 节点自由组合：
  
    ```python
    workflow.add_node("node1", func1)
    workflow.add_node("node2", func2)
    workflow.add_edges(["node1", "node2"])
    
    ```





典型场景：

**适合LangChain的情况**：

* 快速搭建标准RAG流程
* 简单问答系统
* 需要开箱即用的常见模板

**适合LangGraph的情况**：

* 多智能体协作系统
* 需要条件分支的复杂对话
  
    ```python
    def should_continue(state):
        return state["step"] < 5
    
    workflow.add_conditional_edges(
        "main_node",
        should_continue,
        {True: "main_node", False: END}
    )
    
    ```
    
* 需要循环执行的任务（如自动调试）
* 涉及多个决策阶段的业务流程



**LangChain典型代码**：

```python
# 线性链式结构
chain = (
    load_documents 
    | split_text 
    | embed 
    | store_vector
    | query_chain
)
```

**LangGraph典型代码**：

```python
# 图结构工作流
# 1.初始化一个 有向图 (workflow graph) 构建器（每个节点是一个可执行函数（step），每条边是节点之间的执行路径。）
builder = Graph()
# 2. 添加节点（相当于在图里画了三个方块（节点），每个节点执行一个 Python 函数。）
builder.add_node("preprocess", preprocess_input)    # 预处理
builder.add_node("retrieve", retrieve_docs)         # 文档检索
builder.add_node("generate", generate_response)     # 大模型生成答案
# 3.条件边（Conditional Edges）
builder.add_conditional_edges(
    "generate",
    lambda x: "需要修正" in x["output"],
    {"True": "correct", "False": END}
)

"""
解释：
从 generate 节点 出发，不是固定走某一条边，而是要根据条件判断走不同路径。

lambda x: "需要修正" in x["output"]
    这里的 x 是 generate 节点的输出（通常是 dict）。
    如果输出里包含 "需要修正"，返回 True；否则 False。

{"True": "correct", "False": END}
    如果条件成立（True） → 走到 "correct" 节点。
    如果不成立（False） → 走到 END，即流程结束"""

```

这种结构比 LangChain chain 更灵活，因为你可以像画流程图一样控制分支、循环、回退。



二者

* **不是替代关系**：LangGraph通常与LangChain配合使用

* **常见组合模式**：
  
    1.  用LangChain处理标准化组件（嵌入、检索等）
    2.  用LangGraph编排这些组件的执行逻辑
    
    ```python
    from langchain_community.retrievers import BM25Retriever
    from langgraph.graph import Graph
    
    # LangChain组件
    retriever = BM25Retriever.from_documents(docs)
    
    # LangGraph节点
    def retrieve_node(state):
        docs = retriever.invoke(state["query"])
        return {"docs": docs}
    
    ```



何时选择哪种工具？

* **选择LangChain当**：
  
    * 你需要快速实现标准模式
    * 任务可以表示为线性流程
    * 不想处理底层状态管理
* **选择LangGraph当**：
  
    * 需要if-then-else等控制流
    * 构建多智能体系统
    * 需要精细控制执行流程
    
    ```python
    # 例如带审核的工作流
    def content_filter(state):
        return "敏感词" not in state["draft"]
    
    workflow.add_conditional_edges(
        "generate_draft",
        content_filter,
        {True: "publish", False: "reject"}
    )
    
    ```
    

两者结合使用的完整示例：

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langgraph.graph import Graph

# LangChain组件
prompt = ChatPromptTemplate.from_template("回答关于{topic}的问题")
llm = ChatOpenAI()
chain = prompt | llm

# LangGraph工作流
workflow = Graph()
workflow.add_node("generate", lambda x: chain.invoke(x))
workflow.add_node("review", human_review_func)
workflow.add_conditional_edges(
    "generate",
    needs_review,
    {"True": "review", "False": END}
)

```

## 参考文章

https://blog.csdn.net/java_jar/article/details/148795446   
https://devpress.csdn.net/aibjcy/683ffbdd606a8318e85b31d0.html  
https://blog.csdn.net/wxz258/article/details/147021450    
https://blog.csdn.net/java_jar/article/details/148869489  
https://blog.csdn.net/java_jar/article/details/148841607



















---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/langchain%E5%AD%A6%E4%B9%A0/  

