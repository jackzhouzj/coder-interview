# Whisper 完整教程

> @author erik.zhou

## 📋 目录

- [Whisper 基础](#whisper-基础)
- [语音识别](#语音识别)
- [多语言支持](#多语言支持)
- [实时转录](#实时转录)
- [实战案例](#实战案例)

---

## Whisper 基础

### 什么是 Whisper

```python
"""
Whisper 是 OpenAI 开发的自动语音识别系统
支持多语言、多任务（转录、翻译）
"""

# Whisper 特点
features = {
    "多语言": "支持99种语言",
    "鲁棒性": "对噪音和口音有很好的适应性",
    "多任务": "支持转录和翻译",
    "开源": "完全开源可商用"
}
```

### 安装配置

```bash
# 安装 Whisper
pip install openai-whisper

# 安装依赖
pip install torch torchaudio ffmpeg-python

# 或使用 Hugging Face 版本
pip install transformers
```

### 基础使用

```python
"""
Whisper 基础示例
"""
import whisper

# 加载模型
model = whisper.load_model("base")

# 转录音频
result = model.transcribe("audio.mp3")

print(result["text"])
```

---

## 语音识别

### 音频转录

```python
"""
使用 Whisper 进行音频转录
"""
import whisper
import torch

class WhisperTranscriber:
    """Whisper 转录器"""
    
    def __init__(self, model_size="base"):
        """
        初始化转录器
        
        Args:
            model_size: 模型大小 (tiny, base, small, medium, large)
        """
        self.device = "cuda" if torch.cuda.is_available() else "cpu"
        self.model = whisper.load_model(model_size, device=self.device)
    
    def transcribe(self, audio_path, language=None):
        """
        转录音频文件
        
        Args:
            audio_path: 音频文件路径
            language: 语言代码（如 'zh', 'en'）
        """
        options = {}
        if language:
            options['language'] = language
        
        result = self.model.transcribe(audio_path, **options)
        
        return {
            'text': result['text'],
            'language': result['language'],
            'segments': result['segments']
        }
    
    def transcribe_with_timestamps(self, audio_path):
        """带时间戳的转录"""
        result = self.model.transcribe(audio_path)
        
        segments = []
        for segment in result['segments']:
            segments.append({
                'start': segment['start'],
                'end': segment['end'],
                'text': segment['text']
            })
        
        return segments

# 使用示例
transcriber = WhisperTranscriber(model_size="base")

# 转录音频
result = transcriber.transcribe("audio.mp3", language="zh")
print(f"识别文本: {result['text']}")
print(f"语言: {result['language']}")

# 带时间戳的转录
segments = transcriber.transcribe_with_timestamps("audio.mp3")
for seg in segments:
    print(f"[{seg['start']:.2f}s - {seg['end']:.2f}s] {seg['text']}")
```

---

## 多语言支持

### 自动语言检测

```python
"""
Whisper 自动语言检测
"""
import whisper

def detect_language(audio_path):
    """检测音频语言"""
    model = whisper.load_model("base")
    
    # 加载音频
    audio = whisper.load_audio(audio_path)
    audio = whisper.pad_or_trim(audio)
    
    # 生成 mel 频谱图
    mel = whisper.log_mel_spectrogram(audio).to(model.device)
    
    # 检测语言
    _, probs = model.detect_language(mel)
    
    # 获取最可能的语言
    detected_language = max(probs, key=probs.get)
    
    return {
        'language': detected_language,
        'probability': probs[detected_language],
        'all_probs': dict(sorted(probs.items(), key=lambda x: x[1], reverse=True)[:5])
    }

# 使用示例
result = detect_language("audio.mp3")
print(f"检测到的语言: {result['language']} ({result['probability']:.2%})")
print("\n前5种可能的语言:")
for lang, prob in result['all_probs'].items():
    print(f"  {lang}: {prob:.2%}")
```

### 多语言翻译

```python
"""
使用 Whisper 进行语音翻译
"""
def translate_audio(audio_path, target_language="en"):
    """
    将音频翻译为目标语言
    
    Args:
        audio_path: 音频文件路径
        target_language: 目标语言（默认英语）
    """
    model = whisper.load_model("base")
    
    # 转录并翻译
    result = model.transcribe(
        audio_path,
        task="translate",  # 翻译任务
        language=None  # 自动检测源语言
    )
    
    return {
        'original_language': result['language'],
        'translated_text': result['text']
    }

# 使用示例
result = translate_audio("chinese_audio.mp3")
print(f"原始语言: {result['original_language']}")
print(f"翻译结果: {result['translated_text']}")
```

---

## 实时转录

### 流式音频处理

```python
"""
实时音频转录
"""
import pyaudio
import wave
import whisper
import numpy as np

class RealtimeTranscriber:
    """实时转录器"""
    
    def __init__(self, model_size="base"):
        self.model = whisper.load_model(model_size)
        self.chunk_duration = 5  # 每5秒处理一次
        self.sample_rate = 16000
    
    def transcribe_stream(self, duration=30):
        """
        实时转录音频流
        
        Args:
            duration: 录音时长（秒）
        """
        # 初始化 PyAudio
        p = pyaudio.PyAudio()
        
        stream = p.open(
            format=pyaudio.paInt16,
            channels=1,
            rate=self.sample_rate,
            input=True,
            frames_per_buffer=1024
        )
        
        print("开始录音...")
        
        frames = []
        chunk_size = int(self.sample_rate * self.chunk_duration)
        
        for i in range(0, int(self.sample_rate / 1024 * duration)):
            data = stream.read(1024)
            frames.append(data)
            
            # 每隔一定时间处理一次
            if len(frames) * 1024 >= chunk_size:
                # 转换为 numpy 数组
                audio_data = np.frombuffer(b''.join(frames), dtype=np.int16)
                audio_data = audio_data.astype(np.float32) / 32768.0
                
                # 转录
                result = self.model.transcribe(audio_data)
                print(f"\n转录结果: {result['text']}")
                
                # 清空缓冲区
                frames = []
        
        stream.stop_stream()
        stream.close()
        p.terminate()
        
        print("\n录音结束")

# 使用示例
transcriber = RealtimeTranscriber()
transcriber.transcribe_stream(duration=30)
```

---

## 实战案例

### 视频字幕生成

```python
"""
为视频生成字幕
"""
import whisper
from datetime import timedelta

def generate_subtitles(video_path, output_path):
    """
    生成 SRT 字幕文件
    
    Args:
        video_path: 视频文件路径
        output_path: 输出字幕文件路径
    """
    model = whisper.load_model("base")
    
    # 转录视频
    result = model.transcribe(video_path)
    
    # 生成 SRT 格式字幕
    with open(output_path, 'w', encoding='utf-8') as f:
        for i, segment in enumerate(result['segments'], 1):
            # 时间格式化
            start_time = str(timedelta(seconds=segment['start']))
            end_time = str(timedelta(seconds=segment['end']))
            
            # 写入字幕
            f.write(f"{i}\n")
            f.write(f"{start_time} --> {end_time}\n")
            f.write(f"{segment['text'].strip()}\n\n")
    
    print(f"字幕已保存到: {output_path}")

# 使用示例
generate_subtitles("video.mp4", "subtitles.srt")
```

### 会议记录

```python
"""
会议录音转文字记录
"""
import whisper
from datetime import datetime

class MeetingTranscriber:
    """会议转录器"""
    
    def __init__(self, model_size="medium"):
        self.model = whisper.load_model(model_size)
    
    def transcribe_meeting(self, audio_path, output_path):
        """
        转录会议录音并生成文档
        
        Args:
            audio_path: 音频文件路径
            output_path: 输出文档路径
        """
        print("正在转录会议录音...")
        
        # 转录音频
        result = self.model.transcribe(
            audio_path,
            language="zh",
            verbose=True
        )
        
        # 生成会议记录
        with open(output_path, 'w', encoding='utf-8') as f:
            f.write(f"# 会议记录\n\n")
            f.write(f"日期: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n\n")
            f.write(f"## 完整内容\n\n")
            f.write(f"{result['text']}\n\n")
            f.write(f"## 分段内容\n\n")
            
            for i, segment in enumerate(result['segments'], 1):
                start_time = f"{int(segment['start'] // 60):02d}:{int(segment['start'] % 60):02d}"
                f.write(f"**[{start_time}]** {segment['text']}\n\n")
        
        print(f"会议记录已保存到: {output_path}")
        
        return result

# 使用示例
transcriber = MeetingTranscriber()
transcriber.transcribe_meeting("meeting.mp3", "meeting_notes.md")
```

---

## 最佳实践

### 模型选择

```python
"""
Whisper 模型选择指南
"""

models = {
    "tiny": {
        "参数量": "39M",
        "速度": "最快",
        "准确率": "较低",
        "适用场景": "实时转录、资源受限"
    },
    "base": {
        "参数量": "74M",
        "速度": "快",
        "准确率": "中等",
        "适用场景": "一般应用"
    },
    "small": {
        "参数量": "244M",
        "速度": "中等",
        "准确率": "较高",
        "适用场景": "高质量转录"
    },
    "medium": {
        "参数量": "769M",
        "速度": "较慢",
        "准确率": "高",
        "适用场景": "专业转录"
    },
    "large": {
        "参数量": "1550M",
        "速度": "慢",
        "准确率": "最高",
        "适用场景": "最高质量要求"
    }
}

def choose_model(priority="balanced"):
    """
    根据优先级选择模型
    
    Args:
        priority: 优先级 (speed, accuracy, balanced)
    """
    if priority == "speed":
        return "tiny"
    elif priority == "accuracy":
        return "large"
    else:
        return "base"
```

### 音频预处理

```python
"""
音频预处理技巧
"""
import librosa
import soundfile as sf

def preprocess_audio(input_path, output_path):
    """
    预处理音频以提高识别准确率
    
    Args:
        input_path: 输入音频路径
        output_path: 输出音频路径
    """
    # 加载音频
    audio, sr = librosa.load(input_path, sr=16000)
    
    # 降噪
    audio = librosa.effects.preemphasis(audio)
    
    # 归一化
    audio = librosa.util.normalize(audio)
    
    # 保存处理后的音频
    sf.write(output_path, audio, sr)
    
    return output_path

# 使用示例
processed_audio = preprocess_audio("noisy_audio.mp3", "clean_audio.wav")
```

---

**记住：Whisper 是强大的语音识别工具，选择合适的模型和预处理可以显著提升效果！** 🎤

@author erik.zhou
