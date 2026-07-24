# HTML结构模式参考

## 最终版5模块结构（2026-07-23生效，最高优先级）

### 全局结构

```
[总览评分卡]（无section，顶层h1开头）
  - verdict盒（综合评分+等级+复投线+加权分：投放40%+内容35%+预算25%）
  - alert-box alert-warn（📌 整体表现 — 一段话定性+三句话总结+量化测算）
  - kpi-grid（7列指标卡片：总花费/总曝光/总互动/CTR/CPM/CPE/CPC）
  - alert-box alert-bad（核心问题宏观）
  - alert-box alert-ok（亮点可复用资产）
  - 决胜武器表格（武器|为什么有效|落地动作|验证指标）

M1: <div class="section">
  - h2带h2score badge（📊 模块 1：KPI 判定 XX 分 · XX）
  - mini卡片（标准与达成）
  - card表格（法国统一KPI基准线）
  - card表格（逐条KPI判定）

M2: <div class="section">
  - h2带h2score badge（📋 模块 2：内容分析 XX 分 · XX）
  - mini warn（内容问题摘要）
  - card表格（达人逐条质检表：Brief/画面/钩子/节奏/总分/字幕/CTA/核心问题）
  - crx卡片×N（达人逐条视频深度分析）
    - crx-h: 达人名+产品+时长+内容分+预算+性价比+评级
    - crx-b: 总结段落+时间线表格(时间段|画面内容|字幕|卖点验证)

M3: <div class="section">
  - h2带h2score badge（📈 模块 3：投放效率诊断 XX 分 · XX）
  - mini bad（投放问题摘要）
  - mm矩阵（预算流向vs内容质量：达人|预算|占比|内容分|判定）
  - card表格（性价比横评判定规则）
  - crx卡片（达人内容效率横评）
  - card（曝光趋势）
  - card表格（诊断总结：效率指数+性价比指数+判定）
  - card表格（投放优化建议）

M4: <div class="section">
  - h2（🎯 模块 4：下一轮行动指导）
  - mini act（行动指导摘要）
  - action-item action-p0（P0紧急）
  - action-item action-p1（P1重要）
  - action-item action-p2（P2储备）
```

### 综合评分计算（加权公式）

```
综合评分 = 投放执行×40% + 内容质量×35% + 预算配置×25%
评级: A ≥88 | B ≥78(复投线) | C ≥60 | D <60
```

### 核心CSS选择器（81个，完整列表见final模板<head>）

| 类别 | 选择器 | 用途 |
|------|--------|------|
| 综合评分 | `.verdict` `.verdict-score` `.verdict-body` `.vk` `.vn` `.vg` `.vd` `.vh` `.vt` `.vbars` `.vb` `.vb-n` `.vb-t` `.vb-f` `.vb-w` `.vb-v` | 顶层评分盒 |
| KPI | `.kpi-grid` `.kpi-card` `.kpi-label` `.kpi-value` `.kpi-unit` | 指标网格 |
| 模块评分 | `.h2score` | H2内嵌评分badge |
| 提示框 | `.alert-box` `.alert-bad` `.alert-warn` `.alert-ok` | 问题/亮点提示 |
| 卡片 | `.card` `.mini` `.mini-s` `.mini-k` `.mini-n` `.mini-r` `.mini-x` `.mini-b` `.mini-t` | 各类内容卡片 |
| 达人卡 | `.crx` `.crx-h` `.crx-id` `.crx-m` `.crx-n` `.crx-k` `.crx-g` `.crx-b` `.crx-c` `.crx-bar` | 达人效率卡片 |
| 矩阵 | `.mm` `.mm-r` `.mm-n` `.mm-b` `.mm-sc` `.mm-v` `.mm-bar` `.mm-bud` | 预算质量矩阵 |
| 行动项 | `.action-item` `.action-p0` `.action-p1` `.action-p2` | 优先级行动卡片 |
| 通用 | `.badge` `.badge-ok` `.badge-warn` `.badge-bad` `.badge-good` `.badge-mid` | 状态标签 |
| 状态色 | `.s-ok` `.s-warn` `.s-bad` `.f-ok` `.f-warn` `.f-bad` | 文字/填充颜色 |
| 内容 | `.quality` `.problem` `.concl` `.subtitle` `.tiny` `.platform-ig` `.platform-fb` | 内容样式 |
| 进度条 | `.crx-bar` `.crx-bud` `.mm-bar` `.mm-bud` `.hi` `.lo` | 各类进度条 |

### 整体表现段（alert-box alert-warn）

- **位置**：总览评分卡内、verdict盒之后、KPI网格之前
- **用途**：一段话定性 + 三句话总结 + 量化测算
- **文字风格**：直指要害、犀利不留情面。禁用"投放兜住了内容"这类自我开脱式表述。

### ⚠️ 模板优先铁律（P0）

当用户提供参考HTML模板时：
1. **必须**将参考文件作为完整骨架（CSS+DOM结构100%原样保留）
2. **仅替换**数据节点内容（数字/文字/表格行）
3. **严禁**自行用旧CSS重新拼装（CSS组件间有关联依赖，缺少任一选择器都会导致渲染断裂）
4. **严禁**部分复用CSS或"复刻"结构
5. 正确做法：`curl下载` → `Python正则/BeautifulSoup定位数据节点` → `逐节点替换数据` → `保持<head>完整不变`

### 常见CSS问题

| 问题 | 原因 | 修复 |
|------|------|------|
| 达人卡片无进度条 | `.crx-bar`/`.crx-bud`选择器缺失 | 使用完整CSS模板 |
| verdict盒布局断裂 | `.vb`/`.vb-f`/`.vb-n`联动选择器缺失 | 使用完整CSS模板 |
| 矩阵表样式丢失 | `.mm`/`.mm-r`/`.mm-bar`选择器缺失 | 使用完整CSS模板 |
| 评分badge不显示 | `.h2score`选择器缺失 | 在H2内追加`<span class="h2score badge-xx">` |

## ❌ 已废弃结构

### 旧6模块结构（已作废）
```
M1 综合评分卡（独立section）❌ 已改为无section的顶层verdict
M2 KPI判定 ❌ 编号变为M1
M3 达人逐条质检表 ❌ 合并到M2的H3子块
M3.5 视频深度分析 ❌ 合并到M2的H3子块
M4 投放效率诊断 ❌ 编号变为M3
M5 行动指南 ❌ 编号变为M4
```

### 体检报告小结卡片（已否决）
```html
<!-- ❌ 禁止使用这种模块小结卡片 -->
<div class="card" style="background:#f0f4ff;border-left:4px solid #2c3e50;margin:15px 0;">
  <h3 style="color:#1a1a2e;margin:0 0 8px 0;">📋 模块X · 模块小结</h3>
</div>
```
