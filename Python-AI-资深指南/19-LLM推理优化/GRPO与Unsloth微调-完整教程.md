# GRPO 与 Unsloth 微调完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

GRPO（Group Relative Policy Optimization）是 DeepSeek-R1 背后的核心训练算法，它让 LLM 学会"思考"而非"背诵"。Unsloth 是当前最高效的 LLM 微调框架，支持在单张 24GB 显卡上完成 GRPO 训练。本教程讲解如何用 GRPO + Unsloth 训练自己的推理模型和 Agent 模型。

### 版本信息
- **Unsloth 版本**：2025.3+
- **支持模型**：Llama 3/4、Qwen 2.5/3、Gemma 3、DeepSeek、Phi-4 等
- **重要程度**：⭐⭐⭐⭐⭐（必学）
- **难度等级**：⭐⭐⭐⭐⭐（高级）
- **预计学习时间**：25-30 小时

### 学习目标
- 理解 GRPO 算法原理及其与 PPO/DPO 的区别
- 掌握 Unsloth 的 LoRA/QLoRA 微调方法
- 学会训练推理模型（Reasoning Model）
- 学会训练 Agent 模型（Agentic Model）
- 掌握奖励函数设计和训练调优

### 前置知识
- PyTorch 基础
- Transformer 架构理解
- LoRA/QLoRA 概念
- Hugging Face 生态基础

---

## 1. 微调技术演进

### 1.1 从 SFT 到 GRPO

```
LLM 微调技术演进路线

SFT（Supervised Fine-Tuning）2023
├── 监督微调，模仿训练数据
├── 优点：简单直接
└── 局限：只能模仿，不能超越训练数据

RLHF / PPO（2023-2024）
├── 人类反馈强化学习
├── 需要训练 Reward Model + Value Model
├── 优点：能学到人类偏好
└── 局限：训练复杂，需要 4 个模型，显存需求大

DPO（Direct Preference Optimization）2024
├── 直接偏好优化，无需 Reward Model
├── 离线训练，更简单
├── 优点：训练简单，效果不错
└── 局限：离线方法，缺乏探索能力

GRPO（Group Relative Policy Optimization）2025-2026 🔥
├── DeepSeek-R1 的核心算法
├── 无需 Value Model，只需 Reward Function
├── 在线训练，保留探索能力
├── 优点：训练高效，推理能力强
└── 当前最先进的 LLM 训练范式
```

### 1.2 GRPO 算法原理

```python
"""
GRPO 核心思想：

1. 采样阶段
   - 对每个问题，让模型生成 G 个候选回答（Group）
   
2. 评分阶段
   - 用 Reward Function 对每个回答打分
   
3. 相对排序
   - 在组内计算相对优势（Advantage）
   - 好的回答得到正优势，差的得到负优势
   
4. 策略更新
   - 增加好回答的生成概率
   - 降低差回答的生成概率

关键优势：
- 不需要 Value Model（PPO 需要）→ 省一半显存
- 不需要 Reward Model（DPO 需要）→ 用规则函数即可
- 在线训练 → 保留探索能力
- 组内相对比较 → 训练更稳定
"""

# GRPO 伪代码
def grpo_step(model, questions, reward_fn, group_size=8):
    for question in questions:
        # 1. 生成一组候选回答
        responses = [model.generate(question) for _ in range(group_size)]
        
        # 2. 计算奖励
        rewards = [reward_fn(question, response) for response in responses]
        
        # 3. 计算组内相对优势
        mean_reward = sum(rewards) / len(rewards)
        std_reward = std(rewards)
        advantages = [(r - mean_reward) / (std_reward + 1e-8) for r in rewards]
        
        # 4. 更新策略（增加好回答的概率，降低差回答的概率）
        loss = -sum(adv * log_prob(response) for adv, response in zip(advantages, responses))
        loss.backward()
        optimizer.step()
```

---

## 2. Unsloth 环境搭建

### 2.1 安装

