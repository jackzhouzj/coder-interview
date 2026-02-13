# CLIP 完整教程

> @author erik.zhou

## 📋 目录

- [CLIP 基础](#clip-基础)
- [模型架构](#模型架构)
- [零样本分类](#零样本分类)
- [图像检索](#图像检索)
- [实战案例](#实战案例)

---

## CLIP 基础

### 什么是 CLIP

```python
"""
CLIP (Contrastive Language-Image Pre-training)
通过对比学习将图像和文本映射到同一空间
"""

# CLIP 核心特点
features = {
    "零样本学习": "无需训练即可分类新类别",
    "多模态": "同时理解图像和文本",
    "大规模预训练": "在4亿图文对上训练",
    "灵活应用": "图像分类、检索、生成等"
}
```

### 安装配置

```bash
# 安装 CLIP
pip install git+https://github.com/openai/CLIP.git
pip install torch torchvision pillow

# 或使用 Hugging Face 版本
pip install transformers
```

### 基础使用

```python
"""
CLIP 基础示例
"""
import torch
import clip
from PIL import Image

# 加载模型
device = "cuda" if torch.cuda.is_available() else "cpu"
model, preprocess = clip.load("ViT-B/32", device=device)

# 加载图像
image = preprocess(Image.open("cat.jpg")).unsqueeze(0).to(device)

# 准备文本
text = clip.tokenize(["a cat", "a dog", "a bird"]).to(device)

# 计算特征
with torch.no_grad():
    image_features = model.encode_image(image)
    text_features = model.encode_text(text)
    
    # 计算相似度
    logits_per_image, logits_per_text = model(image, text)
    probs = logits_per_image.softmax(dim=-1).cpu().numpy()

print("预测概率:", probs)
```

---

## 模型架构

### 双编码器结构

```python
"""
CLIP 架构解析
"""

class CLIPArchitecture:
    """CLIP 模型架构"""
    
    def __init__(self):
        self.image_encoder = "ViT or ResNet"
        self.text_encoder = "Transformer"
        self.projection = "Linear Layer"
    
    def forward(self, images, texts):
        # 1. 图像编码
        image_features = self.encode_image(images)
        
        # 2. 文本编码
        text_features = self.encode_text(texts)
        
        # 3. 特征归一化
        image_features = image_features / image_features.norm(dim=-1, keepdim=True)
        text_features = text_features / text_features.norm(dim=-1, keepdim=True)
        
        # 4. 计算相似度
        similarity = image_features @ text_features.T
        
        return similarity
```

---

## 零样本分类

### 图像分类

```python
"""
使用 CLIP 进行零样本图像分类
"""
import torch
import clip
from PIL import Image

class CLIPClassifier:
    """CLIP 零样本分类器"""
    
    def __init__(self, model_name="ViT-B/32"):
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.model, self.preprocess = clip.load(model_name, device=self.device)
    
    def classify(self, image_path, categories):
        """
        对图像进行分类
        
        Args:
            image_path: 图像路径
            categories: 类别列表
        """
        # 加载图像
        image = self.preprocess(Image.open(image_path)).unsqueeze(0).to(self.device)
        
        # 准备文本提示
        text_inputs = torch.cat([
            clip.tokenize(f"a photo of a {c}") for c in categories
        ]).to(self.device)
        
        # 计算特征
        with torch.no_grad():
            image_features = self.model.encode_image(image)
            text_features = self.model.encode_text(text_inputs)
            
            # 归一化
            image_features /= image_features.norm(dim=-1, keepdim=True)
            text_features /= text_features.norm(dim=-1, keepdim=True)
            
            # 计算相似度
            similarity = (100.0 * image_features @ text_features.T).softmax(dim=-1)
        
        # 返回结果
        values, indices = similarity[0].topk(5)
        
        results = []
        for value, index in zip(values, indices):
            results.append({
                'category': categories[index],
                'probability': value.item()
            })
        
        return results

# 使用示例
classifier = CLIPClassifier()
categories = ['cat', 'dog', 'bird', 'car', 'tree']
results = classifier.classify('image.jpg', categories)

for result in results:
    print(f"{result['category']}: {result['probability']:.2%}")
```

---

## 图像检索

### 文本到图像检索

```python
"""
使用 CLIP 进行图像检索
"""
import torch
import clip
from PIL import Image
import os

class CLIPImageRetrieval:
    """CLIP 图像检索系统"""
    
    def __init__(self, model_name="ViT-B/32"):
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.model, self.preprocess = clip.load(model_name, device=self.device)
        self.image_features = None
        self.image_paths = []
    
    def index_images(self, image_dir):
        """
        索引图像库
        
        Args:
            image_dir: 图像目录
        """
        image_features_list = []
        
        for filename in os.listdir(image_dir):
            if filename.endswith(('.jpg', '.png', '.jpeg')):
                image_path = os.path.join(image_dir, filename)
                self.image_paths.append(image_path)
                
                # 加载并编码图像
                image = self.preprocess(Image.open(image_path)).unsqueeze(0).to(self.device)
                
                with torch.no_grad():
                    image_feature = self.model.encode_image(image)
                    image_feature /= image_feature.norm(dim=-1, keepdim=True)
                    image_features_list.append(image_feature)
        
        # 合并所有图像特征
        self.image_features = torch.cat(image_features_list, dim=0)
        print(f"已索引 {len(self.image_paths)} 张图像")
    
    def search(self, query_text, top_k=5):
        """
        根据文本查询检索图像
        
        Args:
            query_text: 查询文本
            top_k: 返回前 k 个结果
        """
        # 编码查询文本
        text = clip.tokenize([query_text]).to(self.device)
        
        with torch.no_grad():
            text_feature = self.model.encode_text(text)
            text_feature /= text_feature.norm(dim=-1, keepdim=True)
            
            # 计算相似度
            similarity = (100.0 * text_feature @ self.image_features.T).softmax(dim=-1)
        
        # 获取 top-k 结果
        values, indices = similarity[0].topk(top_k)
        
        results = []
        for value, index in zip(values, indices):
            results.append({
                'image_path': self.image_paths[index],
                'score': value.item()
            })
        
        return results

# 使用示例
retrieval = CLIPImageRetrieval()
retrieval.index_images('./images')

# 搜索图像
results = retrieval.search("a cute cat", top_k=5)
for i, result in enumerate(results, 1):
    print(f"{i}. {result['image_path']} (score: {result['score']:.4f})")
```

---

## 实战案例

### 图像相似度计算

```python
"""
计算两张图像的相似度
"""
import torch
import clip
from PIL import Image

def compute_image_similarity(image1_path, image2_path):
    """计算两张图像的相似度"""
    device = "cuda" if torch.cuda.is_available() else "cpu"
    model, preprocess = clip.load("ViT-B/32", device=device)
    
    # 加载图像
    image1 = preprocess(Image.open(image1_path)).unsqueeze(0).to(device)
    image2 = preprocess(Image.open(image2_path)).unsqueeze(0).to(device)
    
    # 编码图像
    with torch.no_grad():
        features1 = model.encode_image(image1)
        features2 = model.encode_image(image2)
        
        # 归一化
        features1 /= features1.norm(dim=-1, keepdim=True)
        features2 /= features2.norm(dim=-1, keepdim=True)
        
        # 计算余弦相似度
        similarity = (features1 @ features2.T).item()
    
    return similarity

# 使用示例
similarity = compute_image_similarity('cat1.jpg', 'cat2.jpg')
print(f"图像相似度: {similarity:.4f}")
```

### 多语言支持

```python
"""
CLIP 多语言图像分类
"""
def multilingual_classification(image_path):
    """多语言图像分类"""
    device = "cuda" if torch.cuda.is_available() else "cpu"
    model, preprocess = clip.load("ViT-B/32", device=device)
    
    # 加载图像
    image = preprocess(Image.open(image_path)).unsqueeze(0).to(device)
    
    # 多语言类别
    categories = {
        'en': ['cat', 'dog', 'bird'],
        'zh': ['猫', '狗', '鸟'],
        'ja': ['猫', '犬', '鳥']
    }
    
    results = {}
    
    for lang, cats in categories.items():
        text = clip.tokenize([f"a photo of a {c}" for c in cats]).to(device)
        
        with torch.no_grad():
            logits_per_image, _ = model(image, text)
            probs = logits_per_image.softmax(dim=-1).cpu().numpy()
        
        results[lang] = {
            cats[i]: float(probs[0][i])
            for i in range(len(cats))
        }
    
    return results

# 使用示例
results = multilingual_classification('image.jpg')
for lang, probs in results.items():
    print(f"\n{lang}:")
    for category, prob in probs.items():
        print(f"  {category}: {prob:.2%}")
```

---

## 最佳实践

### 提示工程

```python
"""
CLIP 提示工程技巧
"""

# 1. 使用描述性提示
good_prompts = [
    "a photo of a {category}",
    "a picture of a {category}",
    "an image of a {category}"
]

# 2. 添加上下文
contextual_prompts = [
    "a photo of a {category} in the wild",
    "a close-up photo of a {category}",
    "a {category} in its natural habitat"
]

# 3. 集成多个提示
def ensemble_classification(image, categories):
    """使用多个提示进行集成分类"""
    prompts = [
        "a photo of a {}",
        "a picture of a {}",
        "an image of a {}"
    ]
    
    all_probs = []
    
    for prompt_template in prompts:
        texts = [prompt_template.format(c) for c in categories]
        text_inputs = clip.tokenize(texts).to(device)
        
        with torch.no_grad():
            logits_per_image, _ = model(image, text_inputs)
            probs = logits_per_image.softmax(dim=-1)
            all_probs.append(probs)
    
    # 平均概率
    avg_probs = torch.stack(all_probs).mean(dim=0)
    
    return avg_probs
```

---

**记住：CLIP 是零样本学习的强大工具，合理使用提示工程可以显著提升效果！** 🎨

@author erik.zhou
