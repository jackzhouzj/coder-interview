# llama.cpp 完整教程

> @author erik.zhou

## 📋 目录

- [llama.cpp 基础](#llamacpp-基础)
- [模型量化](#模型量化)
- [CPU 推理优化](#cpu-推理优化)
- [本地部署](#本地部署)
- [实战案例](#实战案例)

---

## llama.cpp 基础

### 什么是 llama.cpp

```python
"""
llama.cpp 是纯 C/C++ 实现的 LLM 推理引擎
专注于 CPU 推理优化和跨平台支持
"""

# llama.cpp 特点
features = {
    "纯 C/C++": "无需 Python 依赖，性能优异",
    "CPU 优化": "充分利用 CPU 指令集（AVX2、AVX512）",
    "量化支持": "支持多种量化格式（4-bit、5-bit、8-bit）",
    "跨平台": "支持 Windows、Linux、macOS、iOS、Android",
    "低内存": "量化后可在普通硬件上运行大模型"
}
```

### 安装配置

```bash
# 克隆仓库
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp

# 编译（CPU）
make

# 编译（支持 CUDA）
make LLAMA_CUBLAS=1

# 编译（支持 Metal，macOS）
make LLAMA_METAL=1

# Python 绑定
pip install llama-cpp-python
```

### 基础使用

```python
"""
llama.cpp Python 绑定基础使用
"""
from llama_cpp import Llama

# 加载模型
llm = Llama(
    model_path="./models/llama-2-7b.Q4_K_M.gguf",
    n_ctx=2048,
    n_threads=8,
    n_gpu_layers=0  # CPU 推理设为 0
)

# 生成文本
output = llm(
    "Q: What is the capital of France? A:",
    max_tokens=50,
    temperature=0.7,
    top_p=0.9,
    echo=True
)

print(output['choices'][0]['text'])
```

---

## 模型量化

### 转换模型格式

```bash
# 1. 下载原始模型（Hugging Face 格式）
# 假设已下载到 ./models/llama-2-7b-hf

# 2. 转换为 GGUF 格式
python convert.py ./models/llama-2-7b-hf \
    --outfile ./models/llama-2-7b-f16.gguf \
    --outtype f16

# 3. 量化模型
./quantize ./models/llama-2-7b-f16.gguf \
    ./models/llama-2-7b-Q4_K_M.gguf \
    Q4_K_M
```

### 量化格式对比

```python
"""
llama.cpp 量化格式说明
"""

quantization_formats = {
    "Q2_K": {
        "bits": 2.5,
        "size_ratio": "~25%",
        "quality": "最低",
        "use_case": "极限压缩"
    },
    "Q3_K_S": {
        "bits": 3.5,
        "size_ratio": "~35%",
        "quality": "较低",
        "use_case": "小内存设备"
    },
    "Q4_K_M": {
        "bits": 4.5,
        "size_ratio": "~45%",
        "quality": "中等",
        "use_case": "推荐的平衡选择"
    },
    "Q5_K_M": {
        "bits": 5.5,
        "size_ratio": "~55%",
        "quality": "较高",
        "use_case": "高质量推理"
    },
    "Q6_K": {
        "bits": 6.5,
        "size_ratio": "~65%",
        "quality": "高",
        "use_case": "接近原始质量"
    },
    "Q8_0": {
        "bits": 8.5,
        "size_ratio": "~85%",
        "quality": "最高",
        "use_case": "最高质量要求"
    }
}

def calculate_model_size(original_size_gb, quant_format):
    """
    计算量化后的模型大小
    
    Args:
        original_size_gb: 原始模型大小（GB）
        quant_format: 量化格式
    """
    ratio = float(quantization_formats[quant_format]["size_ratio"].strip("~%")) / 100
    quantized_size = original_size_gb * ratio
    
    return {
        "original_size": original_size_gb,
        "quantized_size": quantized_size,
        "saved_space": original_size_gb - quantized_size,
        "compression_ratio": f"{(1 - ratio) * 100:.1f}%"
    }

# 使用示例
result = calculate_model_size(13.5, "Q4_K_M")
print(f"原始大小: {result['original_size']} GB")
print(f"量化后: {result['quantized_size']:.2f} GB")
print(f"节省空间: {result['saved_space']:.2f} GB")
print(f"压缩率: {result['compression_ratio']}")
```

### 批量量化

```python
"""
批量量化多个模型
"""
import subprocess
import os

class ModelQuantizer:
    """模型量化器"""
    
    def __init__(self, llama_cpp_path="./llama.cpp"):
        self.llama_cpp_path = llama_cpp_path
        self.quantize_bin = os.path.join(llama_cpp_path, "quantize")
    
    def quantize_model(self, input_path, output_path, quant_type="Q4_K_M"):
        """
        量化单个模型
        
        Args:
            input_path: 输入模型路径（GGUF f16）
            output_path: 输出模型路径
            quant_type: 量化类型
        """
        cmd = [
            self.quantize_bin,
            input_path,
            output_path,
            quant_type
        ]
        
        print(f"量化 {input_path} -> {output_path} ({quant_type})")
        subprocess.run(cmd, check=True)
    
    def batch_quantize(self, input_path, output_dir, quant_types=None):
        """
        批量量化为多种格式
        
        Args:
            input_path: 输入模型路径
            output_dir: 输出目录
            quant_types: 量化类型列表
        """
        if quant_types is None:
            quant_types = ["Q4_K_M", "Q5_K_M", "Q8_0"]
        
        os.makedirs(output_dir, exist_ok=True)
        
        base_name = os.path.basename(input_path).replace("-f16.gguf", "")
        
        for quant_type in quant_types:
            output_path = os.path.join(
                output_dir,
                f"{base_name}-{quant_type}.gguf"
            )
            
            self.quantize_model(input_path, output_path, quant_type)

# 使用示例
quantizer = ModelQuantizer()

# 批量量化
quantizer.batch_quantize(
    input_path="./models/llama-2-7b-f16.gguf",
    output_dir="./models/quantized",
    quant_types=["Q4_K_M", "Q5_K_M", "Q8_0"]
)
```

---

## CPU 推理优化

### 线程优化

```python
"""
CPU 推理线程优化
"""
from llama_cpp import Llama
import multiprocessing

class OptimizedLlamaCPU:
    """优化的 CPU 推理"""
    
    def __init__(self, model_path):
        # 获取 CPU 核心数
        cpu_count = multiprocessing.cpu_count()
        
        # 推荐使用物理核心数
        n_threads = cpu_count // 2
        
        self.llm = Llama(
            model_path=model_path,
            n_ctx=2048,
            n_threads=n_threads,
            n_batch=512,  # 批处理大小
            use_mlock=True,  # 锁定内存，防止交换
            use_mmap=True,  # 使用内存映射
            verbose=False
        )
    
    def generate(self, prompt, **kwargs):
        """生成文本"""
        return self.llm(prompt, **kwargs)
    
    def benchmark(self, prompt, num_runs=5):
        """性能基准测试"""
        import time
        
        times = []
        
        for i in range(num_runs):
            start = time.time()
            self.generate(prompt, max_tokens=100)
            end = time.time()
            times.append(end - start)
        
        avg_time = sum(times) / len(times)
        tokens_per_sec = 100 / avg_time
        
        return {
            "avg_time": avg_time,
            "tokens_per_sec": tokens_per_sec,
            "all_times": times
        }

# 使用示例
llm = OptimizedLlamaCPU("./models/llama-2-7b-Q4_K_M.gguf")

# 基准测试
result = llm.benchmark("What is AI?")
print(f"平均时间: {result['avg_time']:.2f}s")
print(f"吞吐量: {result['tokens_per_sec']:.2f} tokens/s")
```

### 内存优化

```python
"""
内存使用优化
"""
from llama_cpp import Llama

class MemoryEfficientLlama:
    """内存高效的 Llama"""
    
    def __init__(self, model_path, max_memory_gb=8):
        """
        初始化
        
        Args:
            model_path: 模型路径
            max_memory_gb: 最大内存限制（GB）
        """
        # 根据内存限制调整上下文长度
        if max_memory_gb <= 4:
            n_ctx = 512
        elif max_memory_gb <= 8:
            n_ctx = 1024
        else:
            n_ctx = 2048
        
        self.llm = Llama(
            model_path=model_path,
            n_ctx=n_ctx,
            n_batch=256,
            use_mlock=False,  # 低内存时不锁定
            use_mmap=True,
            low_vram=True  # 低显存模式
        )
    
    def generate_with_cache(self, prompt, **kwargs):
        """使用缓存生成"""
        # llama.cpp 自动管理 KV 缓存
        return self.llm(prompt, **kwargs)

# 使用示例
llm = MemoryEfficientLlama(
    model_path="./models/llama-2-7b-Q4_K_M.gguf",
    max_memory_gb=4
)

output = llm.generate_with_cache("Hello, how are you?")
print(output['choices'][0]['text'])
```

---

## 本地部署

### 命令行工具

```bash
# 基础推理
./main -m ./models/llama-2-7b-Q4_K_M.gguf \
    -p "What is AI?" \
    -n 100 \
    -t 8

# 交互式对话
./main -m ./models/llama-2-7b-Q4_K_M.gguf \
    -i \
    --color \
    -r "User:" \
    -t 8

# 服务器模式
./server -m ./models/llama-2-7b-Q4_K_M.gguf \
    --host 0.0.0.0 \
    --port 8080 \
    -t 8
```

### HTTP 服务器

```python
"""
基于 llama.cpp 的 HTTP 服务
"""
from flask import Flask, request, jsonify, Response
from llama_cpp import Llama
import json

app = Flask(__name__)

# 加载模型
llm = Llama(
    model_path="./models/llama-2-7b-Q4_K_M.gguf",
    n_ctx=2048,
    n_threads=8
)

@app.route('/v1/completions', methods=['POST'])
def completions():
    """文本补全接口"""
    data = request.json
    
    prompt = data.get('prompt', '')
    max_tokens = data.get('max_tokens', 100)
    temperature = data.get('temperature', 0.7)
    stream = data.get('stream', False)
    
    if stream:
        return Response(
            stream_generate(prompt, max_tokens, temperature),
            mimetype='text/event-stream'
        )
    else:
        output = llm(
            prompt,
            max_tokens=max_tokens,
            temperature=temperature,
            echo=False
        )
        
        return jsonify({
            'id': 'cmpl-xxx',
            'object': 'text_completion',
            'created': int(time.time()),
            'model': 'llama-2-7b',
            'choices': [{
                'text': output['choices'][0]['text'],
                'index': 0,
                'finish_reason': 'stop'
            }]
        })

def stream_generate(prompt, max_tokens, temperature):
    """流式生成"""
    for output in llm(
        prompt,
        max_tokens=max_tokens,
        temperature=temperature,
        stream=True
    ):
        chunk = {
            'id': 'cmpl-xxx',
            'object': 'text_completion',
            'created': int(time.time()),
            'model': 'llama-2-7b',
            'choices': [{
                'text': output['choices'][0]['text'],
                'index': 0,
                'finish_reason': None
            }]
        }
        
        yield f"data: {json.dumps(chunk)}\n\n"
    
    yield "data: [DONE]\n\n"

@app.route('/health', methods=['GET'])
def health():
    """健康检查"""
    return jsonify({'status': 'healthy'})

if __name__ == '__main__':
    import time
    app.run(host='0.0.0.0', port=8080)
```

---

## 实战案例

### 本地聊天助手

```python
"""
基于 llama.cpp 的本地聊天助手
"""
from llama_cpp import Llama

class LocalChatAssistant:
    """本地聊天助手"""
    
    def __init__(self, model_path):
        self.llm = Llama(
            model_path=model_path,
            n_ctx=2048,
            n_threads=8,
            verbose=False
        )
        self.conversation_history = []
    
    def format_prompt(self, message):
        """
        格式化提示（Llama 2 Chat 格式）
        
        Args:
            message: 用户消息
        """
        prompt = "<s>"
        
        for turn in self.conversation_history:
            if turn['role'] == 'user':
                prompt += f"[INST] {turn['content']} [/INST] "
            else:
                prompt += f"{turn['content']} </s><s>"
        
        prompt += f"[INST] {message} [/INST]"
        
        return prompt
    
    def chat(self, message, stream=False):
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
            output = self.llm(
                prompt,
                max_tokens=512,
                temperature=0.7,
                top_p=0.9,
                stop=["</s>", "[INST]"]
            )
            
            response = output['choices'][0]['text'].strip()
            
            # 更新历史
            self.conversation_history.append({
                'role': 'user',
                'content': message
            })
            self.conversation_history.append({
                'role': 'assistant',
                'content': response
            })
            
            return response
    
    def chat_stream(self, prompt, message):
        """流式对话"""
        full_response = ""
        
        for output in self.llm(
            prompt,
            max_tokens=512,
            temperature=0.7,
            stream=True,
            stop=["</s>", "[INST]"]
        ):
            token = output['choices'][0]['text']
            full_response += token
            yield token
        
        # 更新历史
        self.conversation_history.append({
            'role': 'user',
            'content': message
        })
        self.conversation_history.append({
            'role': 'assistant',
            'content': full_response.strip()
        })
    
    def reset(self):
        """重置对话"""
        self.conversation_history = []

# 使用示例
assistant = LocalChatAssistant("./models/llama-2-7b-Q4_K_M.gguf")

# 非流式对话
response = assistant.chat("Hello! How are you?")
print(f"Assistant: {response}")

# 流式对话
print("Assistant: ", end="")
for token in assistant.chat("Tell me a joke", stream=True):
    print(token, end="", flush=True)
print()

# 重置对话
assistant.reset()
```

### 文档问答系统

```python
"""
基于 llama.cpp 的文档问答
"""
from llama_cpp import Llama

class DocumentQA:
    """文档问答系统"""
    
    def __init__(self, model_path):
        self.llm = Llama(
            model_path=model_path,
            n_ctx=4096,  # 更大的上下文
            n_threads=8
        )
    
    def answer_question(self, document, question):
        """
        回答文档相关问题
        
        Args:
            document: 文档内容
            question: 问题
        """
        prompt = f"""Document:
{document}

Question: {question}

Answer based on the document above:"""
        
        output = self.llm(
            prompt,
            max_tokens=256,
            temperature=0.3,  # 较低温度，更准确
            top_p=0.9
        )
        
        return output['choices'][0]['text'].strip()
    
    def batch_qa(self, document, questions):
        """批量问答"""
        results = []
        
        for question in questions:
            answer = self.answer_question(document, question)
            results.append({
                'question': question,
                'answer': answer
            })
        
        return results

# 使用示例
qa_system = DocumentQA("./models/llama-2-7b-Q4_K_M.gguf")

document = """
Python is a high-level programming language.
It was created by Guido van Rossum in 1991.
Python is known for its simple syntax and readability.
"""

questions = [
    "Who created Python?",
    "When was Python created?",
    "What is Python known for?"
]

results = qa_system.batch_qa(document, questions)

for result in results:
    print(f"Q: {result['question']}")
    print(f"A: {result['answer']}\n")
```

---

## 最佳实践

### 模型选择指南

```python
"""
根据硬件选择合适的模型和量化格式
"""

def recommend_model(ram_gb, cpu_cores):
    """
    推荐模型配置
    
    Args:
        ram_gb: 可用内存（GB）
        cpu_cores: CPU 核心数
    """
    recommendations = []
    
    if ram_gb >= 32 and cpu_cores >= 8:
        recommendations.append({
            "model": "Llama-2-13B",
            "quantization": "Q5_K_M",
            "expected_performance": "高质量，中等速度"
        })
    
    if ram_gb >= 16 and cpu_cores >= 4:
        recommendations.append({
            "model": "Llama-2-7B",
            "quantization": "Q4_K_M",
            "expected_performance": "平衡的质量和速度"
        })
    
    if ram_gb >= 8 and cpu_cores >= 2:
        recommendations.append({
            "model": "Llama-2-7B",
            "quantization": "Q3_K_S",
            "expected_performance": "较快速度，质量一般"
        })
    
    if ram_gb < 8:
        recommendations.append({
            "model": "TinyLlama-1.1B",
            "quantization": "Q4_K_M",
            "expected_performance": "快速，适合低配置"
        })
    
    return recommendations

# 使用示例
recommendations = recommend_model(ram_gb=16, cpu_cores=8)

print("推荐配置:")
for i, rec in enumerate(recommendations, 1):
    print(f"\n{i}. {rec['model']} ({rec['quantization']})")
    print(f"   预期性能: {rec['expected_performance']}")
```

---

**记住：llama.cpp 是 CPU 推理的最佳选择，合理量化可以在普通硬件上运行大模型！** 💻

@author erik.zhou
