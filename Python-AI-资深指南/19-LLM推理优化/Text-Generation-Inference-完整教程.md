# Text Generation Inference 完整教程

> @author erik.zhou

## 📋 目录

- [TGI 基础](#tgi-基础)
- [模型部署](#模型部署)
- [推理优化](#推理优化)
- [分布式推理](#分布式推理)
- [实战案例](#实战案例)

---

## TGI 基础

### 什么是 TGI

```python
"""
Text Generation Inference (TGI)
Hugging Face 开发的高性能推理服务器
"""

# TGI 特点
features = {
    "高性能": "优化的推理引擎，支持多种加速技术",
    "易用性": "简单的 API 接口，兼容 OpenAI 格式",
    "流式生成": "支持流式输出，提升用户体验",
    "分布式": "支持多 GPU 和张量并行",
    "生产就绪": "内置监控、日志和错误处理"
}
```

### 安装配置

```bash
# Docker 安装（推荐）
docker pull ghcr.io/huggingface/text-generation-inference:latest

# 启动服务
docker run --gpus all --shm-size 1g -p 8080:80 \
  -v $PWD/data:/data \
  ghcr.io/huggingface/text-generation-inference:latest \
  --model-id meta-llama/Llama-2-7b-chat-hf \
  --num-shard 1

# 或使用 pip 安装
pip install text-generation
```

### 基础使用

```python
"""
TGI 客户端基础使用
"""
from text_generation import Client

# 连接到 TGI 服务
client = Client("http://localhost:8080")

# 生成文本
response = client.generate(
    "What is deep learning?",
    max_new_tokens=100,
    temperature=0.7
)

print(response.generated_text)

# 流式生成
for response in client.generate_stream(
    "Tell me a story",
    max_new_tokens=200
):
    if not response.token.special:
        print(response.token.text, end="")
```

---

## 模型部署

### 部署 Llama 模型

```python
"""
部署 Llama 2 模型
"""
import subprocess

def deploy_llama_model(
    model_id="meta-llama/Llama-2-7b-chat-hf",
    num_shard=1,
    port=8080
):
    """
    部署 Llama 模型
    
    Args:
        model_id: 模型 ID
        num_shard: GPU 分片数量
        port: 服务端口
    """
    cmd = [
        "docker", "run", "--gpus", "all",
        "--shm-size", "1g",
        "-p", f"{port}:80",
        "-v", "$PWD/data:/data",
        "ghcr.io/huggingface/text-generation-inference:latest",
        "--model-id", model_id,
        "--num-shard", str(num_shard),
        "--max-batch-prefill-tokens", "4096",
        "--max-total-tokens", "8192"
    ]
    
    subprocess.run(cmd)

# 使用示例
deploy_llama_model(
    model_id="meta-llama/Llama-2-7b-chat-hf",
    num_shard=1,
    port=8080
)
```

### 自定义配置

```python
"""
TGI 高级配置
"""

class TGIConfig:
    """TGI 配置管理"""
    
    def __init__(self):
        self.config = {
            # 模型配置
            "model_id": "meta-llama/Llama-2-7b-chat-hf",
            "revision": "main",
            "num_shard": 1,
            
            # 性能配置
            "max_batch_prefill_tokens": 4096,
            "max_total_tokens": 8192,
            "max_input_length": 4096,
            "max_batch_total_tokens": 16384,
            
            # 优化配置
            "quantize": None,  # bitsandbytes, gptq
            "dtype": "float16",
            
            # 服务配置
            "port": 8080,
            "hostname": "0.0.0.0",
            "max_concurrent_requests": 128,
            
            # 日志配置
            "json_output": True,
            "otlp_endpoint": None
        }
    
    def to_docker_args(self):
        """转换为 Docker 启动参数"""
        args = []
        
        for key, value in self.config.items():
            if value is not None:
                arg_name = key.replace('_', '-')
                args.extend([f"--{arg_name}", str(value)])
        
        return args

# 使用示例
config = TGIConfig()
config.config["quantize"] = "bitsandbytes"
config.config["num_shard"] = 2

docker_args = config.to_docker_args()
print("Docker 参数:", " ".join(docker_args))
```

---

## 推理优化

### 量化部署

```python
"""
使用量化技术部署模型
"""

def deploy_quantized_model(
    model_id,
    quantize_method="bitsandbytes"
):
    """
    部署量化模型
    
    Args:
        model_id: 模型 ID
        quantize_method: 量化方法 (bitsandbytes, gptq)
    """
    cmd = [
        "docker", "run", "--gpus", "all",
        "--shm-size", "1g",
        "-p", "8080:80",
        "ghcr.io/huggingface/text-generation-inference:latest",
        "--model-id", model_id,
        "--quantize", quantize_method
    ]
    
    subprocess.run(cmd)

# 使用 bitsandbytes 量化
deploy_quantized_model(
    model_id="meta-llama/Llama-2-13b-chat-hf",
    quantize_method="bitsandbytes"
)

# 使用 GPTQ 量化
deploy_quantized_model(
    model_id="TheBloke/Llama-2-13B-chat-GPTQ",
    quantize_method="gptq"
)
```

### 批处理优化

```python
"""
TGI 批处理优化
"""
from text_generation import Client
import asyncio

class BatchInferenceClient:
    """批量推理客户端"""
    
    def __init__(self, endpoint="http://localhost:8080"):
        self.client = Client(endpoint)
    
    async def batch_generate(self, prompts, **kwargs):
        """
        批量生成
        
        Args:
            prompts: 提示列表
            **kwargs: 生成参数
        """
        tasks = [
            self.async_generate(prompt, **kwargs)
            for prompt in prompts
        ]
        
        results = await asyncio.gather(*tasks)
        return results
    
    async def async_generate(self, prompt, **kwargs):
        """异步生成"""
        loop = asyncio.get_event_loop()
        result = await loop.run_in_executor(
            None,
            lambda: self.client.generate(prompt, **kwargs)
        )
        return result.generated_text

# 使用示例
async def main():
    client = BatchInferenceClient()
    
    prompts = [
        "What is AI?",
        "Explain machine learning",
        "What is deep learning?"
    ]
    
    results = await client.batch_generate(
        prompts,
        max_new_tokens=100,
        temperature=0.7
    )
    
    for prompt, result in zip(prompts, results):
        print(f"Prompt: {prompt}")
        print(f"Result: {result}\n")

# 运行
asyncio.run(main())
```

---

## 分布式推理

### 多 GPU 部署

```python
"""
多 GPU 分布式推理
"""

def deploy_multi_gpu(
    model_id,
    num_gpus=2
):
    """
    多 GPU 部署
    
    Args:
        model_id: 模型 ID
        num_gpus: GPU 数量
    """
    cmd = [
        "docker", "run",
        "--gpus", f"'device=0,1'",  # 指定 GPU
        "--shm-size", "1g",
        "-p", "8080:80",
        "ghcr.io/huggingface/text-generation-inference:latest",
        "--model-id", model_id,
        "--num-shard", str(num_gpus),
        "--max-batch-prefill-tokens", "8192"
    ]
    
    subprocess.run(cmd)

# 使用 2 个 GPU
deploy_multi_gpu(
    model_id="meta-llama/Llama-2-70b-chat-hf",
    num_gpus=2
)
```

### 张量并行

```python
"""
张量并行配置
"""

class TensorParallelConfig:
    """张量并行配置"""
    
    def __init__(self, num_gpus):
        self.num_gpus = num_gpus
        self.config = {
            "num_shard": num_gpus,
            "max_batch_prefill_tokens": 4096 * num_gpus,
            "max_total_tokens": 8192 * num_gpus,
            "max_batch_total_tokens": 16384 * num_gpus
        }
    
    def calculate_memory_usage(self, model_size_gb):
        """
        计算内存使用
        
        Args:
            model_size_gb: 模型大小（GB）
        """
        per_gpu_memory = model_size_gb / self.num_gpus
        
        # 加上激活值和 KV 缓存
        total_per_gpu = per_gpu_memory * 1.5
        
        return {
            "per_gpu_memory_gb": total_per_gpu,
            "total_memory_gb": total_per_gpu * self.num_gpus
        }

# 使用示例
config = TensorParallelConfig(num_gpus=4)
memory = config.calculate_memory_usage(model_size_gb=140)

print(f"每个 GPU 内存: {memory['per_gpu_memory_gb']:.2f} GB")
print(f"总内存: {memory['total_memory_gb']:.2f} GB")
```

---

## 实战案例

### 聊天机器人服务

```python
"""
基于 TGI 的聊天机器人
"""
from text_generation import Client
from typing import List, Dict

class ChatBot:
    """聊天机器人"""
    
    def __init__(self, endpoint="http://localhost:8080"):
        self.client = Client(endpoint)
        self.conversation_history = []
    
    def format_prompt(self, message: str) -> str:
        """
        格式化提示
        
        Args:
            message: 用户消息
        """
        # Llama 2 Chat 格式
        prompt = "<s>[INST] "
        
        # 添加历史对话
        for turn in self.conversation_history:
            if turn['role'] == 'user':
                prompt += f"{turn['content']} [/INST] "
            else:
                prompt += f"{turn['content']} </s><s>[INST] "
        
        # 添加当前消息
        prompt += f"{message} [/INST]"
        
        return prompt
    
    def chat(self, message: str, stream=False):
        """
        对话
        
        Args:
            message: 用户消息
            stream: 是否流式输出
        """
        prompt = self.format_prompt(message)
        
        if stream:
            return self.chat_stream(prompt, message)
        else:
            response = self.client.generate(
                prompt,
                max_new_tokens=512,
                temperature=0.7,
                top_p=0.9
            )
            
            # 更新历史
            self.conversation_history.append({
                'role': 'user',
                'content': message
            })
            self.conversation_history.append({
                'role': 'assistant',
                'content': response.generated_text
            })
            
            return response.generated_text
    
    def chat_stream(self, prompt: str, message: str):
        """流式对话"""
        full_response = ""
        
        for response in self.client.generate_stream(
            prompt,
            max_new_tokens=512,
            temperature=0.7
        ):
            if not response.token.special:
                token = response.token.text
                full_response += token
                yield token
        
        # 更新历史
        self.conversation_history.append({
            'role': 'user',
            'content': message
        })
        self.conversation_history.append({
            'role': 'assistant',
            'content': full_response
        })
    
    def reset(self):
        """重置对话历史"""
        self.conversation_history = []

# 使用示例
bot = ChatBot()

# 非流式对话
response = bot.chat("Hello! How are you?")
print(f"Bot: {response}")

# 流式对话
print("Bot: ", end="")
for token in bot.chat("Tell me a joke", stream=True):
    print(token, end="", flush=True)
print()

# 重置对话
bot.reset()
```

### API 服务封装

```python
"""
TGI API 服务封装
"""
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from text_generation import Client
import uvicorn

app = FastAPI()
tgi_client = Client("http://localhost:8080")

class GenerateRequest(BaseModel):
    prompt: str
    max_new_tokens: int = 100
    temperature: float = 0.7
    top_p: float = 0.9
    stream: bool = False

class GenerateResponse(BaseModel):
    generated_text: str
    tokens: int

@app.post("/generate", response_model=GenerateResponse)
async def generate(request: GenerateRequest):
    """生成文本"""
    try:
        response = tgi_client.generate(
            request.prompt,
            max_new_tokens=request.max_new_tokens,
            temperature=request.temperature,
            top_p=request.top_p
        )
        
        return GenerateResponse(
            generated_text=response.generated_text,
            tokens=len(response.generated_text.split())
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health():
    """健康检查"""
    return {"status": "healthy"}

# 启动服务
if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 最佳实践

### 性能监控

```python
"""
TGI 性能监控
"""
import time
from text_generation import Client

class PerformanceMonitor:
    """性能监控器"""
    
    def __init__(self, endpoint="http://localhost:8080"):
        self.client = Client(endpoint)
        self.metrics = []
    
    def benchmark(self, prompts, **kwargs):
        """
        性能基准测试
        
        Args:
            prompts: 测试提示列表
            **kwargs: 生成参数
        """
        for prompt in prompts:
            start_time = time.time()
            
            response = self.client.generate(prompt, **kwargs)
            
            end_time = time.time()
            latency = end_time - start_time
            
            tokens = len(response.generated_text.split())
            throughput = tokens / latency
            
            self.metrics.append({
                'prompt_length': len(prompt.split()),
                'generated_tokens': tokens,
                'latency': latency,
                'throughput': throughput
            })
        
        return self.get_statistics()
    
    def get_statistics(self):
        """获取统计信息"""
        if not self.metrics:
            return {}
        
        latencies = [m['latency'] for m in self.metrics]
        throughputs = [m['throughput'] for m in self.metrics]
        
        return {
            'avg_latency': sum(latencies) / len(latencies),
            'p50_latency': sorted(latencies)[len(latencies) // 2],
            'p95_latency': sorted(latencies)[int(len(latencies) * 0.95)],
            'avg_throughput': sum(throughputs) / len(throughputs)
        }

# 使用示例
monitor = PerformanceMonitor()

test_prompts = [
    "What is AI?" * 10,
    "Explain machine learning" * 10,
    "What is deep learning?" * 10
]

stats = monitor.benchmark(
    test_prompts,
    max_new_tokens=100,
    temperature=0.7
)

print(f"平均延迟: {stats['avg_latency']:.2f}s")
print(f"P95 延迟: {stats['p95_latency']:.2f}s")
print(f"平均吞吐量: {stats['avg_throughput']:.2f} tokens/s")
```

---

**记住：TGI 是生产级的推理服务器，合理配置可以获得最佳性能！** 🚀

@author erik.zhou
