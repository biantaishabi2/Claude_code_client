# ClaudeClient

一个简洁的Python封装，用于与Claude命令行工具进行交互。

## 简介

`ClaudeClient` 是一个基于 `pexpect` 的工具，封装了 Claude 命令行工具的交互过程，使您能够以编程方式发送请求并获取 Claude 的回复，无需手动操作命令行界面。

这个客户端支持：
- 自动管理 Claude 进程的生命周期
- 处理所有状态转换和交互确认
- 发送自定义请求并获取响应
- 自动处理超时和异常情况

## 安装要求

- Python 3.6+
- pexpect 库
- 已安装 Claude 命令行工具

## 使用方法

### 基本用法

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

### 命令行使用

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

### 示例文件

我们提供了一个完整的示例文件 `claude_client_example.py`，其中包含多种使用场景：

1. **基本使用示例**: 简单的单次请求处理
2. **连续对话示例**: 在同一会话中发送多个相关请求
3. **超时处理示例**: 如何处理请求超时情况
4. **集成示例**: 将ClaudeClient集成到批处理程序中

运行示例:

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

## API 参考

### ClaudeClient 类

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

## 集成示例代码

以下是如何在实际应用中集成ClaudeClient的示例：

```python
from claude_client import ClaudeClient

def process_questions(questions):
    """批量处理多个问题并返回Claude的回答"""
    client = ClaudeClient(timeout=120)
    results = {}
    
    try:
        if not client.start():
            return {"error": "无法启动Claude客户端"}
        
        for question in questions:
            try:
                # 为每个问题添加前缀以获得更简洁的回答
                prompt = f"请简明扼要回答以下问题: {question}"
                answer = client.send_request(prompt)
                results[question] = answer
            except Exception as e:
                results[question] = f"处理出错: {str(e)}"
    finally:
        client.close()
    
    return results

# 使用示例
if __name__ == "__main__":
    questions = [
        "Python中如何处理JSON数据?",
        "什么是依赖注入?",
        "解释一下Docker的工作原理"
    ]
    
    answers = process_questions(questions)
    
    for question, answer in answers.items():
        print(f"问: {question}")
        print(f"答: {answer}")
        print("-" * 50)
```

## 工作原理

ClaudeClient 通过以下步骤与 Claude 交互:

1. **启动**: 初始化 Claude 命令行进程并处理初始状态
2. **状态管理**: 自动检测和处理 Claude 的不同状态（初始、等待确认、等待输入、工作中）
3. **请求发送**: 将用户输入发送到 Claude 并等待处理
4. **响应处理**: 捕获 Claude 的输出并返回最后 2000 个字符
5. **资源清理**: 在客户端关闭时正确终止 Claude 进程

## 注意事项

- 确保已安装 Claude 命令行工具并且可以在命令行中运行 `claude` 命令
- 客户端返回的是 Claude 最后 2000 个字符的输出，如需完整输出请调整代码
- 默认超时时间为 60 秒，处理复杂请求时可能需要增加此值
- 每次只能运行一个ClaudeClient实例，因为它们都会使用相同的Claude命令行工具

## 依赖

- [pexpect_claude.py](/home/wangbo/document/wangbo/dev/pexpect_claude.py): 基础的 Claude 交互类

## 示例应用场景

1. **作为服务组件**: 将 Claude 集成到更大的应用程序或服务中
2. **批处理请求**: 批量处理多个请求并保存结果
3. **自动化工作流**: 将 Claude 作为自动化工作流的一部分
4. **问答系统**: 构建基于Claude的问答系统
5. **内容生成**: 自动化内容创建和优化过程

## 故障排除

如果遇到问题，请尝试以下步骤：

1. 启用调试模式查看详细日志: `client = ClaudeClient(debug=True)`
2. 增加超时时间处理复杂请求: `client = ClaudeClient(timeout=180)`
3. 确保仅运行一个Claude实例
4. 如果客户端无响应，尝试手动终止后台Claude进程