```bash
# 🔥 推荐：使用 pip 安装（自动检测 CUDA 版本）
pip install unsloth

# 或指定 CUDA 版本
pip install "unsloth[cu124]"  # CUDA 12.4
pip install "unsloth[cu118]"  # CUDA 11.8

# 安装 TRL（Hugging Face 强化学习库）
pip install trl

# 验证安装
python -c "import unsloth; print(unsloth.__version__)"
```

### 2.2 硬件要求

```python
"""
Unsloth 硬件要求（GRPO 训练）：

最低配置：
- GPU：RTX 3060 12GB（7B 模型 QLoRA）
- RAM：16GB
- 存储：50GB

推荐配置：
- GPU：RTX 4090 24GB（7-14B 模型）
- RAM：32GB
- 存储：100GB

高端配置：
- GPU：A100 80GB（70B+ 模型）
- RAM：64GB+

Unsloth 的优势：
- 比标准训练快 2-5x
- 内存使用减少 60-70%
- 7B 模型 GRPO 只需 ~5GB VRAM（QLoRA 4bit）
"""
```

---

## 3. SFT 监督微调（基础）

### 3.1 LoRA 微调

```python
from unsloth import FastLanguageModel
import torch

# 🔥 加载模型（Unsloth 自动优化）
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Llama-3.1-8B-Instruct",
    max_seq_length=2048,
    dtype=None,  # 自动检测
    load_in_4bit=True,  # QLoRA 4bit 量化
)

# 🔥 添加 LoRA 适配器
model = FastLanguageModel.get_peft_model(
    model,
    r=16,                # LoRA rank
    target_modules=[     # 目标模块
        "q_proj", "k_proj", "v_proj", "o_proj",
        "gate_proj", "up_proj", "down_proj"
    ],
    lora_alpha=16,
    lora_dropout=0,      # Unsloth 优化：dropout=0 更快
    bias="none",
    use_gradient_checkpointing="unsloth",  # 🔥 Unsloth 优化
    random_state=42,
)

# 准备训练数据
from datasets import load_dataset

dataset = load_dataset("json", data_files="train_data.jsonl")

# 数据格式（ChatML）
"""
{"messages": [
    {"role": "system", "content": "你是一个Python专家"},
    {"role": "user", "content": "什么是装饰器？"},
    {"role": "assistant", "content": "装饰器是Python中..."}
]}
"""

# 🔥 训练
from trl import SFTTrainer
from transformers import TrainingArguments

trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset["train"],
    args=TrainingArguments(
        output_dir="./output",
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        warmup_steps=5,
        num_train_epochs=3,
        learning_rate=2e-4,
        fp16=not torch.cuda.is_bf16_supported(),
        bf16=torch.cuda.is_bf16_supported(),
        logging_steps=1,
        optim="adamw_8bit",
        seed=42,
    ),
)

trainer.train()

# 保存模型
model.save_pretrained("./my_model_lora")
tokenizer.save_pretrained("./my_model_lora")
```

---

## 4. GRPO 训练推理模型

### 4.1 奖励函数设计

```python
"""
GRPO 的核心：设计好的奖励函数

奖励函数类型：
1. 格式奖励 — 回答是否符合指定格式
2. 正确性奖励 — 答案是否正确
3. 长度奖励 — 控制回答长度
4. 推理过程奖励 — 是否展示了思考过程
"""

import re

def format_reward(response: str) -> float:
    """格式奖励：检查是否使用了 <think>...</think> 格式"""
    has_think = bool(re.search(r'<think>.*?</think>', response, re.DOTALL))
    # 检查 think 标签是否正确闭合
    think_open = response.count('<think>')
    think_close = response.count('</think>')
    
    if has_think and think_open == think_close:
        return 1.0
    elif has_think:
        return 0.5  # 有思考但格式不完整
    return 0.0

def correctness_reward(response: str, expected: str) -> float:
    """正确性奖励：答案是否正确"""
    # 提取最终答案（在 </think> 之后的内容）
    match = re.search(r'</think>\s*(.*)', response, re.DOTALL)
    if match:
        answer = match.group(1).strip()
    else:
        answer = response.strip()
    
    # 精确匹配
    if expected.lower() in answer.lower():
        return 2.0
    
    # 部分匹配
    expected_words = set(expected.lower().split())
    answer_words = set(answer.lower().split())
    overlap = len(expected_words & answer_words) / max(len(expected_words), 1)
    
    return overlap * 1.5

def length_reward(response: str, min_len: int = 50, max_len: int = 2000) -> float:
    """长度奖励：控制回答长度"""
    length = len(response)
    if length < min_len:
        return -0.5  # 太短
    elif length > max_len:
        return -0.5  # 太长
    return 0.0

def combined_reward(response: str, expected: str) -> float:
    """组合奖励函数"""
    return (
        format_reward(response) * 0.3 +
        correctness_reward(response, expected) * 0.6 +
        length_reward(response) * 0.1
    )
```

