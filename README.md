# AI 智能体开发实战教程

基于 Python 的 AI 智能体开发从零到实战教学项目。

---

## 📋 环境准备

### 环境配置
- Python 版本：3.12+
- 虚拟环境：`venv`
- LLM 后端：OpenAI 兼容协议（本地模型/云端模型）

### 快速开始
1. 复制环境变量模板并配置
   ```bash
   copy env.example .env
   ```
2. 编辑 `.env` 文件，填入你的 LLM 配置
3. 运行对应练习脚本

---

## 📚 课程目录

| 序号 | 练习目录 | 文件名 | 教学目标 & 知识点 |
|------|---------|--------|-----------------|
| **01** | `practice01/` | `llm_client.py` | ✅ **原生 HTTP 库调用 LLM** |
| | | | 🔹 使用 Python 标准库 `urllib` 发送 HTTP 请求<br>🔹 不依赖 openai SDK，理解底层协议<br>🔹 读取 `.env` 环境变量配置<br>🔹 统计 Token 消耗 & 计算生成速度<br>🔹 完整的错误处理机制 |
| **02** | `practice02/` | `chat_stream.py` | ✅ **流式多轮对话客户端** |
| | | | 🔹 SSE 流式输出实时显示，模拟 ChatGPT 打字效果<br>🔹 终端交互式输入，持续对话<br>🔹 历史消息自动维护上下文<br>🔹 Ctrl+C 优雅退出处理<br>🔹 每轮对话性能统计 |
| | | `tool_chat.py` | ✅ **系统提示词驱动工具调用** |
| | | | 🔹 通过系统提示词教 LLM 输出 JSON 格式调用<br>🔹 6个内置工具函数：文件操作 + 网络访问<br>🔹 list_directory - 列出目录及文件属性<br>🔹 rename_file / delete_file - 文件重命名、删除<br>🔹 create_file / read_file - 创建、读取文件<br>🔹 curl_request - 模拟 curl 访问网页<br>🔹 自动解析并执行工具调用，多轮调用循环 |
| | | `openai_tool_chat.py` | ✅ **OpenAI 标准 Function Calling** |
| | | | 🔹 使用 OpenAI 官方标准协议<br>🔹 结构化 tools 参数传递<br>🔹 并行工具调用支持<br>🔹 自动多轮调用循环 |

---

## 📖 练习详情

---

### ✅ Practice 01: 原生 HTTP 库调用 LLM

**📂 文件位置：** `practice01/llm_client.py`

#### 🎯 教学目标
1. **理解 OpenAI API 协议本质**
   - 认识到 LLM API 本质就是 HTTP POST 请求
   - 理解请求头、请求体的格式
   - 掌握 JSON 数据序列化与反序列化

2. **掌握 Python 标准库使用**
   - `urllib.request` - 标准 HTTP 客户端
   - `json` - JSON 数据处理
   - `time` - 精确计时
   - `pathlib` - 跨平台路径处理

3. **工程化能力培养**
   - 配置文件的读取与管理
   - 异常处理与错误提示
   - 性能指标统计（时间、Token、速度）

#### 💡 核心代码讲解

**1. 环境变量读取**
```python
def load_env():
    # 纯 Python 实现，不依赖 python-dotenv 第三方库
    env_path = Path(__file__).parent.parent / ".env"
    # 逐行解析 key=value 格式
```

**2. 标准 HTTP 请求**
```python
from urllib import request, error

data = json.dumps(payload).encode("utf-8")
req = request.Request(url, data=data, headers=headers, method="POST")

with request.urlopen(req, timeout=timeout) as response:
    response_data = json.loads(response.read().decode("utf-8"))
```

**3. 性能统计**
```python
start_time = time.time()
# ... API 调用 ...
elapsed_time = end_time - start_time
tokens_per_second = completion_tokens / elapsed_time
```

#### 🚀 运行方式

```bash
# 方式1：使用启动脚本（推荐）
cd practice01
run.bat

# 方式2：直接运行
py -3.12 practice01/llm_client.py
```

#### 📊 示例输出
```
============================================================
Practice 01: 原生 HTTP 库调用 LLM
============================================================
[OK] 环境配置加载成功
  API: http://localhost:11434/v1
  模型: qwen:7b

提问: 请用3句话介绍Python语言的特点
------------------------------------------------------------
回答:
Python 是一门语法简洁且易读的脚本语言...

------------------------------------------------------------
[性能统计]
  模型:           qwen:7b
  输入 tokens:    17
  输出 tokens:    108
  总 tokens:      125
  耗时:           2.85 秒
  生成速度:       37.89 tokens/秒
```

