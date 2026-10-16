# DA-1000 · 双通道功放电平表

> 一个运行在浏览器里的虚拟功放面板：加载本地音频，实时解析双声道 / 全频 + 超低四路电平，驱动模拟 LED 表头、功率计、响度估算、系统功耗分析、市电三相监测、能量统计和峰值限幅器。

![status](https://img.shields.io/badge/status-stable-brightgreen)
![version](https://img.shields.io/badge/version-v0.13.1__bugfix-blue)
![license](https://img.shields.io/badge/license-MIT-lightgrey)
![deps](https://img.shields.io/badge/dependencies-none-success)

---

## 目录

- [项目目的](#项目目的)
- [项目简介](#项目简介)
- [功能特性](#功能特性)
- [快速开始](#快速开始)
- [界面说明](#界面说明)
- [技术实现](#技术实现)
  - [音频图拓扑](#音频图拓扑)
  - [电子分频](#电子分频)
  - [电平表与弹道](#电平表与弹道)
  - [刷新率控制与峰值累积](#刷新率控制与峰值累积)
  - [增益映射](#增益映射)
  - [功率限制器](#功率限制器)
  - [功率模拟](#功率模拟)
  - [表头灵敏度](#表头灵敏度)
  - [响度估算](#响度估算)
  - [功放类别与效率模型](#功放类别与效率模型)
  - [整体负载](#整体负载)
  - [市电三相模拟](#市电三相模拟)
  - [能量统计](#能量统计)
  - [主题系统](#主题系统)
  - [数值安全体系](#数值安全体系)
- [参数调优](#参数调优)
- [扬声器预设](#扬声器预设)
- [版本历史](#版本历史)
- [已知限制](#已知限制)
- [浏览器兼容性](#浏览器兼容性)
- [许可](#许可)

---

## 项目目的

做这个项目的起点很简单：**想要一个能"看见"自己音响系统工作状态的工具。**

市面上专业的音频分析软件（Smaart、REW、Arta 等）功能强大但价格不菲、上手门槛高；而大多数"音乐可视化"插件又停留在频谱柱状图的层面，跟真实功放面板上的 SIG / -12 / -6 / -3 / 0 / +3 / +6 / CLIP 指示灯完全不是一回事。

所以这个项目试图回答一个问题：

> **能不能只用浏览器原生 API，做出一个看起来、用起来都像真实功放的实时监看面板？**

答案是可以。而且不需要任何第三方库——一个 HTML 文件，双击打开就能用。

---

## 项目简介

DA-1000 是一个**单文件、零依赖**的浏览器音频工具。它的工作方式是：

1. 用户从本地加载一个音频文件；
2. Web Audio API 把音频拆成 **左 / 右 × 全频 / 超低** 四条独立通道；
3. 每条通道的信号经过滤波器、增益节点、限幅器后送往扬声器；
4. 同时，四路信号的实时电平被送往 UI，驱动 LED 表头、瓦特计、响度估算、市电监测和能量统计。

整个系统在功能上模拟了一台 **双通道电子分频功放（Bi-Amp）**：

```
        ┌──────────────────────────────┐
        │      全频通道（Full）         │ → 单15" 等全频扬声器
左声道 ─┤                              │
        │      超低通道（Sub）          │ → 双18" 等超低扬声器
        └──────────────────────────────┘
        （右声道同理）
```

它不是一个"假装在动"的动画，**所有读数、指示灯、功率值都直接来自真实音频采样**，而增益旋钮和限幅器会真实地改变你听到的声音。

---

## 功能特性

### 音频处理

- ✅ 双声道独立解析
- ✅ 全频 / 超低电子分频（Linkwitz-Riley 4 阶，分频点可调 20–1000 Hz）
- ✅ 四通道独立增益控制，**真实作用于输出音频**
- ✅ 峰值限幅器，可设定 0–120% 功率上限
- ✅ 过载保护提示（>100% 红色闪烁警告）

### 界面与交互

- ✅ 8 段 LED 电平表（SIG / -12 / -6 / -3 / 0 / +3 / +6 / CLIP）
- ✅ 模拟弹道：瞬起 + 指数缓落，接近真实 VU / PPM 表
- ✅ 物理旋钮：拖拽、滚轮、双击复位、**键盘直接输入 dB 值**
- ✅ 表头灵敏度调节（-12dB ~ +12dB 显示增益）
- ✅ 数据刷新率选择器（非常低 / 低 / 中 / 高 / 实时）
- ✅ 实时功率显示（瓦特 + 百分比 + 进度条）
- ✅ 扬声器组合下拉选择（含线阵列）
- ✅ 自定义扬声器最大功率输入
- ✅ 响度估算栏（1m / 观众席第一排 / 末排的 SPL）
- ✅ **浅色 / 深色主题切换**，偏好自动保存
- ✅ **图标化系统标签**，内联 SVG，随主题配色
- ✅ 响应式布局，适配桌面与移动端

### 系统分析

- ✅ 系统功耗面板：总输出功率 / 市电消耗 / 实时效率
- ✅ 功放类别选择（AB / H / D），带真实电学效率模型
- ✅ **整体负载**：四通道综合压力指标，>100% 红色闪烁警告
- ✅ 市电三相监测：L1 / L2 / L3 电压 + 电流
- ✅ **能量统计**：累计电能（kWh）+ 5 个时间窗口的平均功率 + 运行时长 + 一键重置
- ✅ 分频点二级菜单（预设 + 自定义）

### 工程特性

- ✅ 单 HTML 文件，零依赖，无需构建
- ✅ 无网络请求，完全离线运行
- ✅ **数值安全体系**：所有时间/增益/功率计算都经过安全辅助函数，NaN 无法传播
- ✅ **跨平台稳定性**：正确处理 rAF 时间戳回退、标签页挂起、时钟漂移
- ✅ MIT 许可，可自由修改和分发

---

## 快速开始

1. 下载 `index.html`（或把本文档下方的完整代码保存为 `.html` 文件）
2. 用**现代浏览器**（Chrome / Edge / Firefox / Safari）双击打开
3. 点击左下角 **「加载音频」**，选择一个本地音频文件（MP3 / WAV / FLAC / OGG / M4A 等浏览器支持的格式）
4. 文件加载后会自动开始播放，也可以用 **「播放 / 暂停」** 按钮或 **空格键** 控制

> ⚠️ **第一次播放需要用户手势**
> 浏览器的自动播放策略要求 AudioContext 必须在用户交互（点击、按键）之后才能启动。因此打开页面后必须先点击一次「加载音频」或「播放」按钮，音频才会真正开始。这是浏览器安全策略，不是 bug。

---

## 界面说明

```
┌──────────────────────────────────────────────────────────────────┐
│ ☀  ● PWR   DA-1000   Stereo Bi-Amp · Active Crossover           │  ← 顶栏
├──────────────────────────────────────────────────────────────────┤
│  LEVEL    SIG  -12  -6  -3   0   +3   +6  CLIP            dB     │
│  FULL L  ●    ●    ●   ●    ●    ○    ○    ○              +1.2   │  ← 表头
│  FULL R  ●    ●    ●   ●    ○    ○    ○    ○              -0.4   │
│  ─────────────────────────────────────────────────────────────   │
│  SUB  L  ●    ●    ●   ●    ●    ●    ○    ○              +4.8   │
│  SUB  R  ●    ●    ●   ○    ○    ○    ○    ○              -3.1   │
├──────────────────────────────────────────────────────────────────┤
│   (全频L)   (全频R)   (超低L)   (超低R)   │  (灵敏度)             │
│      ◉         ◉         ◉         ◉      │     ◉                │  ← 旋钮
│    +0.0dB   +0.0dB   +0.0dB   +0.0dB      │   +0.0dB             │
├──────────────────────────────────────────────────────────────────┤
│  ┌─全频L──┐ ┌─全频R──┐ ┌─超低L──┐ ┌─超低R──┐                    │
│  │ 320 W  │ │ 280 W  │ │ 840 W  │ │ 760 W  │                    │
│  │ ████░░ │ │ ███░░░ │ │ █████░ │ │ ████░░ │                    │  ← 功率
│  │ [单15"]│ │ [单15"]│ │ [双18"]│ │ [双18"]│                    │     面板
│  │  64.0% │ │  56.0% │ │  35.0% │ │  31.7% │                    │
│  │  限100%│ │  限100%│ │  限100%│ │  限100%│                    │
│  │ 1m 前排 末排│ 1m 前排 末排│ 1m 前排 末排│ 1m 前排 末排│          │
│  │114 98 82│ │113 97 81│ │122 108 92│ │121 107 91│              │
│  └────────┘ └────────┘ └────────┘ └────────┘                    │
├──────────────────────────────────────────────────────────────────┤
│ [⑂ 分频点▾120Hz] [🔊 总输出 2.2kW] [⚡ 市电 2.4kW] [%% 效率 91.7%] │  ← 系统
│ [🎛 整体负载 78.4%] [▶ 功放类别▾ D类]                             │     面板
├──────────────────────────────────────────────────────────────────┤
│  🔌 市电 · 三相              L1  237.2V  3.06A                    │  ← 市电
│  MAINS · 3-PHASE AC         L2  235.8V  2.98A                    │     面板
│                             L3  238.4V  3.14A                    │
├──────────────────────────────────────────────────────────────────┤
│  ┌── ⚡ 累计电能 · Energy ──┐    📊 平均功率  AVG · MAINS SIDE     │
│  │        0.842            │    ┌──────────────────────┐           │  ← 能量
│  │        kWh              │    │  [自启动 ▾]          │           │     面板
│  │  [↻ 重置]  ● 运行 02:34:11│    │      2.40 kW        │           │
│  └─────────────────────────┘    └──────────────────────┘           │
├──────────────────────────────────────────────────────────────────┤
│ [加载音频] [播放] 1:23/4:56 [↻ 刷新率▾ 非常低]        track.mp3   │  ← 底栏
└──────────────────────────────────────────────────────────────────┘
```

### 表头区（LEVEL）

- **SIG**：信号存在指示灯，超过 -40dB 表头刻度即点亮
- **-12 / -6 / -3 / 0 / +3 / +6**：标准 PPM 刻度
- **CLIP**：削顶指示灯，峰值 ≥ 0.985 时触发，保持 1.2 秒
- **右侧 dB 读数**：带弹道的实时表头值，超过 0dB 变黄，削顶变红

### 旋钮区

- **全频 L/R、超低 L/R**：四条通道的独立增益，范围 **-40dB ~ +10dB**，中点 0.5 对应 0dB 单位增益
- **灵敏度（METER SENS）**：只影响表头显示，不影响实际声音，范围 **+12dB ~ -12dB**

**操作方式：**

| 操作 | 效果 |
|---|---|
| 拖拽指针 | 连续旋转 |
| 滚轮 | 微调（每格 2.5%） |
| 双击 | 复位到 0dB |
| 点击下方输入框 | 键盘直接输入 dB 值（如 `-3.5`、`+5`、`0`） |
| 输入框内回车 | 确认 |
| 输入框内 Esc | 取消 |

### 功率卡片

每张卡片从上到下依次显示：

- **大号数字**：实时输出功率（瓦特）
- **进度条**：相对该扬声器额定功率的占比
- **下拉菜单**：选择扬声器组合（"自定义 …" 时可手动输入最大功率）
- **百分比**：当前出力百分比，超过 100% 会变成**红色并闪烁**
- **「限 XX%」输入框**：功率上限设定，0–120%，超过 100% 会红边警示
- **响度栏**：三个格子显示 1m / 前排 / 末排的估算 SPL（选择"自定义"时隐藏）

### 系统面板

六个格子，每个带小图标：

- **分频点**（分叉箭头）：二级菜单，可选 50 / 60 / 80 / 100 / 120 / 150 / 200 / 250 Hz 或自定义（20–1000 Hz）
- **总输出功率**（喇叭）：四通道实际输出之和，超过 1000W 自动切换 kW
- **市电消耗**（闪电）：从墙壁插座实际抽取的功率
- **效率**（百分号）：实时效率百分比 + 可视化进度条
- **整体负载**（仪表盘）：四通道综合压力指标 + 进度条，>100% 红色闪烁警告
- **功放类别**（放大器）：AB / H / D 三选一，影响效率模型

### 市电面板

- 三格分别显示 **L1 / L2 / L3** 的电压（V）和电流（A）
- 电压在 220–255 V 之间连续随机游走
- 电流由总消耗功率反算，三相带轻微不平衡
- 电压预警：偏低（< 224V）黄字，偏高（> 250V）橙字

### 能量统计面板

分为左右两栏：

**左侧 — 累计电能卡片（绿色高亮）**

- 大号数字：自启动或上次重置以来，市电侧累计消耗的电能（kWh）
- 下方「重置」按钮：一键清零电能和所有平均功率统计
- 运行时读数：`HH:MM:SS` 格式，从启动或重置起算

**右侧 — 平均功率**

- 一个下拉菜单，可选：**自启动 / 最近 10 分钟 / 最近 5 分钟 / 最近 30 秒 / 最近 5 秒**
- 下方大号数字显示对应窗口内的市电侧平均功率，自动根据量级切换 W / kW 单位

### 底栏

- **加载音频 / 播放 / 暂停** 按钮（空格键也可控制播放）
- **播放进度**：`当前 / 总时长`
- **刷新率下拉**：非常低 / 低 / 中 / 高 / 实时
- **文件名**：当前加载的音频文件名

### 主题切换

顶栏最左侧是一个圆形按钮，点击在**浅色 / 深色模式**之间切换。偏好保存到 `localStorage`，下次打开时自动应用。

---

## 技术实现

### 音频图拓扑

整个系统基于 **Web Audio API** 构建。以下是 v0.13.1 的完整信号流：

```
                       ┌─────────────────────────────────────────────┐
                       │            左声道 (Channel 0)                │
                       │                                             │
                       │  ┌── HP×2 (LR4) ──→ Analyser ──→ Gain ──┐  │
<audio> ──→ MediaElementSource ──→ ChannelSplitter(2)             │  │
                       │  │                                       ↓  │
                       │  │                              ChannelMerger(2) ──→ Destination
                       │  │                                       ↑  │
                       │  └── LP×2 (LR4) ──→ Analyser ──→ Gain ──┘  │
                       │                                             │
                       └─────────────────────────────────────────────┘
                       （右声道结构完全相同）
```

**关键设计决策：**

1. **`MediaElementSource` 只能对同一个 `<audio>` 元素创建一次。** 代码中用 `ensureAudio()` 做惰性初始化并加守卫，避免重复创建报错。

2. **分析器放在增益节点之前。** 这一点非常重要：
   - 如果分析器放在增益之后，限幅器读取到自己衰减过的信号，会形成负反馈闭环，导致振荡或抖动；
   - 放在增益之前，分析器看到的是"需求信号"，实际输出电平由 `增益前峰值 × 总增益` 计算得出；
   - 好处是限制器的攻击/释放过程不会污染测量结果，读数平滑稳定。

3. **`ChannelMerger` 用于双声道回合并。** 全频和超低的增益输出分别送往同一个 `ChannelMerger` 的不同输入通道，浏览器会把它们自动求和（相当于真实系统的电子分频后功放求和）。

4. **不使用 `crossOrigin`。** 本地文件通过 `blob:` URL 加载，本身就是同源，加 CORS 头反而多余。

### 电子分频

分频点默认 120 Hz，可在 20–1000 Hz 之间自由调节。采用 **Linkwitz-Riley 4 阶** 结构：

```
高通：BiquadFilter(highpass, Q=0.707) × 2 级串联
低通：BiquadFilter(lowpass,  Q=0.707) × 2 级串联
```

**为什么是 LR4？**

- **Butterworth（Q=0.707）单级** 是 2 阶，分频点处 -3dB，两个滤波器叠加会在交叉点产生约 -6dB 的凹陷；
- **Linkwitz-Riley** 是两个 Butterworth 级联，得到 4 阶响应，分频点处 -6dB；
- 高通和低通在分频点处**相位一致**（同相），叠加后幅频响应平坦，不产生凹陷；
- 这是专业音响系统中最常用的分频结构。

**分频点实时调整：**

在播放中改变分频点时，所有 8 个滤波器使用 `setTargetAtTime` 做 30ms 平滑过渡，不会产生爆音或咔哒声：

```js
for (const filt of crossoverFilters) {
  for (const f of filt.hp) f.frequency.setTargetAtTime(freq, t, 0.03);
  for (const f of filt.lp) f.frequency.setTargetAtTime(freq, t, 0.03);
}
```

### 电平表与弹道

电平表读取 `AnalyserNode.getFloatTimeDomainData()` 得到的时域数据，取峰值：

```js
function readPeak(analyser, buf) {
  analyser.getFloatTimeDomainData(buf);
  let p = 0;
  for (let i = 0; i < buf.length; i++) {
    const v = buf[i] < 0 ? -buf[i] : buf[i];
    if (v > p) p = v;
  }
  return p;
}
```

**弹道（Ballistics）** 模拟真实 PPM 表的行为：

```js
if (meterDb >= row.db) {
  row.db = meterDb;                                    // Attack：瞬时跟上
} else {
  const k = safeK(displayDt, RELEASE_MS);              // Release：指数回落
  if (k > 0) row.db += (meterDb - row.db) * k;
}
```

- **Attack**：瞬间响应，不错过任何瞬态峰值
- **Release**：一阶指数衰减，时间常数 `RELEASE_MS = 220ms`
- 使用 `Math.exp` 而非简单线性插值，保证回落速度与刷新率无关

### 刷新率控制与峰值累积

为了让表头在不同使用场景下都能舒适观察，v0.7 引入了刷新率选择器，v0.12 增加"非常低"档：

| 模式 | 刷新间隔 | 刷新率 | 适用场景 |
|---|---|---|---|
| 非常低 | 250ms | ~4 fps | 长时间监看，接近机械表观感 |
| 低 | 100ms | ~10 fps | 长时间监看，只看稳定趋势 |
| 中 | 50ms | ~20 fps | 日常监看 |
| 高 | 25ms | ~40 fps | 精细调整，快速动态 |
| 实时 | 0ms | ~60 fps | 逐帧刷新（默认） |

**关键设计：采样与显示解耦。**

如果只是"隔 X 毫秒读一次分析器"，就会漏掉两次读取之间的瞬态峰值。解决方案是分两层：

**第一层——实时（每一帧，约 60fps）**
- 读取 `AnalyserNode` 时域数据，算出瞬时峰值
- 更新限幅器（必须实时响应，否则音频会爆）
- 削顶检测（削顶是瞬态事件，不能漏）
- 把当前的输出峰值**累积**到 `row.peakAccum`（取最大值）

**第二层——按刷新率**
- 到达刷新时间点时，取累积峰值作为这一区间的代表值
- 用它更新表头弹道、LED、dB 读数、功率
- 清空累积器，等下一个区间
- 弹道使用的时间常数是 `displayDt`（两次刷新间隔），所以衰减速度与刷新率无关

结果是：**刷新率越低，显示越安静，但每一个峰值都不会被漏掉。** 低刷新率下，表头会呈现一种"定格—跳跃—定格"的节奏，反而更像真实的机械 PPM 表。

### 增益映射

旋钮位置 `v ∈ [0, 1]` 到 dB 的映射是分段线性的：

```
v = 0.0  →  -40 dB   （基本静音）
v = 0.5  →    0 dB   （单位增益，中点）
v = 1.0  →  +10 dB   （最大增益）
```

实现：

```js
function knobToDb(v) {
  if (v <= 0.5) return -40 + (v / 0.5) * 40;
  else          return ((v - 0.5) / 0.5) * 10;
}
```

**为什么用分段线性而不是等 dB 刻度？**

- 真实功放的增益旋钮刻度通常是分段的：下半段跨度大（衰减为主），上半段跨度小（增益为主）
- 单位增益点落在物理中点，手感上"上下对称"
- 用户从 0dB 往左拧是衰减，往右拧是增益，符合直觉

增益变化通过 `setTargetAtTime` 施加到 `GainNode`：

```js
node.gain.setTargetAtTime(targetGain, ctx.currentTime, 0.02);
```

20ms 的时间常数避免了快速旋转旋钮时产生的"拉链噪声"（zipper noise）。

### 功率限制器

这是 v0.6 引入的核心功能。目标是：**允许功放过载到 120%，但保护扬声器不超过设定功率。**

**工作原理：**

1. 计算"需求功率"——假设没有限幅，当前信号经旋钮增益后会有多大功率：

   ```js
   const demandPeak  = rawPeak * knobGain;
   const demandPower = maxW * demandPeak * demandPeak;
   ```

2. 与"限制功率"比较：

   ```js
   const limitPower = maxW * (limitPct / 100);
   ```

3. 若需求超过限制，计算所需的总增益衰减：

   ```js
   if (demandPower > limitPower) {
     const neededOutPeak   = Math.sqrt(limitPower / maxW);
     const neededTotalGain = neededOutPeak / rawPeak;
     targetLim = neededTotalGain / knobGain;   // 0..1 的衰减系数
   }
   ```

4. **非对称时间常数** 平滑衰减系数：

   ```js
   const attacking = targetLim < limiterGain[band][ch];
   const tauMs = attacking ? 5 : 250;   // 攻击 5ms，释放 250ms
   const limK = safeK(dt, tauMs);
   ```

5. 最终施加到增益节点的总增益：

   ```js
   const totalGain = knobGain * limiterGain[band][ch];
   ```

**为什么攻击和释放时间不同？**

- **快攻击（5ms）**：瞬态峰值一出现就立刻压住，防止削顶
- **慢释放（250ms）**：释放太慢会让声音发闷、失去动态；太快会"抽气"（pumping）。250ms 是一个音乐性较好的折中

**过载允许到 120%：**

功率百分比上限为 `MAX_LIMIT_PCT = 120`，即允许 `+1.6dB` 的过载。这是因为专业功放通常有 1.5~2 倍的峰值储备（headroom），可以短时间过载而不失真。但过载对扬声器（尤其是高音单元）是危险的，所以 UI 上用**红色闪烁**强提示。

### 功率模拟

功率换算基于 **0 dBFS = 额定满功率** 的假设：

```js
const powerDb = Math.min(dbfs, 0);              // 上限钳到 0dB
const power   = maxW * Math.pow(10, powerDb / 10);
```

对于过载状态，上限放宽到 1.44（即 120%）：

```js
const pFactor  = Math.min(rawOutPeak * rawOutPeak, 1.44);
const powerRaw = maxW * pFactor;
```

功率读数经过 **160ms 指数平滑**，避免数字疯狂跳动：

```js
const pk = safeK(displayDt, 160);
if (pk > 0) {
  row.powerSmooth += (powerRaw - row.powerSmooth) * pk;
  if (!isFinite(row.powerSmooth) || row.powerSmooth < 0) row.powerSmooth = 0;
}
```

**瓦特数显示精度** 根据量级自适应：

| 范围 | 显示格式 |
|---|---|
| 0 W | `0.0` |
| < 10 W | 两位小数（如 `3.42`） |
| < 100 W | 一位小数（如 `47.5`） |
| ≥ 100 W | 整数（如 `320`） |

### 表头灵敏度

灵敏度旋钮调节的是表头显示的"参考点"：

```js
sensOffset = 12 - 24 * sensValue;   // +12dB ~ -12dB
```

最终的表头读数：

```js
const ref     = BASE_REF[band] + sensOffset;
const meterDb = dbfs - ref;
```

其中 `BASE_REF = { full: -6, sub: -12 }` 是全频和超低的**基础参考点**：

- 全频以 -6 dBFS 为表头 0dB 参考
- 超低以 -12 dBFS 为表头 0dB 参考（超低信号通常录制电平更高，参考点下移避免表头常年打满）

调节灵敏度会**同时影响四条通道**的显示，但**不影响实际声音**。这是设计上的刻意区分：

- **增益旋钮** → 改变声音
- **灵敏度旋钮** → 只改变显示

### 响度估算

每个通道的功率卡片底部有一行**响度估算栏**，显示三个位置的估算 SPL：

- **1m**：标准灵敏度参考点
- **前排**：观众席第一排的距离
- **末排**：观众席最后一排的距离

计算基于 **反平方定律**：

```
SPL(d) = sensitivity + 10·log10(P) - 20·log10(d)
```

其中：

- `sensitivity` 是扬声器的 1W/1m 灵敏度（dB SPL）
- `P` 是当前实际输出功率（已包含增益、限幅）
- `d` 是距离（米）

**距离是按扬声器类型合理推测的：**

| 扬声器 | 灵敏度 | 第一排 | 末排 |
|---|---|---|---|
| 单 8" | 94 dB | 4 m | 20 m |
| 单 12" | 98 dB | 10 m | 40 m |
| 单 15" | 99 dB | 15 m | 50 m |
| 双 15" | 102 dB | 22 m | 80 m |
| 线阵列 4 组 | 106 dB | 20 m | 80 m |
| 线阵列 8 组 | 109 dB | 25 m | 120 m |
| 单 18" 超低 | 98 dB | 12 m | 45 m |
| 双 18" 超低 | 101 dB | 18 m | 65 m |

**这就是为什么单 8" 放在 4 米、而 8 寸线阵列可以放在 25 米**——线阵列的投射距离和阵列耦合增益是两个不同量级的东西。

**颜色预警：**

- < 105 dB：正常（钢蓝）
- 105–112 dB：偏响（黄色）
- ≥ 112 dB：危险（红色）

阈值的依据是职业健康标准——**105 dB 是长时间暴露的界限，112 dB 已进入需要佩戴听力保护的区间**。

**自定义扬声器时隐藏响度栏**：因为不知道灵敏度和距离，整个响度栏直接 `display:none`。

### 功放类别与效率模型

系统功耗面板中的"效率"不是简单乘个系数，而是按真实功放的**电学特性**建模。每个类别用三个参数描述：

| 类别 | 增量效率 | 空闲基础功耗 | 每瓦额定功率的空闲功耗 |
|---|---|---|---|
| AB | 60% | 20 W | 0.08 W/W |
| H | 80% | 15 W | 0.04 W/W |
| D | 92% | 5 W | 0.01 W/W |

**功耗模型：**

```
空闲功耗 = idleBase + idlePerW × 该通道最大功率
实际消耗 = 空闲功耗 + 输出功率 / 增量效率
实时效率 = 输出功率 / 实际消耗
```

这个模型的物理含义是：**输出级存在一个固定的能量转换效率上限**，同时整机还有一个与功率规模相关的"待机开销"。把这两个叠加，就能得到一条符合真实功放的电学曲线。

用 400W 全频通道举例，你能直接感受到三种类别的差异：

| 输出 | AB 类消耗 | AB 效率 | D 类消耗 | D 效率 |
|---|---|---|---|---|
| 0 W（待机） | 52 W | 0% | 9 W | 0% |
| 10 W | 68.7 W | 14.6% | 19.9 W | 50.3% |
| 100 W | 218.7 W | 45.7% | 117.7 W | 84.9% |
| 400 W | 718.7 W | 55.7% | 443.8 W | 90.1% |

几个符合直觉的现象：

- **无音频播放时效率 = 0%** —— 因为输出功率为零，除以任何消耗都是零
- **待机时消耗不为零** —— 静态功耗在真实消耗里占比很大，尤其 AB 类
- **扬声器组合变化会影响待机功耗** —— 换到线阵列 8 组（4000W）时，AB 类每个通道的静态功耗自动从 52W 涨到 340W
- **小信号时效率极低** —— 这是真实功放最容易被忽视的特性

### 整体负载

位于效率右侧的"整体负载"是一个**复合压力指标**，代表系统当前最紧张的状态。它不受刷新率选择器影响，固定以 **100ms 间隔**更新（对应刷新率"低"档）。

**计算逻辑：**

```js
const loads = rows.map(r => safeNum((r.powerSmooth / r.preset.maxW) * 100, 0));
const maxLoad = loads.length ? Math.max(...loads) : 0;
const avgLoad = loads.length ? loads.reduce((a, b) => a + b, 0) / loads.length : 0;
const rawLoad = safeNum(maxLoad * 0.55 + avgLoad * 0.45, 0);
```

**为什么用加权混合？**

- **纯最大值**：会高估压力。如果只有一路满载，其它三路全静音，整体负载不该是 100%
- **纯平均值**：会低估压力。如果一路 120% 过载、其它三路 30%，平均值只有 52.5%，看不出严重性
- **55% 最大值 + 45% 平均值**：既突出"最紧张的那一路"，又反映"整体是否同时在发力"

举例说明：

| 场景 | 最大负载 | 平均负载 | 整体负载 |
|---|---|---|---|
| 四通道都在 80% | 80% | 80% | 80% |
| 单通道 100%，其它 0% | 100% | 25% | 66.3% |
| 单通道 120%，其它 30% | 120% | 52.5% | 89.6% |
| 四通道都在 100% | 100% | 100% | 100% |
| 四通道都在 110% | 110% | 110% | 110% |

**颜色分级：**

| 负载 | 颜色 | 语义 |
|---|---|---|
| < 70% | 钢绿 | 正常 |
| 70–85% | 黄色 | 有一定压力 |
| 85–100% | 橙色 | 接近上限 |
| > 100% | 红色闪烁 | 系统过载 |

**为什么固定 100ms 刷新？**

- **整体负载变化慢**：它是四个通道的综合反映，本来就不需要高频刷新
- **读数稳定**：高速刷新会让这个复合指标跳动得难以观察
- **警告仍及时**：100ms 的延迟人眼几乎察觉不到，过载警告同样"立刻"出现

**为什么用 180ms 时间常数平滑？**

与各通道功率的 160ms 接近，但略慢一点，因为它本身就是复合指标，再快也没意义。

### 市电三相模拟

市电面板位于系统面板下方，显示 L1 / L2 / L3 三相的电压和电流。

**电压模型**：连续随机游走 + 弹性回中：

```js
p.volt += (Math.random() - 0.5) * 0.22 * stepScale;        // 随机抖动
p.volt += (MAINS_NOMINAL - p.volt) * 0.0018 * stepScale;    // 缓慢回中
p.volt = safeVolt(p.volt);                                  // 每次写入后校验
// 硬边界：220 - 255 V
```

几个参数的含义：

- **`MAINS_NOMINAL = 237`** —— 长期均值（中国 220V 电网实际运行中位电压）
- **随机步长 0.22** —— 每帧最多变化约 ±0.11V，看起来是连续的小幅摆动而不是跳变
- **回中系数 0.0018** —— 弱回拉，避免长期贴在 220 或 255 的边界
- **`stepScale = sqrt(dt/16.7)`** —— 让波动速度与帧率无关
- **`safeVolt`** 每次写入后校验：任何非有限值或明显越界值立即恢复标称值

每相独立游走，所以三相电压各不相同（真实情况也是这样）。

**电流模型**：从已有的**市电消耗功率**反算，三相大致均匀但带轻微不平衡：

```js
const perPhaseW = totalConW / 3;
targetAmp = (perPhaseW * loadBias) / (volt * MAINS_PF);
```

- **`loadBias`** 分别为 1.00 / 0.97 / 1.03，模拟三相负载的天然不平衡
- **`MAINS_PF = 0.92`** 是假设的功率因数，带 PFC 的现代功放通常在 0.9 以上
- 电流用 150ms 时间常数平滑，读数不会乱跳
- 目标值无效时**跳过本次更新**，保留旧值，绝不写入 NaN

**电压颜色预警：**

- 224V 以下（偏低）：**黄色**
- 224–250V（正常）：绿色
- 250V 以上（偏高）：**橙色**

**完整因果链**：拖动全频增益旋钮时可以看到

1. 增益增加 → 输出功率上升
2. → 市电消耗上升（按功放类别的效率换算）
3. → 三相电流同步上升
4. → 效率读数变化（因为大功率时增量效率占比更高）
5. → 如果过载，限幅器介入，功率读数被压住，电流也跟着稳定

### 能量统计

能量面板（v0.12 引入）提供两个统计维度：**累计电能**和**多窗口平均功率**。

**累计电能：**

对市电侧消耗功率随时间积分：

```js
const dtSec = safeDt(dtMs, 1000) / 1000;
const p = safeNum(totalConSmooth, 0);
totalEnergyKWh += p * dtSec / 3600000;
if (!isFinite(totalEnergyKWh) || totalEnergyKWh < 0) totalEnergyKWh = 0;
```

`dtMs` 是两次刷新之间的真实毫秒差，所以即使用户切换刷新率，积分仍然准确。数值按量级选择显示精度：

| 范围 | 显示 |
|---|---|
| < 0.001 | `0.000` |
| < 1 | 三位小数（如 `0.842`） |
| < 100 | 两位小数（如 `42.18`） |
| ≥ 100 | 整数（如 `253`） |

**多窗口平均功率：**

五个时间窗口共用同一个采样缓冲区：

| 窗口 | 样本数 | 采样间隔 | 实际时长 |
|---|---|---|---|
| 自启动 | 无限（累加） | 250ms | 从启动起 |
| 10 分钟 | 2400 | 250ms | 600 秒 |
| 5 分钟 | 1200 | 250ms | 300 秒 |
| 30 秒 | 120 | 250ms | 30 秒 |
| 5 秒 | 20 | 250ms | 5 秒 |

采样缓冲区是一个 **2400 长度的 `Float32Array` 环形缓冲区**：

```js
function pushSample(v) {
  v = safeNum(v, 0);
  if (v < 0) v = 0;
  powerHistory[historyHead] = v;
  historyHead = (historyHead + 1) % ENERGY_HISTORY;
  if (historyCount < ENERGY_HISTORY) historyCount++;
}

function avgRecent(n) {
  const cnt = Math.min(n, historyCount);
  if (cnt === 0) return null;
  let sum = 0;
  for (let i = 0; i < cnt; i++) {
    const idx = (historyHead - 1 - i + ENERGY_HISTORY * 2) % ENERGY_HISTORY;
    const v = powerHistory[idx];
    if (isFinite(v)) sum += v;
  }
  const avg = sum / cnt;
  return isFinite(avg) ? avg : null;
}
```

"自启动"窗口不用缓冲区，用独立的 `totalSum / totalCount` 累计，所以无论运行多久都不会因为缓冲区回绕而失真。

**采样节奏：**

- **采样间隔固定 250ms**，与刷新率选择器无关
- **UI 更新间隔 400ms**，独立于刷新率选择器，也独立于整体负载的 100ms

这样即使把刷新率调到"实时"（60fps），采样也保持稳定的 4 Hz，窗口平均的物理意义不会改变。

**采样循环的防御性设计：**

```js
let guard = 0;
while (now - historyLastSample >= ENERGY_SAMPLE_MS && guard < 100) {
  historyLastSample += ENERGY_SAMPLE_MS;
  pushSample(totalConSmooth);
  totalSum += safeNum(totalConSmooth, 0);
  totalCount++;
  guard++;
}
if (guard >= 100) historyLastSample = now;  // 积压过多，直接对齐当前时间
```

标签页被挂起很久后，`historyLastSample` 会远远落后于 `now`。如果不加上限，这个循环会尝试补齐所有缺失的采样点，可能阻塞主线程。加上限后，超过 100 次就直接对齐，牺牲一点时间精度换取流畅度。

**重置：**

点击重置按钮清空所有状态：

```js
totalEnergyKWh = 0;
totalSum = 0;
totalCount = 0;
historyHead = 0;
historyCount = 0;
const now = performance.now();
historyLastSample = isFinite(now) ? now : Date.now();
energyStartTime = historyLastSample;
lastEnergyDisplay = 0;
```

UI 立刻回到全零状态，按钮本身闪一下蓝色光晕（500ms `resetFlash` 动画）作为反馈。

### 主题系统

v0.12 引入了浅色 / 深色主题切换。所有颜色都定义为 **CSS 自定义属性**，集中在两个块里：

```css
:root{
  --page-1:#1b1f24;
  --tx-1:#c9ced6;
  /* ... 深色模式的所有变量 */
}

body.light{
  --page-1:#eef1f5;
  --tx-1:#2a3038;
  /* ... 浅色模式覆盖 */
}
```

切换时只需在 `<body>` 上添加 / 移除 `light` 类，所有颜色自然过渡（用 `transition: background .35s ease` 等实现）。

**切换逻辑：**

```js
themeBtn.addEventListener('click', () => {
  const isLight = document.body.classList.toggle('light');
  try{
    localStorage.setItem('da1000-theme', isLight ? 'light' : 'dark');
  }catch(e){}
});
```

页面加载时从 `localStorage` 读取偏好：

```js
try{
  const saved = localStorage.getItem('da1000-theme');
  if (saved === 'light' || saved === 'dark') applyTheme(saved);
}catch(e){}
```

用 `try/catch` 包裹是因为在某些隐私模式或沙盒环境下 `localStorage` 会抛异常。

**图标系统：**

所有面板标签都带有内联 SVG 图标，`stroke: currentColor`，自动跟随主题颜色。图标集合包括：

- 分叉箭头（分频点）
- 喇叭 + 声波（总输出功率）
- 闪电（市电消耗、累计电能）
- 百分号（效率）
- 仪表盘（整体负载）
- 放大器三角（功放类别）
- 电源插头（市电三相）
- 柱状图（平均功率）
- 循环箭头（刷新率）
- 太阳 / 月亮（主题切换）

图标尺寸统一 13×13px，线宽 1.3px，视觉上与 8.5px 的标签文字协调。

### 数值安全体系

v0.13 起，项目引入了**完整的数值安全体系**，彻底解决跨平台的 NaN 污染问题。这不是简单的"加几个 isFinite 检查"，而是一套**辅助函数 + 分层守卫**的架构。

#### 问题背景

`requestAnimationFrame` 的时间戳在**某些平台和某些时刻会轻微回退**（帧率抖动、切标签页、系统时钟微调）。这在 Windows 上更常见，macOS 上时钟源更平滑。

如果只做简单的钳制，会踩一个致命的 JavaScript 陷阱：

```js
Math.max(0, NaN)   // ← 返回 NaN，不是 0！
Math.min(NaN, 100) // ← 返回 NaN
```

一旦 `now - lastT` 变成 `NaN`，之前的"钳到非负"代码完全失效，`dt` 就是 `NaN`，然后一路污染下去：

```
dt = NaN
  → smoothKnobGain = NaN
  → knobGain = NaN
  → powerSmooth = NaN
  → totalOutSmooth / totalConSmooth = NaN
  → totalEnergyKWh = NaN
  → 电压模拟 = NaN
```

更糟的是：一旦 `lastT` 被污染成 `NaN`，之后每一帧 `now - NaN = NaN`，**永久卡死**，只能刷新页面。

#### 解决方案

**第一层：5 个数值安全辅助函数**

```js
/* 时间差：非有限数或负数返回 0，超大值钳到 max */
function safeDt(v, max) {
  if (typeof v !== 'number' || !isFinite(v) || v < 0) return 0;
  return v > max ? max : v;
}

/* 指数平滑系数：任何非法输入返回 0（=跳过本次更新） */
function safeK(dt, tau) {
  if (typeof dt !== 'number' || !isFinite(dt) || dt <= 0) return 0;
  if (typeof tau !== 'number' || !isFinite(tau) || tau <= 0) return 0;
  const k = 1 - Math.exp(-dt / tau);
  return isFinite(k) ? (k < 0 ? 0 : (k > 1 ? 1 : k)) : 0;
}

/* 通用数值守卫：非有限数返回 fallback */
function safeNum(v, fallback) {
  return (typeof v === 'number' && isFinite(v)) ? v : fallback;
}

/* 电压守卫：异常值拉回标称值 */
function safeVolt(v) {
  if (typeof v !== 'number' || !isFinite(v)) return MAINS_NOMINAL;
  if (v < MAINS_MIN - 20 || v > MAINS_MAX + 20) return MAINS_NOMINAL;
  return v;
}

/* 时间守卫：非有限时间戳返回 fallback */
function safeTime(v, fallback) {
  return (typeof v === 'number' && isFinite(v)) ? v : fallback;
}
```

**第二层：`frame()` 的三道防线**

```js
function frame(now) {
  requestAnimationFrame(frame);

  /* 防线 1：now 本身不合法 → 跳过整帧，不污染任何状态 */
  if (typeof now !== 'number' || !isFinite(now)) return;

  /* 防线 2：lastT 未初始化或已损坏 → 重置为当前时间，本帧不做物理更新 */
  if (lastT === null || !isFinite(lastT)) {
    lastT = now;
    return;
  }

  /* 防线 3：dt 通过 safeDt 处理，NaN / 负数 / 超大值全部被驯服 */
  const dt = safeDt(now - lastT, 100);
  lastT = now;
  // ...
}
```

**关键点：** `requestAnimationFrame(frame)` 在函数开头就被调用，所以**即使 return 了，下一帧仍然会被调度**。丢掉一帧（16ms）用户完全察觉不到，但避免了永久卡死。

**第三层：全链路覆盖**

所有涉及时间、增益、功率、电压的计算都改用安全函数：

| 位置 | 之前 | 之后 |
|---|---|---|
| `frame()` dt | `Math.max(0, Math.min(now - lastT, 100))` | `safeDt(now - lastT, 100)` |
| 弹道释放 | `1 - Math.exp(-displayDt / RELEASE_MS)` | `safeK(displayDt, RELEASE_MS)` |
| 功率平滑 | `1 - Math.exp(-displayDt / 160)` | `safeK(displayDt, 160)` |
| 限幅器 | `1 - Math.exp(-dt / tauMs)` | `safeK(dt, tauMs)` |
| 市电相位 | `p.volt += ...` | `p.volt = safeVolt(p.volt)` |
| 累加器 | `+=` 直接累加 | 累加后 `isFinite` 校验 |

**第四层：DOM 输出守卫**

即使在所有计算都经过安全函数之后，DOM 输出前仍然做最后一次校验：

```js
/* updateMainsDisplay 内的电压输出 */
let voltText;
if (isFinite(p.volt)) {
  voltText = p.volt.toFixed(1);
} else {
  p.volt = MAINS_NOMINAL;
  voltText = MAINS_NOMINAL.toFixed(1);
}
cell.voltEl.innerHTML = voltText + '<small>V</small>';
```

**保证任何情况下都不会把 "NaN" 写进界面。**

#### 效果

- **任何 NaN 源头都被堵死** —— 从源头（`now`）到终点（DOM）的每一层都有守卫
- **即使出现意外 NaN 也能自愈** —— `safeVolt` 会恢复标称值，`frame()` 的防线 2 会重置 `lastT`
- **不会永久卡死** —— 丢掉一帧比永久卡死好得多
- **跨平台稳定** —— Windows / macOS / Linux 表现一致

#### 教训

任何对 `requestAnimationFrame` 时间戳做数学运算的地方都**不能假设单调递增**。而且：

- `Math.max(0, NaN)` 返回 `NaN`，不是 `0`
- `Math.min(NaN, x)` 返回 `NaN`
- `NaN` 与任何数的比较都返回 `false`（所以边界检查会全部失效）
- 一旦某个累积器被污染成 `NaN`，如果没有显式重置逻辑，它会**永久保持 NaN**

跨平台代码必须显式钳制，并且**必须对结果做 `isFinite` 检查**，不能依赖数学函数的直觉。

---

## 参数调优

所有可调参数集中在 `<script>` 开头的配置区：

| 参数 | 默认值 | 说明 |
|---|---|---|
| `BASE_REF.full` | `-6` | 全频表头基础参考（dBFS） |
| `BASE_REF.sub` | `-12` | 超低表头基础参考（dBFS） |
| `RELEASE_MS` | `220` | 表头回落时间常数（ms） |
| `CLIP_HOLD` | `1200` | CLIP 指示灯保持时长（ms） |
| `CLIP_LEVEL` | `0.985` | 削顶阈值（线性幅度） |
| `MAX_LIMIT_PCT` | `120` | 功放过载上限（%） |
| `MAINS_NOMINAL` | `237` | 市电标称电压（V） |
| `MAINS_PF` | `0.92` | 功率因数 |
| `LOAD_REFRESH_MS` | `100` | 整体负载刷新间隔（ms） |
| `ENERGY_SAMPLE_MS` | `250` | 能量统计采样间隔（ms） |
| `ENERGY_HISTORY` | `2400` | 能量统计环形缓冲区长度（= 10 分钟） |
| `ENERGY_DISPLAY_MS` | `400` | 能量面板 UI 刷新间隔（ms） |

**刷新率预设**在 `REFRESH_MODES` 中：

```js
const REFRESH_MODES = {
  verylow:  { label: '非常低', intervalMs: 250 },
  low:      { label: '低',     intervalMs: 100 },
  medium:   { label: '中',     intervalMs: 50  },
  high:     { label: '高',     intervalMs: 25  },
  realtime: { label: '实时',   intervalMs: 0   },
};
```

**限制器时间常数**在 `frame()` 内定义：

```js
const tauMs = attacking ? 5 : 250;
```

**整体负载权重**在 `updateTotalLoad()` 内定义：

```js
const rawLoad = maxLoad * 0.55 + avgLoad * 0.45;
```

**分频点**由 UI 选择器控制（20–1000 Hz），初始值 `crossoverFreq = 120`。

**常见调优场景：**

- **希望表头更"恋战"**：`RELEASE_MS` 调到 400–600
- **希望表头更灵敏利落**：`RELEASE_MS` 调到 100–150
- **限幅器更"硬"**：攻击改为 1–2ms，释放改为 400–600ms
- **模拟更激进的功放**：`MAX_LIMIT_PCT` 改为 150
- **市电更"跳"**：电压随机步长 0.22 调到 0.4
- **市电更稳**：电流平滑 150ms 调到 300
- **整体负载对单通道更敏感**：`0.55` 调到 `0.7`
- **整体负载更温和**：`0.55` 调到 `0.4`
- **整体负载刷新更快**：`LOAD_REFRESH_MS` 调到 `50`
- **能量统计刷新更快**：`ENERGY_DISPLAY_MS` 调到 `200`
- **能量统计支持更长窗口**：`ENERGY_HISTORY` 调到 `7200`（30 分钟），同时更新 `avgRecent` 调用处的参数

---

## 扬声器预设

内置的扬声器组合参考了常见专业音响设备的 AES / 节目功率额定值，并附带灵敏度与观众席距离估算：

### 全频通道（Full Range）

| 预设 | 额定功率 | 灵敏度 | 第一排 | 末排 |
|---|---|---|---|---|
| 单 8" | 150 W | 94 dB | 4 m | 20 m |
| 单 10" | 250 W | 96 dB | 6 m | 25 m |
| 单 12" | 400 W | 98 dB | 10 m | 40 m |
| 单 15" | 500 W | 99 dB | 15 m | 50 m |
| 双 8" | 300 W | 97 dB | 10 m | 35 m |
| 双 12" | 800 W | 101 dB | 18 m | 60 m |
| 双 15" | 1000 W | 102 dB | 22 m | 80 m |
| 线阵列 4 组 | 2000 W | 106 dB | 20 m | 80 m |
| 线阵列 6 组 | 3000 W | 108 dB | 22 m | 100 m |
| 线阵列 8 组 | 4000 W | 109 dB | 25 m | 120 m |
| **自定义** | 用户输入 | — | — | — |

### 超低通道（Subwoofer）

| 预设 | 额定功率 | 灵敏度 | 第一排 | 末排 |
|---|---|---|---|---|
| 单 15" | 800 W | 96 dB | 8 m | 30 m |
| 单 18" | 1200 W | 98 dB | 12 m | 45 m |
| 单 21" | 2000 W | 99 dB | 15 m | 50 m |
| 双 18" | 2400 W | 101 dB | 18 m | 65 m |
| 双 21" | 4000 W | 102 dB | 22 m | 80 m |
| 线阵列 4 组 | 4800 W | 106 dB | 20 m | 80 m |
| 线阵列 6 组 | 7200 W | 108 dB | 25 m | 100 m |
| 线阵列 8 组 | 9600 W | 109 dB | 30 m | 130 m |
| **自定义** | 用户输入 | — | — | — |

> **注意：** 这些功率值和灵敏度是"参考值"，用于功率计、限幅器和响度估算的换算基准。实际扬声器的额定功率因品牌、型号、测试标准（AES / IEC / 节目功率 / 峰值功率）不同而差异很大。使用前请以扬声器手册的额定值或系统设计值为准。

如需修改，直接编辑 `SPEAKER_PRESETS` 对象即可。

---

## 版本历史

| 版本 | 主要变更 |
|---|---|
| **V0.1** | 基础版：加载音频、单一模拟电平表（SIG ~ CLIP）、播放控制 |
| **V0.2** | 双声道分离 + 全频 / 超低分频，四通道 LED 阵列 |
| **V0.3** | 四通道增益旋钮（物理拖拽）、表头灵敏度调节、扬声器端功率显示 |
| **V0.4** | 增益旋钮真实作用于音频输出，引入真实电子分频（LR4）架构 ⚠️ *有 bug* |
| **V0.4.1** | 修复 V0.4 音频链路断裂：补上 `src.connect(splitter)`，音频恢复 |
| **V0.5** | 旋钮支持键盘直接输入 dB 值；扬声器组合下拉菜单（含线阵列）；自定义功率 |
| **V0.6** | 功率百分比限制器（0–120%，过载红色警告）；自定义扬声器最大功率输入 |
| **V0.7** | 刷新率选择器（低 / 中 / 高 / 实时），采样与显示解耦 + 峰值累积 |
| **V0.8** | 系统功耗面板（总输出 / 市电消耗 / 效率）；功放类别（AB / H / D）效率模型 |
| **V0.9** | 分频点二级菜单（预设 + 自定义，20–1000 Hz） |
| **V0.10** | 响度估算栏（1m / 前排 / 末排 SPL）；修复限制器输入框宽度 bug（32px → 52px） |
| **V0.11** | 市电三相监测面板（L1 / L2 / L3 电压 + 电流） |
| **V0.11.1** | 新增整体负载指标；修复效率状态条颜色与文本不一致的 bug |
| **V0.12** | **图标系统**（内联 SVG，自动跟随主题）；**能量统计面板**（累计电能 kWh + 5 窗口平均功率 + 重置 + 运行时长）；平均功率改为**下拉选择**；**浅色 / 深色主题切换**（localStorage 记忆）；新增**"非常低"刷新率**（250ms） |
| **V0.12.1** | 修复 Windows 上电压变 `NaN` 的 bug（rAF 时间戳回退导致 `dt < 0` → `Math.sqrt(负)` → NaN）；使用 `Math.max(0, ...)` 钳制 —— ⚠️ *未彻底修复* |
| **V0.13** | **数值安全体系**：引入 `safeDt` / `safeK` / `safeNum` / `safeVolt` / `safeTime` 辅助函数；`frame()` 加入三道防线；全链路覆盖时间、增益、功率、电压计算；DOM 输出前最后校验 |
| **V0.13.1_bugfix** | **当前版本**：修复 V0.12.1 遗留的问题 —— `Math.max(0, NaN)` 返回 NaN 而非 0，导致 `dt` 变成 NaN 后永久污染累积器。用 `safeDt` 彻底替换所有直接钳制；累加后强制 `isFinite` 校验；采样循环加上限防止挂起后死循环 |

### V0.4 的 bug 说明

V0.4 重构音频图为真实分频架构时，漏掉了最上游的一条连线：

```js
src.connect(splitter);   // ← 这行在 V0.4 中缺失
```

导致 `MediaElementSource` 节点悬空，音频既不出声，分析器也读不到任何信号。V0.4.1 补上这一行后恢复正常。

**教训：** 音频图的每一段连接都必须是显式的，Web Audio API 不会为你隐式连接任何节点。

### V0.10 的 bug 修复

功率卡片底部的限制器输入框宽度原为 32px，仅能显示两位数字，`100` 会溢出。修复为 52px 宽 + 字号 10px + 内边距 5px，可以完整显示 `120`。

### V0.11.1 的 bug 修复

效率状态条在效率较低（< 50%）时可能不显示或颜色错位。原因有两处：

1. **状态条类名逻辑与文本相反**：文本用 `warn` 表示低效率，但状态条把 `warn` / `hot` 写反了。
2. **`background` 过渡对渐变无效**：`transition: background` 在渐变上会触发某些浏览器的渲染问题，可能导致一帧背景为空。

修复：状态条直接复用文本的类名；把 `background` 从过渡中移除（渐变本来也不支持平滑过渡）。

### V0.12.1 的 bug 说明（不彻底的修复）

Windows 上（尤其在 Chrome 中）市电电压显示变成 `NaN`。当时的分析：

**`requestAnimationFrame` 的时间戳在某些平台和时刻会轻微回退**（帧率抖动、切标签页、系统时钟微调）。一旦 `now < lastT`，`dt` 就是负数，`Math.sqrt(dt / 16.7)` 返回 `NaN`。

当时的修复是用 `Math.max(0, Math.min(now - lastT, 100))` 钳到非负。**但这个方法有致命缺陷**：`Math.max(0, NaN)` 返回 `NaN`，不是 `0`。所以只要 `now - lastT` 因为任何原因变成 `NaN`，钳制就完全失效。而且一旦 `lastT` 被污染成 `NaN`，之后每一帧 `now - NaN = NaN`，**永久卡死**，只能刷新页面。

这就是为什么"上次修好了，现在又出现"。

### V0.13.1_bugfix 的彻底修复

用 **5 个数值安全辅助函数** + **`frame()` 三道防线** + **全链路覆盖** 重构了数值处理：

**5 个安全辅助函数：**

```js
function safeDt(v, max) {
  if (typeof v !== 'number' || !isFinite(v) || v < 0) return 0;
  return v > max ? max : v;
}
function safeK(dt, tau) {
  if (typeof dt !== 'number' || !isFinite(dt) || dt <= 0) return 0;
  if (typeof tau !== 'number' || !isFinite(tau) || tau <= 0) return 0;
  const k = 1 - Math.exp(-dt / tau);
  return isFinite(k) ? (k < 0 ? 0 : (k > 1 ? 1 : k)) : 0;
}
function safeNum(v, fallback) {
  return (typeof v === 'number' && isFinite(v)) ? v : fallback;
}
function safeVolt(v) {
  if (typeof v !== 'number' || !isFinite(v)) return MAINS_NOMINAL;
  if (v < MAINS_MIN - 20 || v > MAINS_MAX + 20) return MAINS_NOMINAL;
  return v;
}
function safeTime(v, fallback) {
  return (typeof v === 'number' && isFinite(v)) ? v : fallback;
}
```

**`frame()` 三道防线：**

```js
function frame(now) {
  requestAnimationFrame(frame);

  /* 防线 1：now 本身不合法 → 跳过整帧 */
  if (typeof now !== 'number' || !isFinite(now)) return;

  /* 防线 2：lastT 损坏 → 重置为当前时间 */
  if (lastT === null || !isFinite(lastT)) {
    lastT = now;
    return;
  }

  /* 防线 3：dt 用 safeDt 处理 */
  const dt = safeDt(now - lastT, 100);
  lastT = now;
  // ...
}
```

**全链路覆盖：**

- 所有 `1 - Math.exp(-dt/tau)` 都改成 `safeK(dt, tau)`
- 所有 `p.volt += ...` 之后都紧跟 `p.volt = safeVolt(p.volt)`
- 所有累积器（`smoothKnobGain` / `limiterGain` / `row.powerSmooth` / `totalOutSmooth` / `totalConSmooth` / `totalLoadSmooth` / `totalEnergyKWh` / `totalSum`）在每次更新后强制 `isFinite` 校验
- DOM 输出前最后校验一次，绝不写 "NaN"
- 能量统计采样循环加上限（100 次），防止标签页挂起后长时间阻塞

**效果：**

- **任何 NaN 源头都被堵死** —— 从 `now` 到 DOM 的每一层都有守卫
- **即使出现意外 NaN 也能自愈** —— `safeVolt` 恢复标称值，`frame()` 防线 2 重置 `lastT`
- **不会永久卡死** —— 丢一帧比卡死好得多
- **跨平台表现一致** —— Windows / macOS / Linux 都稳定

**教训：**

1. `Math.max(0, NaN)` 返回 `NaN`，不是 `0`
2. `NaN` 与任何数的比较都返回 `false`，边界检查会全部失效
3. 一旦累积器被污染成 `NaN`，如果没有显式重置逻辑，它会**永久保持**
4. 跨平台代码必须显式 `isFinite` 检查，不能依赖数学函数的直觉

---

## 已知限制

- **浏览器自动播放策略**：AudioContext 需要用户手势才能启动，首次加载后必须点击一次
- **多声道文件**：只取前两个声道（左 / 右），5.1 / 7.1 等其他声道被忽略
- **功率模拟是估算**：基于「0 dBFS = 额定满功率」的假设，不反映实际扬声器阻抗、灵敏度、功放增益等因素
- **响度估算忽略房间声学**：基于自由场反平方定律，未考虑室内反射、混响、空气吸收等因素，实际听感通常会略高于估算值
- **市电模型是模拟**：电压随机游走和电流反算都是为了视觉真实感，不反映真实电网行为
- **整体负载是启发式指标**：55/45 的加权比例没有严格的工程依据，是经验上比较能反映"系统整体紧张程度"的折中
- **能量统计是估算**：基于模拟的市电消耗功率积分，不反映真实电表读数
- **不支持实时输入**：目前只能加载本地文件，不支持麦克风 / 线路输入（可通过扩展 `getUserMedia` 实现）
- **移动端触控**：旋钮使用 Pointer Events，支持触控，但小屏幕上旋钮位置较挤
- **不支持音频导出**：这是一个监看工具，不提供离线渲染 / 导出功能

---

## 浏览器兼容性

| 浏览器 | 最低版本 | 状态 |
|---|---|---|
| Chrome / Edge | 66+ | ✅ 完全支持 |
| Firefox | 76+ | ✅ 完全支持 |
| Safari (macOS) | 14.1+ | ✅ 完全支持 |
| Safari (iOS) | 14.5+ | ✅ 完全支持 |
| Opera | 53+ | ✅ 完全支持 |

**依赖的 Web API：**

- Web Audio API（`AudioContext`、`AnalyserNode`、`BiquadFilterNode`、`GainNode`、`ChannelSplitter`、`ChannelMerger`）
- Pointer Events
- CSS 自定义属性、Grid、Gradients
- `localStorage`（用于主题偏好记忆，可选，失败时静默忽略）
- `URL.createObjectURL`

所有 API 都是现代浏览器的标准特性，无需 polyfill。

---

## 许可

MIT License

Copyright (c) 2024 DA-1000 Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

*README 版本：对应 DA-1000 v0.13.1_bugfix*