### 4.2 GRPO 训练

```python
from unsloth import FastLanguageModel
from trl import GRPOConfig, GRPOTrainer
from datasets import load_dataset
import re

# 🔥 加载模型
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Llama-3.1-8B-Instruct",
    max_seq_length=2048,
    load_in_4bit=True,
)

model = FastLanguageModel.get_peft_model(
    model,
    r=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                     "gate_proj", "up_proj", "down_proj"],
    lora_alpha=16,
    use_gradient_checkpointing="unsloth",
)

# 准备数据集
# 格式：{"prompt": "问题", "answer": "期望答案"}
dataset = load_dataset("json", data_files="reasoning_data.jsonl")

# 🔥 定义奖励函数（传给 GRPOTrainer）
def reward_function(completions, prompts, **kwargs):
    """GRPO 奖励函数"""
    rewards = []
    for completion, prompt in zip(completions, prompts):
        # 获取期望答案（从数据集中）
        expected = kwargs.get("expected_answers", [""])[0]
        
        score = 0.0
        
        # 格式奖励
        if '<think>' in completion and '</think>' in completion:
            score += 1.0
        
        # 正确性奖励
        if expected and expected.lower() in completion.lower():
            score += 2.0
        
        rewards.append(score)
    
    return rewards

# 🔥 GRPO 训练配置
training_args = GRPOConfig(
    output_dir="./grpo_output",
    
    # GRPO 核心参数
    num_generations=8,          # 每个问题生成 8 个候选（Group Size）
    max_completion_length=1024, # 最大生成长度
    
    # 训练参数
    per_device_train_batch_size=1,
    gradient_accumulation_steps=4,
    num_train_epochs=3,
    learning_rate=5e-6,         # GRPO 通常用较小的学习率
    
    # 优化
    fp16=True,
    optim="adamw_8bit",
    logging_steps=1,
    save_steps=100,
    
    # KL 散度约束（防止偏离原始模型太远）
    beta=0.04,
)

# 🔥 创建 GRPO Trainer
trainer = GRPOTrainer(
    model=model,
    tokenizer=tokenizer,
    args=training_args,
    train_dataset=dataset["train"],
    reward_funcs=[reward_function],  # 奖励函数列表
)

# 开始训练
trainer.train()

# 保存
model.save_pretrained("./reasoning_model_grpo")
tokenizer.save_pretrained("./reasoning_model_grpo")
```

### 4.3 训练数据准备

```python
"""
GRPO 训练数据格式：

推理模型训练数据示例：
"""

training_examples = [
    {
        "prompt": "计算 17 × 23 的结果",
        "answer": "391"
    },
    {
        "prompt": "一个水池有两个进水管和一个出水管。进水管A每小时注入3吨水，进水管B每小时注入2吨水，出水管每小时排出1吨水。水池容量为20吨，从空池开始，多久能注满？",
        "answer": "5小时"
    },
    {
        "prompt": "以下Python代码的输出是什么？\n```python\nx = [1, 2, 3]\ny = x\ny.append(4)\nprint(x)\n```",
        "answer": "[1, 2, 3, 4]"
    }
]

# 系统提示词（引导模型使用 <think> 标签）
SYSTEM_PROMPT = """你是一个善于思考的AI助手。回答问题时，请先在 <think> 标签中展示你的思考过程，然后给出最终答案。

格式：
<think>
这里是你的思考过程...
</think>

这里是最终答案。"""
```

