---
title: 使用LangChain接入DeepSeek大模型
categories:
  - Blog
tags:
  - AI
  - 大模型
  - DeepSeek
  - OpenAI
  - LangChain
  - LLM
  - Jupyter Notebook
---

LangChain的默认模型是OpenAI，本篇将介绍如何使用LangChain支持DeepSeek的deepseek-chat模型。

## LangChain

LangChain这个框架是用来开发大模型的应用程序，可以简化LLM应用程序的构建。

中文介绍可以参考这个网站，https://python.langchain.ac.cn/docs/introduction/ ，

其对应的官方网站尾，https://python.langchain.com/docs/introduction/ 。

LangChain的架构图：

![](https://python.langchain.ac.cn/svg/langchain_stack_112024.svg)

## 定义DeepSeek模型

这里采用API接入的方式，封装一个应用LangChain调用DeepSeek LLM API的工具。

源代码如下：

```python
from langchain_core.language_models.llms import BaseLLM
from langchain_core.outputs import LLMResult
from typing import Optional, List, Dict, Any, Iterator
import requests


class DeepSeekLLM(BaseLLM):
    """封装DeepSeek LLM"""
    api_key: str  # DeepSeek API Key
    model: str = "deepseek-chat"  # 默认模型
    temperature: float = 0.7  # 温度参数
    base_url: str = "https://api.deepseek.com/v1"  # DeepSeek API 地址

    def _generate(
            self,
            prompts: List[str],
            stop: Optional[List[str]] = None,
            run_manager: Optional[Any] = None,
            **kwargs: Any,
    ) -> LLMResult:
        """实现必须的抽象方法"""
        responses = []
        for prompt in prompts:
            response = self._call(prompt, stop=stop, **kwargs)
            responses.append(response)
        return LLMResult(generations=[[{"text": r}] for r in responses])

    def _call(
            self,
            prompt: str,
            stop: Optional[List[str]] = None,
            **kwargs: Any,
    ) -> str:
        """调用DeepSeek API"""
        url = f"{self.base_url}/chat/completions"
        headers = {"Authorization": f"Bearer {self.api_key}"}
        data = {
            "model": self.model,
            "messages": [{"role": "user", "content": prompt}],
            "temperature": self.temperature,
            **kwargs
        }
        if stop:
            data["stop"] = stop

        response = requests.post(url, json=data, headers=headers)
        response.raise_for_status()
        return response.json()["choices"][0]["message"]["content"]

    @property
    def _llm_type(self) -> str:
        return "deepseek"
```

后续在需要使用的地方，进行`import`即可。

```python
from ai.DeepSeekLLM import DeepSeekLLM
```

## 直接调用

执行以下代码，将通过API直接访问DeepSeek的chat模型。

```python
from openai import OpenAI
# 从env文件中加载api_key
from dotenv import load_dotenv
import os
load_dotenv()
# print(os.getenv("deepseek_api_key"))
client = OpenAI(
    api_key=os.getenv("deepseek_api_key"),
    base_url="https://api.deepseek.com"
)

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "system", "content": "who are you "},
        {"role": "user", "content": "Hello"},
    ],
    stream=False
)

print('模型列表：' + str(client.models.list().data))

print('开场白：' + response.choices[0].message.content)
```

通过调试，得到模型列表为：

```sh
模型列表：[Model(id='deepseek-chat', created=None, object='model', owned_by='deepseek'), Model(id='deepseek-reasoner', created=None, object='model', owned_by='deepseek')]
```

通过Model的id可以看到有2个可以使用的模型，deepseek-chat和deepseek-reasoner，即对话模型和推理模型。

返回的开场白为：

```sh
开场白：Hello! I'm an AI assistant here to help you with questions, information, or just to chat. How can I assist you today? 😊
```

可以看到，已经可以正常的访问DeepSeek大模型啦。接下来将进一步尝试其他的调用方式。

## function calling 模式

函数调用（Function Calling） 是一种让大语言模型（LLM）能够与外部工具交互的关键技术。通过这种方式，LLM可以扩展其能力，例如获取实时数据或执行特定任务。函数调用的核心在于，LLM根据用户输入，判断是否需要调用某个函数，并以结构化的方式输出调用信息。

代码示例：

```python
from openai import OpenAI
from dotenv import load_dotenv
import os

load_dotenv()

client = OpenAI(api_key=os.getenv("deepseek_api_key"), base_url="https://api.deepseek.com")

def send_messages(messages):
    response = client.chat.completions.create(
        model="deepseek-chat",
        messages=messages,
        tools=tools
    )
    return response.choices[0].message


tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get weather of an location, the user shoud supply a location first",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "The city and state, e.g. San Francisco, CA",
                    }
                },
                "required": ["location"]
            },
        }
    },
]

messages = [{"role": "user", "content": "How's the weather in Hangzhou?"}]
message = send_messages(messages)
print(f"User>\t {messages[0]['content']}")

tool = message.tool_calls[0]
messages.append(message)

messages.append({"role": "tool", "tool_call_id": tool.id, "content": "25℃"})
message = send_messages(messages)
print(f"Model>\t {message.content}")
```

上面的代码中，定义了一个获取天气的函数（get_weather），通过description字段描述大模型应该如何响应，并且增加了parameters指定返回的数据格式。

跑起来看看：

```sh
User>	 How's the weather in Hangzhou?
Model>	 The current weather in Hangzhou is 25°C (77°F). It's a pleasant temperature - not too hot and not too cold.
```

通过function calling模式，可以从大模型中得到更加精准的结果，避免那些长篇大论。

