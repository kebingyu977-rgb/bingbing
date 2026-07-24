# ASR管线集成说明

西南欧复盘V2复用 `international-video-audit` 的完整ASR管线。

## 核心命令

```bash
# 1. 音频提取
ffmpeg -i '/tmp/video.mp4' -vn -acodec pcm_s16le -ar 16000 -ac 1 '/tmp/audio.wav'

# 2. 上传音频
python3 /opt/data/share/skills/file-storage/scripts/upload_file.py /tmp/audio.wav

# 3. ASR识别 — 必须用 chat create
coral-cli chat create \
  --appId '2077296142100779009' \
  --env 'test' \
  --access-key 'cli-k5SiCaaa7VcCWHi7O3S1qqh9jEHU2K9Lt8Vd2mVUDsfqGdca' \
  --url '<audio_url>'

# 4. 抽帧
ffmpeg -i '/tmp/video.mp4' -vf "fps=2,scale=640:-1:flags=lanczos" /tmp/frames/frame_%04d.jpg -y
```

## ASR失败模式

| 场景 | 判定 | 处理 |
|------|------|------|
| 超时 | asr_success: false | 纯视觉分析，不扣分 |
| 空音频 | asr_success: false | 标注"无口播" |
| 乱码(gibberish) | asr_success: false | 标注原因，纯视觉分析 |
| 纯字幕无语音 | asr_success: false | 标注"无口播" |

## ASR↔Vision交叉验证

- 卖点ASR提及+画面有演示 → ✅ 完整证据
- 卖点ASR提及+画面无演示 → ⚠️ 口播未可视化，优化机会
- 卖点画面有演示+ASR未提及 → ⚠️ 信息传递不完整
- 卖点ASR未提及+画面无演示 → ❌ 缺失

## 评分影响

- brief_alignment / persuasion 等维度：有视觉证据就给正常分，仅ASR文本无视觉对应才下调
- ASR失败 → 不扣分 → 分数应上升而非下降