---

### ✅ Practice 02: 流式多轮对话客户端

**📂 文件位置：** `practice02/chat_stream.py`

#### 🎯 教学目标
1. **掌握 SSE 流式输出协议**
   - 理解 `stream: True` 参数含义
   - 逐行解析 `data: {...}` 格式
   - 实时 flush 输出到终端，实现打字机效果
   - 处理结束标记 `data: [DONE]`

2. **终端交互式程序设计**
   - `input()` 函数获取用户输入
   - `sys.stdout.flush()` 实时输出
   - `signal` 信号处理捕获 Ctrl+C
   - 无限循环实现持续对话

3. **多轮对话上下文管理**
   - `messages` 列表维护对话历史
   - 自动追加用户提问和助手回答
   - 上下文记忆机制是 Agent 的基础
   - 错误回滚机制（调用失败时弹出历史）

#### 💡 核心代码讲解

**1. 流式输出核心实现**
```python
def print_stream(content):
    sys.stdout.write(content)
    sys.stdout.flush()  # 关键：立即输出，不缓冲

# 逐块解析 SSE 响应
for line in response:
    line = line.decode("utf-8").strip()
    if line.startswith("data: "):
        chunk_data = json.loads(line[6:])
        content = chunk_data["choices"][0]["delta"].get("content", "")
        if content:
            print_stream(content)
```

**2. Ctrl+C 优雅退出**
```python
import signal

def signal_handler(sig, frame):
    print("\n\n[系统] 对话已结束，再见！")
    sys.exit(0)

signal.signal(signal.SIGINT, signal_handler)
```

**3. 上下文自动维护**
```python
messages = []

while True:
    user_input = input("[你] ")
    messages.append({"role": "user", "content": user_input})
    
    response, stats = call_llm_stream(env, messages)

---

### ✅ Practice 02: 系统提示词驱动工具调用

**📂 文件位置：** `practice02/tool_chat.py`

#### 🎯 教学目标
1. **理解原生工具调用原理**
   - 不依赖 OpenAI Function Calling 协议
   - 通过系统提示词教 LLM 输出 JSON 格式
   - 正则表达式智能解析工具调用
   - 结果回传形成完整调用链路

2. **6个内置工具函数**
   - `list_directory` - 列出目录，包含文件大小、修改时间
   - `rename_file` / `delete_file` - 文件重命名、删除
   - `create_file` / `read_file` - 创建并写入、读取内容
   - ✅ **新增** `curl_request` - 模拟 curl 访问网页，自动编码处理

3. **通用 Agent 架构思想**
   - Thought → Action → Observation 循环
   - 最多支持 5 轮连续工具调用
   - 最终用自然语言总结结果

#### 💡 核心代码讲解

**1. 系统提示词工程**
```python
SYSTEM_PROMPT = \"\"\"你拥有以下6个工具可以调用：

【工具6】curl_request - 模拟 curl 访问网页
参数：
  - url: 网页完整 URL

调用规则：请输出严格的JSON格式：
```json
{{"name": "工具名", "parameters": {{"参数名": "参数值"}}}}
```
\"\"\"
```

**2. 智能 JSON 解析器**
```python
def parse_tool_call(content):
    # 1. 先解析 ```json ... ``` 代码块
    pattern = r'```json\s*(\{.*?\})\s*```'
    
    # 2. 失败则尝试直接解析 JSON
    try:
        tool_call = json.loads(content)
    except:
        pass
```

**3. 工具执行与结果反馈**
```python
result = execute_tool(tool_call)

# 结果回传给 LLM，继续思考
messages.append({
    "role": "user",
    "content": f"工具执行结果：{json.dumps(result)}。请基于结果继续处理..."
})
```

#### 🚀 运行方式

```bash
# 运行工具调用对话客户端
cd practice02
run_tool.bat
```

#### 💻 运行示例

```
[系统] 已加载 6 个工具函数:
  1. list_directory  - 列出目录文件及属性
  2. rename_file     - 重命名文件
  3. delete_file     - 删除文件
  4. create_file     - 创建文件并写入内容
  5. read_file       - 读取文件内容
  6. curl_request    - 模拟 curl 访问网页