---

## 5. GRPO 训练 Agent 模型

### 5.1 Agent 训练（ART 框架）

```python
"""
使用 Unsloth + ART（Agent Reinforcement Trainer）训练 Agent 模型

ART 在 Unsloth 的 GRPO 基础上增加了：
1. 多轮对话（Multi-Turn）支持
2. 工具调用（Tool Call）支持
3. 轨迹（Trajectory）评分
"""

# 安装 ART
# pip install agent-reinforcement-trainer

from unsloth import FastLanguageModel
from art import ARTTrainer, Tool, Environment

# 加载模型
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/Llama-3.1-8B-Instruct",
    max_seq_length=4096,
    load_in_4bit=True,
)

model = FastLanguageModel.get_peft_model(model, r=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                     "gate_proj", "up_proj", "down_proj"],
    use_gradient_checkpointing="unsloth")

# 🔥 定义工具
def search_database(query: str) -> str:
    """搜索数据库"""
    # 模拟数据库查询
    data = {"用户数": "10000", "订单数": "50000"}
    for key, value in data.items():
        if key in query:
            return f"查询结果：{key} = {value}"
    return "未找到相关数据"

def calculate(expression: str) -> str:
    """数学计算"""
    try:
        return str(eval(expression))
    except:
        return "计算错误"

tools = [
    Tool(name="search_database", func=search_database,
         description="搜索数据库获取信息"),
    Tool(name="calculate", func=calculate,
         description="执行数学计算"),
]

# 🔥 定义 Agent 奖励函数
def agent_reward(trajectory) -> float:
    """评估 Agent 轨迹"""
    score = 0.0
    
    # 1. 是否完成了任务
    if trajectory.task_completed:
        score += 2.0
    
    # 2. 工具使用是否合理
    tool_calls = trajectory.get_tool_calls()
    if len(tool_calls) > 0 and len(tool_calls) <= 5:
        score += 1.0  # 使用了工具且不过多
    elif len(tool_calls) > 5:
        score -= 0.5  # 工具调用过多
    
    # 3. 是否有无效的工具调用
    failed_calls = [tc for tc in tool_calls if tc.failed]
    score -= len(failed_calls) * 0.3
    
    # 4. 效率（轮次越少越好）
    num_turns = trajectory.num_turns
    if num_turns <= 3:
        score += 0.5
    elif num_turns > 8:
        score -= 0.5
    
    return score

# 🔥 训练 Agent
trainer = ARTTrainer(
    model=model,
    tokenizer=tokenizer,
    tools=tools,
    reward_fn=agent_reward,
    num_generations=4,
    max_turns=10,
    learning_rate=5e-6,
)

# Agent 训练数据
agent_tasks = [
    {"task": "查询数据库中的用户数，然后计算如果每个用户平均消费100元，总收入是多少"},
    {"task": "搜索订单数据，分析订单趋势"},
]

trainer.train(agent_tasks, num_epochs=3)
```

---

## 6. 模型导出和部署

### 6.1 导出格式

```python
# 🔥 保存为 LoRA 适配器
model.save_pretrained("./my_model_lora")

# 🔥 合并 LoRA 并保存完整模型
model.save_pretrained_merged(
    "./my_model_merged",
    tokenizer,
    save_method="merged_16bit"  # 16bit 精度
)

# 🔥 导出为 GGUF（用于 llama.cpp / Ollama）
model.save_pretrained_gguf(
    "./my_model_gguf",
    tokenizer,
    quantization_method="q4_k_m"  # 4bit 量化
)

# 🔥 推送到 Hugging Face Hub
model.push_to_hub_merged(
    "your-username/my-reasoning-model",
    tokenizer,
    save_method="merged_16bit"
)

# 导出 GGUF 到 Hub
model.push_to_hub_gguf(
    "your-username/my-reasoning-model-gguf",
    tokenizer,
    quantization_method="q4_k_m"
)
```

