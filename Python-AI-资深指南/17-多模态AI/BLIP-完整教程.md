# BLIP 完整教程

> @author erik.zhou

## 📋 目录

- [BLIP 基础](#blip-基础)
- [图像描述](#图像描述)
- [视觉问答](#视觉问答)
- [图文匹配](#图文匹配)
- [实战案例](#实战案例)

---

## BLIP 基础

### 什么是 BLIP

```python
"""
BLIP (Bootstrapping Language-Image Pre-training)
统一的视觉-语言理解和生成模型
"""

# BLIP 特点
features = {
    "统一架构": "同时支持理解和生成任务",
    "自举学习": "从噪声数据中学习",
    "多任务": "图像描述、VQA、检索等",
    "高性能": "在多个基准上达到SOTA"
}
```

### 安装配置

```bash
# 安装依赖
pip install transformers torch pillow

# 或从源码安装
pip install git+https://github.com/salesforce/BLIP.git
```

### 基础使用

```python
"""
BLIP 基础示例
"""
from transformers import BlipProcessor, BlipForConditionalGeneration
from PIL import Image

# 加载模型
processor = BlipProcessor.from_pretrained("Salesforce/blip-image-captioning-base")
model = BlipForConditionalGeneration.from_pretrained("Salesforce/blip-image-captioning-base")

# 加载图像
image = Image.open("image.jpg")

# 生成描述
inputs = processor(image, return_tensors="pt")
outputs = model.generate(**inputs)
caption = processor.decode(outputs[0], skip_special_tokens=True)

print(f"图像描述: {caption}")
```

---

## 图像描述

### 自动图像描述

```python
"""
使用 BLIP 生成图像描述
"""
from transformers import BlipProcessor, BlipForConditionalGeneration
from PIL import Image
import torch

class BLIPCaptioner:
    """BLIP 图像描述生成器"""
    
    def __init__(self, model_name="Salesforce/blip-image-captioning-base"):
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.processor = BlipProcessor.from_pretrained(model_name)
        self.model = BlipForConditionalGeneration.from_pretrained(model_name).to(self.device)
    
    def generate_caption(self, image_path, num_captions=1):
        """
        生成图像描述
        
        Args:
            image_path: 图像路径
            num_captions: 生成描述数量
        """
        image = Image.open(image_path).convert('RGB')
        
        # 处理图像
        inputs = self.processor(image, return_tensors="pt").to(self.device)
        
        # 生成描述
        outputs = self.model.generate(
            **inputs,
            num_return_sequences=num_captions,
            num_beams=num_captions,
            max_length=50
        )
        
        captions = [
            self.processor.decode(output, skip_special_tokens=True)
            for output in outputs
        ]
        
        return captions
    
    def generate_with_prompt(self, image_path, prompt):
        """
        使用提示生成描述
        
        Args:
            image_path: 图像路径
            prompt: 提示文本
        """
        image = Image.open(image_path).convert('RGB')
        
        # 处理图像和文本
        inputs = self.processor(image, prompt, return_tensors="pt").to(self.device)
        
        # 生成描述
        outputs = self.model.generate(**inputs, max_length=50)
        caption = self.processor.decode(outputs[0], skip_special_tokens=True)
        
        return caption

# 使用示例
captioner = BLIPCaptioner()

# 生成多个描述
captions = captioner.generate_caption("image.jpg", num_captions=3)
for i, caption in enumerate(captions, 1):
    print(f"{i}. {caption}")

# 使用提示生成
prompt = "a photo of"
caption = captioner.generate_with_prompt("image.jpg", prompt)
print(f"\n带提示的描述: {caption}")
```

---

## 视觉问答

### VQA 系统

```python
"""
使用 BLIP 进行视觉问答
"""
from transformers import BlipProcessor, BlipForQuestionAnswering
from PIL import Image
import torch

class BLIPVQA:
    """BLIP 视觉问答系统"""
    
    def __init__(self, model_name="Salesforce/blip-vqa-base"):
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.processor = BlipProcessor.from_pretrained(model_name)
        self.model = BlipForQuestionAnswering.from_pretrained(model_name).to(self.device)
    
    def answer_question(self, image_path, question):
        """
        回答关于图像的问题
        
        Args:
            image_path: 图像路径
            question: 问题文本
        """
        image = Image.open(image_path).convert('RGB')
        
        # 处理输入
        inputs = self.processor(image, question, return_tensors="pt").to(self.device)
        
        # 生成答案
        outputs = self.model.generate(**inputs, max_length=50)
        answer = self.processor.decode(outputs[0], skip_special_tokens=True)
        
        return answer
    
    def batch_qa(self, image_path, questions):
        """
        批量问答
        
        Args:
            image_path: 图像路径
            questions: 问题列表
        """
        results = []
        
        for question in questions:
            answer = self.answer_question(image_path, question)
            results.append({
                'question': question,
                'answer': answer
            })
        
        return results

# 使用示例
vqa = BLIPVQA()

# 单个问题
answer = vqa.answer_question("image.jpg", "What is in the image?")
print(f"答案: {answer}")

# 批量问答
questions = [
    "What color is the cat?",
    "Where is the cat?",
    "What is the cat doing?"
]

results = vqa.batch_qa("cat.jpg", questions)
for result in results:
    print(f"Q: {result['question']}")
    print(f"A: {result['answer']}\n")
```

---

## 图文匹配

### 图像-文本检索

```python
"""
使用 BLIP 进行图像-文本匹配
"""
from transformers import BlipProcessor, BlipForImageTextRetrieval
from PIL import Image
import torch

class BLIPRetrieval:
    """BLIP 图文检索系统"""
    
    def __init__(self, model_name="Salesforce/blip-itm-base-coco"):
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.processor = BlipProcessor.from_pretrained(model_name)
        self.model = BlipForImageTextRetrieval.from_pretrained(model_name).to(self.device)
    
    def compute_similarity(self, image_path, text):
        """
        计算图像和文本的相似度
        
        Args:
            image_path: 图像路径
            text: 文本描述
        """
        image = Image.open(image_path).convert('RGB')
        
        # 处理输入
        inputs = self.processor(image, text, return_tensors="pt").to(self.device)
        
        # 计算相似度
        with torch.no_grad():
            outputs = self.model(**inputs)
            similarity = outputs.itm_score[:, 1].item()  # 匹配分数
        
        return similarity
    
    def rank_texts(self, image_path, texts):
        """
        对文本进行排序
        
        Args:
            image_path: 图像路径
            texts: 文本列表
        """
        results = []
        
        for text in texts:
            similarity = self.compute_similarity(image_path, text)
            results.append({
                'text': text,
                'similarity': similarity
            })
        
        # 按相似度排序
        results.sort(key=lambda x: x['similarity'], reverse=True)
        
        return results
    
    def find_best_match(self, image_paths, text):
        """
        找到最匹配的图像
        
        Args:
            image_paths: 图像路径列表
            text: 文本描述
        """
        results = []
        
        for image_path in image_paths:
            similarity = self.compute_similarity(image_path, text)
            results.append({
                'image_path': image_path,
                'similarity': similarity
            })
        
        # 按相似度排序
        results.sort(key=lambda x: x['similarity'], reverse=True)
        
        return results

# 使用示例
retrieval = BLIPRetrieval()

# 文本排序
texts = [
    "a cat sitting on a couch",
    "a dog playing in the park",
    "a bird flying in the sky"
]

results = retrieval.rank_texts("image.jpg", texts)
print("文本排序结果:")
for i, result in enumerate(results, 1):
    print(f"{i}. {result['text']} (相似度: {result['similarity']:.4f})")

# 图像检索
image_paths = ["cat.jpg", "dog.jpg", "bird.jpg"]
query = "a cute cat"

results = retrieval.find_best_match(image_paths, query)
print(f"\n查询: {query}")
print("最匹配的图像:")
for i, result in enumerate(results, 1):
    print(f"{i}. {result['image_path']} (相似度: {result['similarity']:.4f})")
```

---

## 实战案例

### 智能相册

```python
"""
基于 BLIP 的智能相册系统
"""
import os
from PIL import Image

class SmartPhotoAlbum:
    """智能相册系统"""
    
    def __init__(self):
        self.captioner = BLIPCaptioner()
        self.vqa = BLIPVQA()
        self.retrieval = BLIPRetrieval()
        self.photo_database = {}
    
    def index_photos(self, photo_dir):
        """
        索引照片目录
        
        Args:
            photo_dir: 照片目录路径
        """
        print("正在索引照片...")
        
        for filename in os.listdir(photo_dir):
            if filename.endswith(('.jpg', '.png', '.jpeg')):
                photo_path = os.path.join(photo_dir, filename)
                
                # 生成描述
                caption = self.captioner.generate_caption(photo_path)[0]
                
                self.photo_database[photo_path] = {
                    'filename': filename,
                    'caption': caption
                }
        
        print(f"已索引 {len(self.photo_database)} 张照片")
    
    def search_by_text(self, query, top_k=5):
        """
        根据文本搜索照片
        
        Args:
            query: 搜索查询
            top_k: 返回前 k 个结果
        """
        results = []
        
        for photo_path, info in self.photo_database.items():
            similarity = self.retrieval.compute_similarity(photo_path, query)
            results.append({
                'photo_path': photo_path,
                'filename': info['filename'],
                'caption': info['caption'],
                'similarity': similarity
            })
        
        results.sort(key=lambda x: x['similarity'], reverse=True)
        
        return results[:top_k]
    
    def ask_about_photo(self, photo_path, question):
        """
        询问关于照片的问题
        
        Args:
            photo_path: 照片路径
            question: 问题
        """
        return self.vqa.answer_question(photo_path, question)

# 使用示例
album = SmartPhotoAlbum()
album.index_photos("./photos")

# 搜索照片
results = album.search_by_text("a cat", top_k=3)
print("搜索结果:")
for i, result in enumerate(results, 1):
    print(f"{i}. {result['filename']}")
    print(f"   描述: {result['caption']}")
    print(f"   相似度: {result['similarity']:.4f}\n")

# 询问照片
answer = album.ask_about_photo("cat.jpg", "What color is the cat?")
print(f"答案: {answer}")
```

---

**记住：BLIP 是多功能的视觉-语言模型，适合各种图文理解任务！** 🖼️

@author erik.zhou