[你] 帮我访问百度首页，看看标题
[思考中...]

[调用工具] curl_request
  参数: {"url": "https://www.baidu.com"}
  结果: {
    "success": true,
    "status_code": 200,
    "content_length": 2443,
    "content": "<html>...<title>百度一下，你就知道</title>..."
  }

[AI] 百度首页访问成功！页面标题是：百度一下，你就知道
```
    
    messages.append({"role": "assistant", "content": response})
    # messages 列表越来越长，包含完整历史
```

#### 🚀 运行方式

```bash
# 方式1：使用启动脚本（推荐）
cd practice02
run.bat

# 方式2：直接运行
py -3.12 practice02/chat_stream.py
```

#### 💬 示例对话

```
============================================================
Practice 02: 流式对话客户端
============================================================
[OK] 环境配置加载成功
[系统] 欢迎！输入内容开始聊天，按 Ctrl+C 退出
------------------------------------------------------------

[你] 你好，我叫小明

[AI] 你好小明！很高兴认识你。有什么我可以帮助你的吗？

  ─── 本次对话统计 ───
  历史消息数: 1 轮
  输出 tokens: 25
  耗时: 0.85 秒
  生成速度: 29.41 tokens/秒
------------------------------------------------------------

[你] 还记得我叫什么吗？

[AI] 当然记得！你叫小明 😊

  ─── 本次对话统计 ───
  历史消息数: 2 轮
  输出 tokens: 12
  耗时: 0.42 秒
  生成速度: 28.57 tokens/秒
```

> 💡 **重要发现：**
> 第二问中，AI 记住了你叫"小明"！
> 这就是 **上下文记忆** 的力量，也是所有智能体的基础。

---

## ⚙️ 环境配置说明

### env.example 模板变量
| 变量名 | 说明 | 示例值 |
|--------|------|--------|
| `LLM_BASE_URL` | OpenAI 兼容 API 地址 | `http://localhost:11434/v1` |
| `LLM_API_KEY` | API 密钥 | `sk-xxxxxx` |
| `LLM_MODEL` | 模型名称 | `qwen:7b`, `gpt-3.5-turbo` |
| `LLM_TEMPERATURE` | 采样温度（可选） | `0.7` |
| `LLM_MAX_TOKENS` | 最大生成长度（可选） | `2000` |
| `LLM_TIMEOUT` | 请求超时秒数（可选） | `60` |

### 支持的 LLM 后端
| 服务商 | BASE_URL |
|--------|----------|
| **Ollama 本地** | `http://localhost:11434/v1` |
| **LM Studio** | `http://localhost:1234/v1` |
| **OpenAI** | `https://api.openai.com/v1` |
| **通义千问** | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| **豆包** | `https://ark.cn-beijing.volces.com/api/v3` |

---

## 🔧 常见问题

### Q: 为什么用 `py -3.12` 而不是直接 `python`？
A: 系统可能存在多个 Python 版本，`py -3.12` 确保使用正确的版本

### Q: UnicodeEncodeError 编码错误怎么办？
A: Windows GBK 编码问题，已修复使用纯 ASCII 状态标识

### Q: 网络连接超时？
A: 检查你的 LLM 后端是否启动，或者网络代理配置

---

## 📌 项目结构
```
trce/
├── .gitignore              # Git 排除规则
├── .env                    # 你的私有配置（不提交）
├── env.example             # 环境变量模板
├── README.md               # 本文件
├── venv/                   # 虚拟环境
├── practice01/             # 第1课：单次请求
│   ├── llm_client.py       # 非流式HTTP客户端
│   ├── run.bat             # Windows 启动脚本
│   └── run.ps1             # PowerShell 启动脚本
└── practice02/             # 第2课：流式多轮对话
    ├── chat_stream.py      # 流式聊天 + 上下文记忆
    ├── run.bat             # Windows 启动脚本
    └── run.ps1             # PowerShell 启动脚本
```

---

## 🎓 教学理念

**本教程特点：**
1. ✅ 从零开始，不跳过基础
2. ✅ 优先使用标准库，理解原理
3. ✅ 每一行代码都有其教学意义
4. ✅ 从 HTTP 协议到高级封装逐步深入

> 💡 **知其然，知其所以然**
>
> 我们不从 `pip install openai` 开始，
> 而是从 `urllib.request` 开始，
> 让你真正理解 AI 智能体的底层原理。
