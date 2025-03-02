# Claude 客户端工具包

一个全面的Python工具包，用于与Claude命令行工具进行交互。

## 两种客户端实现

本工具包提供了两种Claude客户端实现：

1. **基础版 (ClaudeClient)** - 简单的请求-响应交互
2. **增强版 (EnhancedClaudeClient)** - 支持任务完成分析和自动跟进

## 简介

这套工具封装了与Claude命令行工具的交互过程，使您能够以编程方式发送请求并获取Claude的回复，无需手动操作命令行界面。

### 基础版特点
- 自动管理Claude进程的生命周期
- 处理所有状态转换和交互确认
- 发送自定义请求并获取响应
- 自动处理超时和异常情况

### 增强版额外特点
- 智能判断任务是否完成
- 自动生成跟进问题直到任务完成
- 记录完整对话历史
- 支持对话状态分析

## 安装要求

- Python 3.6+
- pexpect 库
- 已安装Claude命令行工具

## 使用方法

### 基础版用法

```python
from claude_client import ClaudeClient

# 创建客户端实例
client = ClaudeClient()

try:
    # 启动 Claude 进程
    client.start()
    
    # 发送请求并获取响应
    response = client.send_request("请帮我写一个Python函数来计算斐波那契数列")
    print(response)
    
    # 发送另一个请求
    response = client.send_request("现在帮我优化这个函数")
    print(response)
finally:
    # 确保关闭 Claude 进程
    client.close()
```

### 增强版用法

```python
from enhanced_claude_client import EnhancedClaudeClient

# 创建增强版客户端实例
client = EnhancedClaudeClient()

try:
    # 启动 Claude 进程
    client.start()
    
    # 发送请求并等待任务完成（自动跟进）
    response, history = client.send_request(
        "请写一个Python程序，包含以下功能：\n1. 读取CSV文件\n2. 分析数据\n3. 生成报表",
        auto_continue=True,
        max_iterations=3
    )
    
    # 输出最终响应
    print(response)
    
    # 显示完整对话历史
    for i, (question, answer) in enumerate(history):
        print(f"交互 {i+1}:")
        print(f"问: {question}")
        print(f"答: {answer[:100]}...")  # 只显示前100个字符
finally:
    # 确保关闭 Claude 进程
    client.close()
```

### 命令行使用

#### 基础版

```bash
# 直接发送单个请求
python claude_client.py "请帮我编写一个Python脚本来分析CSV文件"

# 启用调试模式
python claude_client.py --debug "请帮我编写一个Python脚本来分析CSV文件"

# 设置超时时间（秒）
python claude_client.py --timeout 120 "请帮我编写一个复杂的机器学习模型"

# 进入交互模式
python claude_client.py
```

#### 增强版

```bash
# 直接发送单个请求（自动跟进）
python enhanced_claude_client.py "请编写一个详细的机器学习模型"

# 禁用自动跟进
python enhanced_claude_client.py --no-auto "请分析这段代码"

# 设置最大交互次数
python enhanced_claude_client.py --max-iter 5 "请详细解释量子计算原理"

# 启用调试模式
python enhanced_claude_client.py --debug "请解释递归函数"
```

### 示例文件

我们提供了完整的示例文件，展示不同使用场景：

#### 基础版示例 (`claude_client_example.py`)

```bash
# 基本示例
python claude_client_example.py basic

# 连续对话示例
python claude_client_example.py conversation

# 超时处理示例
python claude_client_example.py timeout

# 集成示例
python claude_client_example.py integration
```

#### 增强版示例 (`enhanced_claude_example.py`)

```bash
# 基本示例
python enhanced_claude_example.py basic

# 自动继续对话示例
python enhanced_claude_example.py auto

# 交互式使用示例
python enhanced_claude_example.py interactive

# 需要更多信息示例
python enhanced_claude_example.py more_info
```

## API 参考

### 基础版 - ClaudeClient 类

#### 初始化参数

- `debug` (bool, 可选): 启用调试输出。默认为 False。
- `timeout` (int, 可选): 等待 Claude 响应的超时时间（秒）。默认为 60。

#### 方法

- `start()`: 启动 Claude 进程并准备接收请求。
  - 返回: bool - 启动是否成功

- `send_request(request_text)`: 发送请求到 Claude 并等待响应。
  - 参数: `request_text` (str) - 要发送给 Claude 的请求文本
  - 返回: str - Claude 的响应文本（最后 2000 个字符）
  - 异常:
    - `RuntimeError`: Claude 进程不存在或已终止
    - `TimeoutError`: 等待响应超时

