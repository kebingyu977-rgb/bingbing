---
name: southwestern-eu-campaign-review-v4
description: 西南欧线下市场KOL投放复盘评估技能V4。整合KPI基准线(tineco-kpi-baseline)+视频内容评分(纯视觉4维:Brief匹配度4分+画面演示3分+钩子1.5分+节奏1.5分)+达人性价比横评(性价比指数=内容分÷总花费€)+高频帧分析(2fps=每0.5秒1帧，30秒抽60帧)，按投放流(0-100)+内容流(0-10)双轨评估。⚠️ V4修正：性价比改用内容分÷总花费€直接公式；高频帧升级2fps；HTML采用优化版5模块结构+完整CSS(verdict/crx/video-card/mm/action-item选择器)，crx-c总结卡片用独立白底圆角样式+🎯前缀。适用DE/FR/IT三市场。
---

# 西南欧线下KOL投放复盘 V4

## 简介

西南欧线下市场（IT/ES/PT等）KOL投放复盘标准化管线V4。双轨并行架构：
- **投放流**：Excel投放数据 → KPI基准线(tineco-kpi-baseline) → 投放分(0-100) → 逐placement消耗加权
- **内容流**：Brief+视频 → 纯视觉4维评分(Brief匹配度4分+画面演示3分+钩子1.5分+节奏1.5分) → 内容分(0-10) → 逐视频费用加权
- **达人性价比横评**：性价比指数=内容分÷总花费€（直接公式，更直观）
- **高频帧分析**：2fps（每0.5秒1帧，30秒抽60帧），替换0.5fps避免漏检关键动作（折叠/CTA/尾帧仅持续2-3秒）
- **HTML版式**：浅色白底(#f5f7fa)+深色文字(#222)+标题(#1a1a2e)，替代深色背景版式
- **⚠️ V4核心变更**：性价比改用内容分÷总花费€直接公式；高频帧升级2fps；HTML浅色白底+标题深色。彻底移除ASR维度(用户确认准确率不足)，回归纯画面证据。ASR仅用于内部理解，不得写入报告。

最终输出JSON schema数据 → 单文件HTML报告(6模块，白底浅色诊断版式)。

## 触发条件

- 用户要求复盘西南欧市场KOL投放（意大利/法国/西班牙/葡萄牙/德国）
- 提供投放数据Excel(XLSX) + 视频文件(CDN链接或本地)
- 提供Brief文件(PPTX/PDF/DOCX)
- 明确要求生成Campaign Review评估报告
- 关键词：复盘/评估/Campaign Review/KOL复盘/投放复盘/视频分析/西南欧

**两种工作模式**：
1. **完整报告模式**（6模块）：有Excel投放数据+Brief+视频，生成完整S0-S5报告
2. **单视频深度分析模式**：用户提供视频CDN链接+Brief，生成单条视频内容评分HTML

## 输入物清单

| 输入物 | 格式 | 必需 | 来源 |
|--------|------|------|------|
| 投放数据 | XLSX | 是 | 含placement明细(CPC/CTR/CPM/Views/Clicks/Spending/Platform/达人) |
| Brief文件 | PPTX/PDF/DOCX | 是 | 含必做项/禁止项/加分项 |
| 视频文件 | MP4 | 是 | 每条达人视频(CDN链接或本地) |
| 达人成交价 | Excel列或手动 | 是 | 决定内容分及格线 |
| 加热CTR | Excel列或手动 | 否 | 无则跳过评级，只出内容分 |

## 数据管线

### Phase 1: Excel投放数据解析（投放流）

```python
# 安全解析Excel XML（处理#DIV/0!等公式错误）
def safe_float(val):
    if val is None or val == '' or (isinstance(val, str) and val.startswith('#')):
        return None
    try: return float(val)
    except: return None
```

**铁律**：
- `#DIV/0!` 等公式错误必须跳过
- `row r` 属性 ≠ 列表索引，必须逐行扫描 `row.get('r')`
- 汇总指标一律消耗加权(Σspend ÷ Σclicks)，不得简单平均

### Phase 2.5: 深度版判定标准（默认输出，非可选项）

**⚠️ 管理层讨论不写入报告**：用户提出的策略讨论问题（如"30秒能不能讲清产品"）仅在对话中讨论，**不写入HTML报告**。报告保持纯净的复盘结论+数据。

用户说"不太行呀"或"深度问题"时，意味着当前输出是"评分汇总级"，缺少：
1. **秒级时间线** — 每条视频按5-10秒分段，标注每段时间、内容摘要、ASR原文、视觉画面
2. **ASR↔Vision交叉验证** — 每项判定必须同时引用ASR台词原文+视觉帧号(frame_NNN)，不能只写"✅有痛点"
3. **Brief逐条证据链** — 每项Must-Have标注fulfilled/partially/not_fulfilled + ASR原文 + 视觉帧号 + Brief覆盖率百分比
4. **帧级证据** — 引用具体帧号(frame_0001/frame_0050等)，说明画面内容而非抽象描述
5. **5项检查详细依据** — 不只写"✅/⚠️/❌"，必须写出具体ASR台词和视觉画面证据

**深度版是默认输出**，不是可选项。只要用户提供视频审核JSON结果或视频CDN链接，必须输出深度版（含时间线+ASR交叉+Brief逐条证据链）。仅当用户只给Excel投放数据、无任何视频内容时，才输出标准版(S0-S5模块，无逐视频深度分析)。

**5项检查的"依据"字段铁律**：
- 禁止抽象描述（"有痛点""原生感强""内容结构可复制"）
- 必须引用ASR台词原文（ASR成功时）或明确标注"ASR不可用(原因)，仅视觉"
- 必须引用视觉帧号（frame_0001/frame_0050等）并描述具体画面内容
- 每项依据格式：`ASR:"..." + 视觉:frame_NNN:画面描述` 或 `ASR不可用(原因) + 视觉:frame_NNN:画面描述`

### Phase 2: KPI评估 → 投放分(0-100)

**数据清洗前置（必须在分目标/分平台之前执行）**：
1. **CTR > 20% = 口径异常**：clicks(all)不是link clicks，剔除该placement
2. **clicks > impressions**：clicks口径错误，剔除
3. FR 8场清洗后：FB=37条、IG=42条，共79条有效placement

**KPI基准线生成流程**（用真实数据分布，拒绝Agency宽松线）：
1. 清洗CTR>20%异常 → 计算P25/P50/P75分布 → P25=Excellent, P50=Good, P75=Pass, >P75=Poor
2. **FR 8场79条分布为唯一基准，三国统一，不做市场溢价系数调整**
3. 德国CPC图片标准（Ex≤€0.15）比FR IG P25=€0.11还宽松，是Agency放水，已废弃，不做锚点
4. DE/IT = FR基准（无证据表明更贵或更便宜，不做放宽）
5. TikTok推算：CPC=IG×0.7, CTR=IG×1.3, CPM=IG×0.6

**FR 8场分布（清洗后，P25/P50/P75）**：
FB: CPC 0.06/0.08/0.10 | CTR 5.2%/3.7%/2.9% | CPM 2.62/2.99/3.76
IG: CPC 0.11/0.12/0.15 | CTR 5.9%/4.6%/3.1% | CPM 4.22/4.95/6.64

**判定顺序：先清洗 → 再分目标 → 再分平台 → 再套阈值 → 最后加权成分。**

**⚠️ CPC口径陷阱（严重）**：KOL organic投放（非traffic广告）的sessions/进店量是自然流量而非付费link clicks。此时CPC会严重超标（€3-11 vs 标准€0.15），**不应判CPC**。改用eCPM/ER/播放-粉丝比等KOL指标。判定方法：投放目标=Organic KOL → 不判CPC；投放目标=Traffic → 正常判CPC；无法确定 → 标注"CPC口径不可用"，用eCPM+ER替代。

详见 `references/kpi-baseline-data-driven.md`。

#### 步骤1：分目标类型

| 目标 | 判什么 | 不判什么 |
|------|--------|---------|
| Traffic / Link Click | CPC / CTR / CPM | — |
| Reach | CPM / Frequency | CPC / CTR 不适用 |
| Organic KOL | eCPM / 播放-粉丝比 / 播放-均播比 | CPC / CTR / CPM |

**不可混用**。Reach类不判CPC/CTR。

#### 步骤2：分平台判阈值（引流Traffic）

**FR/DE/IT 三国统一标准**（基于FR 8场79条分布，不做市场溢价调整）

| 平台 | 指标 | 优秀(100) | 好(88) | 达标(78) | 差(50) |
|------|------|-----------|--------|----------|--------|
| FB | CPC | ≤€0.06 | 0.06-0.08 | 0.08-0.10 | >€0.10 |
| FB | CTR | ≥5.2% | 3.7-5.2% | 2.9-3.7% | <2.9% |
| FB | CPM | ≤€2.62 | 2.62-2.99 | 2.99-3.76 | >€3.76 |
| IG | CPC | ≤€0.11 | 0.11-0.12 | 0.12-0.15 | >€0.15 |
| IG | CTR | ≥5.9% | 4.6-5.9% | 3.1-4.6% | <3.1% |
| IG | CPM | ≤€4.22 | 4.22-4.95 | 4.95-6.64 | >€6.64 |

**⚠️ 三国不区分系数**：DE CPC图片标准（Ex≤€0.15）比FR IG P25=€0.11更宽松，是Agency放水。IT线下市场虽小于FR，但投放效率逻辑相同。用户原话："别被牵着鼻子走，给客观标准"。三国统一用FR分布。

**TikTok（推算，基于IG系数）**

| 指标 | 优秀 | 好 | 达标 | 差 |
|------|------|----|------|----|
| CPC | ≤€0.08 | 0.08-0.08 | 0.08-0.10 | >€0.10 |
| CTR | ≥7.7% | 6.0-7.7% | 4.0-6.0% | <4.0% |
| CPM | ≤€2.53 | 2.53-2.97 | 2.97-3.98 | >€3.98 |

#### 步骤3：达人自然内容(Organic KOL)

**三国统一KOL指标基准**（基于FR IG分布）

| 指标 | 优秀 | 好 | 达标 | 差 |
|------|------|----|------|----|
| eCPM | ≤€4.22 | 4.22-4.95 | 4.95-6.64 | >€6.64 |
| ER(=(L+C)/Views) | ≥5.9% | 4.6-5.9% | 3.1-4.6% | <3.1% |
| 播放/粉丝比 | ≥150% | 100-150% | 50-100% | <40% |
| 实际播放÷近10条均播 | ≥200% | 120-200% | 80-120% | <60% |

**红线(一票否决)**
- 单达人eCPM > €10 → 无效投放，不复投
- 单达人实际播放 < 近10条均播的60% → 选人失误
- 视频出现竞品 → P0，独立追责
- 无专属追踪链接或折扣码 → 点击率按"差"计

#### 步骤4：档位赋分与投放分计算

| 档位 | 赋分 |
|------|------|
| 优秀 | 100 |
| 好 | 88 |
| 达标 | 78 |
| 差 | 50 |

**指标权重**（无ROAS时）：CPC 40% · CTR 35% · CPM 25%

**投放分** = Σ(各placement得分 × 该placement消耗) ÷ Σ总消耗

**旺季系数**（Nov-Dec）：FB CPC/CPM ×1.10，IG ×1.30，CTR不调整。

**扣分项**（总分基础上直接扣）

| 情形 | 扣分 |
|------|------|
| "差"档placement预算占比>30% | −5 |
| "差"档placement预算占比>50% | −10 |
| 无投前书面KPI | −10 |
| 数据口径不可复现 | −10 |
| 竞品出镜/素材未过审上线 | −15 |

**等级**：A ≥85 | B 70-84 | C 60-69 | D <60

### Phase 3: Brief解析（内容流）

**PPTX解析**：curl下载 → unzip解压 → ppt/slides/slide*.xml XML遍历提取文本。
**DOCX解析**：zipfile → word/document.xml ET提取。

提取结构：
- Must-Have KSPs（必做项）
- Prohibited Items（禁止项）
- Nice-to-Have（加分项）
- CTA要求（必须包含哪些行动引导）

### Phase 4: 视频分析 → ASR管线 + Vision

**复用international-video-audit管线**：

```bash
# 1. 下载视频
cd /tmp && curl -k -o 'video_N.mp4' 'CDN_URL'

# 2. 元数据
ffprobe -v quiet -print_format json -show_format -show_streams '/tmp/video_N.mp4'

# 3. 音频提取 + ASR（必须用chat create）
ffmpeg -i '/tmp/video_N.mp4' -vn -acodec pcm_s16le -ar 16000 -ac 1 '/tmp/audio_N.wav'
python3 /opt/data/share/skills/file-storage/scripts/upload_file.py /tmp/audio_N.wav
coral-cli chat create \
  --appId '2077296142100779009' \
  --env 'test' \
  --access-key 'cli-k5SiCaaa7VcCWHi7O3S1qqh9jEHU2K9Lt8Vd2mVUDsfqGdca' \
  --url '<audio_url>'

# 4. 抽帧（2fps=每0.5秒1帧，640px宽，30秒视频抽60帧）
ffmpeg -i '/tmp/video_N.mp4' -vf "fps=2,scale=640:-1:flags=lanczos" /tmp/frames_N/frame_%04d.jpg -y

# 5. Vision分析（15-25帧：首5帧+间隔帧+尾5帧）
```

**ASR+Vision交叉验证**：
- 卖点是否ASR口播中提到了？
- 画面是否有对应演示？
- ASR提到但画面缺失 = 优化机会
- 画面演示但ASR未提 = 信息传递失败

**ASR失败处理**：
- 超时/空音频 → `asr_success: false`
- 乱码（非目标语言字符） → `asr_success: false`，标注gibberish原因
- 纯字幕无口播 → `asr_success: false`
- 降级为纯视觉分析，**不扣分**

**P0/P1风险铁律**：P0=模型识别不清，P1=创意空间。评分时从风险清单移除，**不扣分**。

### Phase 5: 内容评估 → 内容分(0-10)（⚠️ V3修正：移除ASR维度）

**核心变更**：用户确认ASR准确率不足，**彻底从内容评分和报告中移除ASR**。回归纯画面证据。ASR仅用于内部理解，不得写入报告。

#### 4维评分矩阵（满分10分，不等权）

| # | 维度 | 权重 | 判定标准 | 扣分逻辑 |
|---|------|:----:|----------|----------|
| 1 | **Brief匹配度** | **4.0分** | 逐条核对 Must-Have 卖点清单 | 全满足=4分；部分满足(有口播无画面)=2分；完全缺失=0分 |
| 2 | **画面演示** | **3.0分** | 纯视觉证据验证卖点演示质量 | 混合垃圾实测=3分；单一垃圾=2分；空机滑动=1分；口播无画面=0分 |
| 3 | **钩子** | **1.5分** | 开场3秒悬念/冲突/好奇心 | 悬念钩子=1.5分；生活化=1.0分；平淡=0.5分 |
| 4 | **节奏** | **1.5分** | 时长/信息密度/卖点节奏 | ≤30秒聚焦3卖点=1.5分；30-60秒分散=1.0分；>60秒冗长=0.5分 |

**画面演示判定铁律**：
- "混合垃圾实测" = 同时演示多种垃圾类型(狗粮+毛发+番茄酱+咖啡)
- "单一垃圾" = 仅演示一种垃圾
- "空机滑动" = 机器在动但无垃圾吸入画面
- "口播无画面" = ASR提到卖点但画面没对应演示(用户原话："ASR不是很灵光，不然不合理")

**内容评级对照表**：
| 分数 | 评级 | 处置 |
|:---:|:---:|------|
| ≥8.5 | A(优质) | 扩量/续约 |
| 7.0-8.4 | B(达标) | 微调后发布 |
| 5.0-6.9 | C(警告) | 打回重拍/扣款 |
| <5.0 | D(劣质) | 废弃/拒付 |

**CTA红线**：CTA/Hashtag/@mention全缺 → 无论内容分多高，内容分上限4分（无法转化归因）。

**情景剧降权**：产品演示<40%的达人Brief合规率通常<30%，内容分降权处理。

### Phase 5.5: 达人性价比横评

**核心公式**：性价比指数 = 内容分 ÷ 总花费(€)

| 性价比指数 | 内容分门槛 | 判定 | 操作 |
|-----------|:---------:|------|------|
| ≥ 0.020 | ≥ 7.0分 | 高性价比 | 续约，可上浮10%定价 |
| ≥ 0.005 < 0.020 | ≥ 7.0分 | 中等性价比 | 继续合作，控价 |
| ≥ 0.003 < 0.005 | ≥ 7.0分 | 偏低性价比 | 需要控价+修改 |
| 任意 | < 7.0分 | 低质(性价比无效) | 直接剔除，性价比指数无效 |

**判定规则**：
1. 性价比 = 内容分 ÷ 总花费(€) — 直接公式，越高效越好
2. 内容分<7分的内容性价比指数无效（低质内容即使便宜也不复用）
3. 每条视频标题行必须显示：`| 内容分: X | 预算: €Y | 性价比: Z`
4. 性价比排名从高到低排列，方便快速识别最优达人

**⚠️ V4修正**：废弃eCPM40%+ER30%+内容20%+成交价10%的加权公式。用户实测验证后确认内容分÷总花费€更直观有效。

**HTML展示要求**：
- 效率横评表包含7列：达人 | 时长 | 内容分 | 总花费€ | 性价比 | 评级 | 建议
- 性价比列加粗+颜色编码：绿色(≥0.005) / 红色(<0.005)
- 每条视频深度分析卡片的h4标题行追加：`| 内容分: X | 预算: €Y | 性价比: Z`

## 输出格式

| 价格档 | 及格线 |
|--------|--------|
| 寄样/白嫖 | ≥5.5 |
| €100-500 | ≥6.5 |
| €500-2,000 | ≥7.5 |
| >€2,000 | ≥8.5 |

同一把尺子，不同过线高度。贵的代价体现在及格线上，不体现在检查项上。

#### 风险提示（单列，不进分数）

| 项 | 判定 |
|----|------|
| 竞品露出 | 无意入镜→记录归因brief交底不到位，不归零；有意对比→brief写了=正常评分，没写→P1追责 |
| 广告标注 | #ad/Sponsorizzato缺失→P0（欧盟三国强制） |
| 过度承诺/比较广告 | 未过法务确认即上线→P1 |
| BGM/素材版权 | 未授权影视片段/流行音乐→P1 |

#### 评级（内容分 × 加热后CTR）

**爆款口径**：CTR ≥ FB 7.0% / IG 8.0%（三国统一）

| | CTR达优秀 | CTR未达 |
|---|-----------|---------|
| **内容分≥及格线** | 🔥**S爆款** — 结构进模板库，下一场重新brief给同类达人 | 👍**A素材没问题，是投放问题** — 追agency受众/版位/预算分配 |
| **内容分<及格线** | ⚠️**B侥幸** — 达人自带流量或踩红利，不可复制，不进模板库 | 💀**C双失** — 追brief质量+选人 |

没有加热CTR的视频只出内容分，不评级。

### Phase 6: 活动级综合

**内容分** = Σ(单条内容分 × 该条费用) ÷ Σ费用
**活动及格线** = Σ(该条及格线 × 该条费用) ÷ Σ费用

必须费用加权，不能简单平均。

**投放分与内容分交叉出活动结论**（两个分不合成一个数）：

| | 内容≥线 | 内容<线 |
|---|---------|---------|
| **投放≥70** | 🔥 可复制 — 结构进模板库，下一场放量 | 👍 买量能力OK，资产薄弱 — 追brief+选人 |
| **投放<70** | ⚠️ 素材好被投烂 — 追受众/版位/预算 | 💀 双失 |

## 输出格式

### REPORT_DATA JSON（Schema校验，无自由文本字段）

```json
{
  "campaign": {
    "market": "IT|FR|DE",
    "activity_name": "...",
    "total_spend": 0.0,
    "placement_count": 0,
    "video_count": 0
  },
  "placement_stream": {
    "score_0_100": 0.0,
    "rating": "A|B|C|D",
    "by_platform": {
      "FB": {"cpc": 0.0, "ctr": 0.0, "cpm": 0.0, "score": 0},
      "IG": {"cpc": 0.0, "ctr": 0.0, "cpm": 0.0, "score": 0}
    },
    "placements": [
      {
        "creator": "...",
        "platform": "FB|IG",
        "objective": "Traffic|Reach|Organic",
        "spend": 0.0,
        "cpc": 0.0, "ctr": 0.0, "cpm": 0.0,
        "tier": "Excellent|Good|Average|Poor",
        "placement_score": 0
      }
    ],
    "deductions": [
      {"reason": "...", "amount": -5}
    ]
  },
  "content_stream": {
    "score_0_10_fee_weighted": 0.0,
    "pass_line_fee_weighted": 0.0,
    "videos": [
      {
        "content_id": "...",
        "creator": "...",
        "price_tier": "寄样|€100-500|€500-2000|>€2000",
        "pass_line": 0.0,
        "checks": {
          "hook_3s": {"判定": "✅|⚠️|❌", "得分": 0, "依据": "..."},
          "silent_comprehension": {"判定": "✅|⚠️|❌", "得分": 0, "依据": "..."},
          "brief_execution": {"判定": "✅|⚠️|❌", "得分": 0, "依据": "..."},
          "native_feel": {"判定": "✅|⚠️|❌", "得分": 0, "依据": "..."},
          "replicability": {"判定": "✅|⚠️|❌", "得分": 0, "依据": "..."}
        },
        "content_score": 0.0,
        "passed": true,
        "ctr_heated": 0.0,
        "rating": "S|A|B|C|null",
        "risk_items": [],
        "content_type": "剧情向|测评向|氛围向|vlog"
      }
    ]
  },
  "cross_attribution": {
    "placement_score": 0.0,
    "content_score_weighted": 0.0,
    "content_pass_line_weighted": 0.0,
    "activity_conclusion": "可复制|买量OK资产薄弱|素材好被投烂|双失"
  },
  "actions": [
    {"priority": "P0|P1|P2", "action": "...", "target": "...", "deadline": "..."}
  ]
}
## 输出格式

### 5模块HTML报告（优化版结构，2026-07-23生效，永久锁定）

**⚠️ 5模块独立section结构（最高优先级）**：报告采用5模块结构，每个模块为独立`<div class="section">`，模块标题H2内嵌评分badge。

| 模块 | 结构 | 内容 | 数据来源 |
|------|------|------|---------|
| [总览评分卡] | 无section包裹，顶层H1开头 | `.verdict`综合评分盒 + `.vbars`投放/内容/预算三维度进度条 + `.alert-box`整体表现段落 + `.kpi-grid`7列指标卡片 | 加权计算+Excel汇总 |
| **M1** | `<h2>.h2score`内嵌82分徽章 | `.mini`状态卡 + `.card`KPI基准线表 + `.card`逐条判定表（含达标线/本轮/判定） | Phase 2 KPI判定 |
| **M2** | `<h2>.h2score`内嵌70分徽章 | `.mini`状态卡 + `.card`达人逐条质检表(Brief4+画面3+钩子1.5+节奏1.5=10) + `.card`达人逐条视频深度内容分析(crx卡片：总分/预算占比/Brief/画面/钩子/节奏逐条评价) | Phase 3 内容流 |
| **M3** | `<h2>.h2score`内嵌45分徽章 | `.mini`状态卡 + `.card`预算流向vs内容质量矩阵(`.mm`表格) + `.card`性价比横评判定规则 + `.card`逐达人性价比计算(性价比指数=内容分÷总花费€) + `.card`曝光量趋势分析 | Phase 4 投放诊断 |
| **M4** | `<h2>`无评分徽章 | `.mini`状态卡 + `.action-item .action-p0/p1/p2`行动项(2-3条P0紧急/2-3条P1重要/2-3条P2储备) | Phase 5 行动指导 |

**⚠️ 5模块严格独立**：M1/M2/M3/M4之间用`<!-- MODULE X -->`注释分隔，**禁止模块间内容交叉**（如投放数据不写入M2内容模块，达人内容不写入M3投放诊断）。

#### 整体表现段落撰写规范（永久锁定，2026-07-23最终定稿）

**位置**：总览评分卡末尾，`.alert-box.alert-warn`组件内，标题为`📌 整体表现`。

**结构**：一段定性数据 + 一句话结论 + 量化测算（三块，不是三句话车轳转）。

**最终定稿格式（直接套用，2026-07-23定稿）**：
```
投放侧效率数据拉满——CPC优于标准X%、CTR优于Y%——但这是流量买得好，不是内容做得好。
内容侧综合Z分刚好踩在剔除线上，€TOTAL里有€AMOUNT（PCT%）砸给了不及格的XXX和XXX。
效率冠军YYY（€YYY→X.X分）只分到PCT%。

<一句话定性：流量买对了，内容没做对，钱花错了。>

量化测算：XXX+XXX合计€AMOUNT（占PCT%），两组实际CPC均值€A。若这笔钱按YYY/ZZZ的实际CPC（€B）投放，可多获得约N次点击（+X%），等值媒体价值约€M。
```

**⚠️ verdict评分区块干净铁律**：verdict盒内只放综合评分+等级+加权公式，详细分析段落+三根进度条(vbars)全部放在alert-box"整体表现"段落内。

**撰写铁律（每条都是用户否决后得出的教训）**：
- **禁止"投放兜住了内容"等自我开脱表述**（用户否决v1）
- **三句话必须是三个独立维度**，不能一句话车轱辘转三遍。"流量买对了→内容没做对→钱花错了"是同一意思说三遍，不是三句话（用户否决v2）
- **不能前后矛盾**。"投放做对了"和"预算搞反了"是矛盾的，因为预算分配就是投放的一部分（用户否决v3）
- **"出价"术语不准确**。投放效率的CPC/CTR/CPM不是竞价bid，用"投放效率达标"（用户否决v4）
- **语气犀利，直指矛头**。短句子，有数据支撑，不绕弯子
- **量化测算必须有**：具体金额+占比+CPC差异+机会成本
- **verdict评分区块保持干净**。评定结论+加权公式即可，详细段落+进度条放alert-box（用户否决把详细段落塞进verdict）

**HTML模板**：`assets/fr-campaign-template-v4-final.html`（49,898 chars，包含完整CSS 10,545 chars + 81个CSS类名）

- 新结构：[总览评分卡] → M1 KPI判定 → M2 内容分析(质检+视频合并) → M3 投放诊断 → M4 行动指导
- M2将原来的M3(质检表)和M3.5(视频深度)合并为一个模块内的两个H3子块
- M3增加了「预算流向vs内容质量矩阵」和「曝光趋势」两个新子模块
- M4行动指导改用`.action-item`/`.action-p0/p1/p2`样式

**⚠️ crx总结卡片样式（2026-07-23最终版）**：`.crx-c`总结必须用独立卡片样式（`font-size:14px; background:#fff; border:1px solid; border-left:4px solid var(--accent); border-radius:8px; box-shadow:0 1px 3px rgba(0,0,0,0.04);`），和下面逐帧表格区分开。禁止用原来的浅灰底+小字号。前缀加🎯。用户原话："总结那个模块的呈现可不可以更显眼一点"。

**⚠️ 禁止"体检报告"式结构（P0铁律）**：用户明确否决各模块前加"模块小结"卡片(评分/标准/问题)。每个模块直接以`<h2>`标题展开即可，无需前置小结。

**模板文件（最高优先级）**：最终模板为 `assets/fr-campaign-template-v4-final.html`（41,338 chars）。**⚠️ 模板优先铁律（P0）**：当用户提供参考HTML模板时，必须将该文件作为完整骨架（CSS+DOM结构100%原样保留），仅替换数据内容。严禁自行用旧CSS重新拼装、严禁部分复用CSS、严禁"复刻"——CSS组件之间有关联依赖（如crx-h/crx-b/crx-g联动，vb/vb-t/vb-f联动，verdict-score/verdict-body/vbars联动），缺少任何一个选择器都会导致渲染断裂。**verdict评分区块保持干净**：只放综合评分+等级+加权公式，详细段落和进度条放alert-box组件内。正确做法：curl下载参考文件 → 用Python正则/BeautifulSoup定位数据节点 → 逐节点替换数据 → 保持<head>完整不变。

**⚠️ 版式重构禁区**：不要自行发明新的版式结构。严格遵循5模块结构+CSS。

**总览评分卡标准结构**：
- **H1标题**：`🇫🇷 法国 I7/S7 Fold 活动评估报告` + subtitle（时间线+预算）
- **verdict盒**：`<div class="verdict">`含vk(综合评分标签)/vn(分数，36px)/vg(等级，如C+)/vd(满分基准+复投线)
- **KPI指标网格**：`<div class="kpi-grid">` + `<div class="kpi-card">` × 7列（总花费/总曝光/总互动/CTR/CPM/CPE/CPC）
- **核心问题（宏观）**：`<div class="alert-box alert-bad">`列出P0级问题
- **亮点**：`<div class="alert-box alert-ok">`列出可复用资产
- **决胜武器**：表格(武器|为什么有效|落地动作|验证指标)

## 版式铁律

**优化版CSS（2026-07-23生效）**：完整CSS见 `assets/optimized-css.txt`（10,545 chars）。以下为核心选择器：

| 选择器 | 用途 | 说明 |
|--------|------|------|
| `.verdict` | 综合评分盒 | flex布局+左侧6px红色边框+vk/vn/vg/vd四分区 |
| `.h2score` | H2内嵌评分badge | `<span class="h2score badge-xx">82 分</span>`，badge-ok/warn/bad三级 |
| `.crx` | 达人效率卡片 | crx-h(id+平台) + crx-m(分数+预算+性价比) + crx-bar(进度条) + crx-c(独立总结卡片:白底+圆角+阴影+🎯) |
| `.video-card` | 视频卡片 | video-title + dimension-grid(四维评分) + asr-box(配音文本) |
| `.mm` | 模块小结/矩阵表 | mm-r(行) + mm-n(名) + mm-b(预算) + mm-sc(分数) + mm-v(判定) |
| `.action-item` | 行动项 | action-p0(红色左框)/action-p1(橙色左框)/action-p2(灰色左框) |
| `.alert-box` | 提示框 | alert-bad(红色)/alert-warn(橙色)/alert-ok(绿色) |
| `.badge` | 状态标签 | badge-ok(绿色)/badge-warn(橙色)/badge-bad(红色) |
| `.dimension-grid` | 四维评分网格 | 4列grid + dimension(标题+分数+进度条) |
| `.kpi-grid` | KPI指标网格 | auto-fit grid + kpi-card |
| `.mini` | 小卡片 | mini-s(小标题) + mini-k(关键内容) + mini-n(条目) |
| `.concl` | 结论框 | 灰色背景#f1f5f9圆角卡片 |

**浅色白底主题**：
- body: background:#f8fafc, color:#1a1a2e
- card: background:#ffffff, border-radius:12px, padding:20px
- h1: font-size:28px, font-weight:700, color:#1a1a2e
- h2: font-size:20px, font-weight:600, padding-bottom:8px, border-bottom:2px solid var(--accent), display:inline-block
- h3: font-size:16px, font-weight:600, color:#1a1a2e
- table: th background:#f1f5f9, color:#3a3a5a (非深色背景)
- **只替换数据，不替换CSS**

**模板文件（最终版）**：`assets/fr-campaign-template-v4-final.html`（49,898 bytes，2026-07-23终版）。旧版 `assets/fr-campaign-template-v4-optimized.html` 已废弃。

**参考文件**：
- `references/html-structural-patterns.md` — HTML结构模式（标准6模块结构、体检报告式结构禁区、常见CSS问题修复）
- `references/30s-video-sellingpoint-capacity.md` — 30秒视频卖点承载力分析（FR实战数据）

## 单视频深度分析模式

当用户提供视频CDN链接+Brief（无Excel投放数据）时：
1. 执行Phase 4（ASR+Vision管线）
2. 执行Phase 5（5项检查评分）
3. 输出单视频HTML：内容大意+5项检查+风险提示+评级(仅内容分，无CTR不评级)+下轮优化

## 交付规则

- 文件名：`{市场}-{活动名}-campaign-review-v4.html`
- 位置：`/opt/data/outputs/`，上传后提供CDN直链
- 用户要求不需要CDN直链时，仅生成HTML到outputs/，跳过上传
- 逐场独立分析，不做跨活动横向对比

## 异常处理

| 场景 | 处理方式 |
|------|---------|
| Excel公式错误 | safe_float()跳过#开头值 |
| ASR不可用 | 标注"无口播"，纯视觉分析，不扣分 |
| 视频下载失败 | 标记video_source_unavailable |
| Brief无优先级 | 第3项只给1分，报告中注明 |
| 无加热CTR | 只出内容分，不评级 |
| 数据口径不可复现 | 投放分扣10分，标注"数据不可信" |

## 铁律

1. **不要凭常识判断好坏**：一律以tineco-kpi-baseline的阈值为准。
2. **不要合并投放分和内容分为一个数**：合完就丢了归因。
3. **费用加权**：内容分和及格线都必须费用加权，不得简单平均。
4. **逐placement判定**：汇总达标但单条placement落"差"档且占预算>10%的，必须单列。
5. **CPC/CPM是上限不是区间**：做到€2.0 CPM是"更好"，不是"脱靶"。
6. **Brief优先级缺失惩罚**：brief未标必做项/加分项时，第3项只能给1分，这是给甲方的信号。
7. **内容类型不评分**：记录但不调标准，攒样本后再定。
8. **P0/P1风险不扣分**：P0=模型识别不清，P1=创意空间。
9. **ASR降权=不给虚高ASR加分**：不是罚视频的分。基于视觉证据给分。
10. **两个分不合成一个数**："这场72分"没法用，"投放78/内容6.2(线7.1)"才能直接指向问题。