### 6.2 在 Ollama 中使用

```bash
# 创建 Modelfile
cat > Modelfile << 'EOF'
FROM ./my_model_gguf/unsloth.Q4_K_M.gguf

TEMPLATE """<|begin_of_text|><|start_header_id|>system<|end_header_id|>
{{ .System }}<|eot_id|><|start_header_id|>user<|end_header_id|>
{{ .Prompt }}<|eot_id|><|start_header_id|>assistant<|end_header_id|>
{{ .Response }}<|eot_id|>"""

SYSTEM "你是一个善于思考的AI助手。回答前先在<think>标签中思考。"

PARAMETER temperature 0.7
PARAMETER top_p 0.9
EOF

# 创建 Ollama 模型
ollama create my-reasoning-model -f Modelfile

# 运行
ollama run my-reasoning-model "解释Python的GIL机制"
```

---

## 7. 训练调优指南

### 7.1 LoRA 超参数调优

```python
"""
LoRA 超参数调优指南：

r（rank）：
- 默认 16，控制 LoRA 的表达能力
- 简单任务：8-16
- 复杂任务：32-64
- 越大效果越好但显存越多

lora_alpha：
- 通常设为 r 的 1-2 倍
- alpha/r 是实际的缩放因子
- 推荐：lora_alpha = r（即缩放因子为1）

target_modules：
- 最少：["q_proj", "v_proj"]
- 推荐：["q_proj", "k_proj", "v_proj", "o_proj"]
- 最全：加上 ["gate_proj", "up_proj", "down_proj"]
- 模块越多效果越好但训练越慢

learning_rate：
- SFT：1e-4 ~ 5e-4
- GRPO：1e-6 ~ 1e-5（更小，因为是 RL）
"""
```

### 7.2 GRPO 特有参数

```python
"""
GRPO 关键参数：

num_generations（Group Size）：
- 默认 8，每个问题生成的候选数
- 越大训练越稳定但越慢
- 推荐：4-16

beta（KL 系数）：
- 控制新策略与原始策略的偏离程度
- 太大：模型不敢探索
- 太小：模型可能崩溃
- 推荐：0.01-0.1

max_completion_length：
- 最大生成长度
- 推理任务：1024-2048
- Agent 任务：2048-4096

奖励函数设计原则：
1. 奖励要有区分度（不要全是 0 或 1）
2. 正确性奖励权重最高
3. 格式奖励辅助引导
4. 避免奖励 hacking（模型找到捷径获取高奖励）
"""
```

---

## 8. 总结

### 核心要点
1. GRPO 是当前最先进的 LLM 训练范式，让模型学会推理而非背诵
2. Unsloth 让 GRPO 训练在消费级显卡上成为可能（5GB VRAM）
3. 奖励函数设计是 GRPO 训练的核心，直接决定模型能力
4. ART 框架扩展了 GRPO 到多轮 Agent 训练场景
5. 训练好的模型可以导出为 GGUF 在 Ollama 中本地运行

### 学习路径
1. 掌握 Unsloth + LoRA 的 SFT 微调
2. 理解 GRPO 算法原理
3. 设计奖励函数，训练推理模型
4. 进阶：使用 ART 训练 Agent 模型
5. 模型导出和部署（GGUF/Ollama）

---

## 🔗 相关资源

- [Unsloth 官方文档](https://docs.unsloth.ai/)
- [Unsloth GitHub](https://github.com/unslothai/unsloth)
- [Unsloth GRPO 教程](https://docs.unsloth.ai/basics/reinforcement-learning-rl-guide)
- [TRL GRPO 文档](https://huggingface.co/docs/trl/grpo_trainer)
- [DeepSeek-R1 论文](https://arxiv.org/abs/2501.12948)
- [Hugging Face LLM Course - GRPO](https://huggingface.co/learn/llm-course/en/chapter12/6)
- [ART Agent Trainer](https://docs.unsloth.ai/basics/reinforcement-learning-rl-guide/training-ai-agents-with-rl)

---

**@author erik.zhou**