- `close()`: 关闭 Claude 进程并释放资源。

### 增强版 - EnhancedClaudeClient 类

#### 初始化参数

- `analyzer` (可选): 任务分析器实例，默认使用规则分析器
- `followup_generator` (可选): 跟进问题生成器实例，默认使用规则生成器
- `debug` (bool, 可选): 启用调试输出。默认为 False。
- `timeout` (int, 可选): 等待 Claude 响应的超时时间（秒）。默认为 60。

#### 方法

- `start()`: 启动 Claude 进程并准备接收请求。
  - 返回: bool - 启动是否成功

- `send_request(request_text, auto_continue=True, max_iterations=3)`: 发送请求并可选择自动继续对话直到任务完成。
  - 参数:
    - `request_text` (str): 请求文本
    - `auto_continue` (bool): 是否自动继续对话直到任务完成
    - `max_iterations` (int): 最大自动交互次数
  - 返回:
    - `response` (str): Claude的最终响应文本
    - `conversation_history` (list): 完整的对话历史记录

- `analyze_completion(response)`: 分析任务是否完成。
  - 参数: `response` (str): Claude的响应文本
  - 返回: 任务状态 ("COMPLETED", "NEEDS_MORE_INFO", "CONTINUE")

- `generate_followup(task_status, conversation_history, last_response)`: 生成跟进问题。
  - 返回: 生成的跟进问题，如果不需要跟进则返回None

- `close()`: 关闭 Claude 进程并释放资源。

## 任务分析器

增强版客户端支持两种任务分析器：

### 规则分析器 (RuleBasedAnalyzer)

使用预定义规则判断任务是否完成：
- 完成指示词: "希望这对你有帮助", "总结一下", 等
- 需要更多信息指示词: "你能提供更多信息吗", "请说明", 等

### LLM分析器 (LLMTaskAnalyzer)

使用另一个LLM来判断任务是否完成，通过分析对话历史和最新响应。

## 实现原理

### 基础版 ClaudeClient

通过以下步骤与 Claude 交互:

1. **启动**: 初始化 Claude 命令行进程并处理初始状态
2. **状态管理**: 自动检测和处理 Claude 的不同状态
3. **请求发送**: 将用户输入发送到 Claude 并等待处理
4. **响应处理**: 捕获 Claude 的输出并返回
5. **资源清理**: 在客户端关闭时正确终止 Claude 进程

### 增强版 EnhancedClaudeClient

1. **基础交互**: 复用基础版的交互机制
2. **任务分析**: 分析Claude的响应，判断任务是否完成
3. **自动跟进**: 根据任务状态生成后续问题
4. **对话记录**: 维护完整的对话历史
5. **完整返回**: 返回最终响应和对话历史

## 工具组件集成

增强版客户端使用了`agent_tools`包中的组件：

- **task_analyzer**: 任务完成状态分析器
- **followup_generator**: 自动跟进问题生成器
- **llm_service**: LLM服务统一接口 (可选使用)
- **parser**: LLM响应解析器 (可选使用)

## 注意事项

- 确保已安装 Claude 命令行工具并且可以在命令行中运行 `claude` 命令
- 客户端返回的是 Claude 最后 2000 个字符的输出，如需完整输出请调整代码
- 默认超时时间为 60 秒，处理复杂请求时可能需要增加此值
- 每次只能运行一个客户端实例，因为它们都会使用相同的Claude命令行工具

## 依赖

- [pexpect_claude.py](/home/wangbo/document/wangbo/dev/pexpect_claude.py): 基础的 Claude 交互类
- [agent_tools](/home/wangbo/document/wangbo/dev/agent_tools/): 增强功能组件集

## 应用场景

1. **作为服务组件**: 将 Claude 集成到更大的应用程序或服务中
2. **批处理请求**: 批量处理多个请求并保存结果
3. **自动化工作流**: 将 Claude 作为自动化工作流的一部分
4. **问答系统**: 构建基于Claude的问答系统
5. **内容生成**: 自动化内容创建和优化过程
6. **智能助手**: 构建能够自动跟进的对话助手
7. **长任务处理**: 自动处理需要多轮交互的复杂任务

## 故障排除

如果遇到问题，请尝试以下步骤：

1. 启用调试模式查看详细日志: `client = EnhancedClaudeClient(debug=True)`
2. 增加超时时间处理复杂请求: `client = EnhancedClaudeClient(timeout=180)`
3. 确保仅运行一个Claude实例
4. 如果客户端无响应，尝试手动终止后台Claude进程