# 课程 HTML 页面搭建规范（Ivan Learning 出品标准）

本文档固化第 1/2 课打磨出的全部工程经验，任何模型按此规范 + `assets/lesson-template.html` 派生，产出质量与官方课程一致。先读 `clip-selection.md`（选片段/核台词），再按本文档搭建页面。

## 0. 产物形态（硬性）

- **单文件自包含 `.html`**：base64 内嵌视频 + 音频 + poster，零外部依赖，手机/电脑直接打开。
- 移动端优先：`.wrap` max-width 720px，桌面居中。
- 播放全部由用户点击触发，禁止自动播放；按钮 ≥44px 触达。
- 无 emoji；页面注释与讲解用中文，台词英文原样。

## 1. 页面结构（三区 + 数据模型）

### 1.1 DOM 骨架（与模板一致，id 不可改）

```html
<header class="hero">
  <div class="kicker">…4 个 tag：第 N 课 / 剧集出处 / 等级 / 时长…</div>
  <h1>英文名场面 <span class="h-mark">梗点</span></h1>
  <p>一句话简介</p>
  <nav class="steps"><a href="#blind"><span class="no">1</span>盲听</a><a href="#explain"><span class="no">2</span>精讲</a><a href="#intensive"><span class="no">3</span>精听</a></nav>
</header>

<section id="blind">
  <div class="shead"><span class="no">01</span><h2>盲听</h2><span class="en">Blind listening</span></div>
  <div class="vwrap"><video id="bv" playsinline preload="auto" controls poster="…"></video></div>
  <div class="bigplayer">
    <button class="bigbtn" id="blindBtn">播放整段视频</button>
    <div class="bar"><i id="blindBar"></i></div>
  </div>
  <div class="qlist"><h3>听之前想一想</h3><ol>…3-5 个引导问题…</ol></div>
  <p class="hint">先看画面听 2-3 遍，不急着看下面的文字。</p>
</section>

<section id="explain">
  <div class="shead"><span class="no">02</span><h2>精讲</h2><span class="en">Line-by-line analysis</span></div>
  <div id="cues"></div>            <!-- JS 渲染台词卡 -->
  <div class="pron">…发音对比卡（本课有对比才放）…</div>
</section>

<section id="intensive">
  <div class="shead"><span class="no">03</span><h2>精听</h2><span class="en">Intensive listening</span></div>
  <div class="panel">
    <div class="pdisp"><div class="pnum" id="pnum">第 1 / N 句</div><div class="pen" id="pen">—</div><div class="pzh" id="pzh"></div></div>
    <div class="pbar"><i id="pbar"></i></div>
    <div class="pbtns">
      <button class="b" id="prevBtn">上一句</button>
      <button class="b playbtn" id="playBtn">播放</button>
      <button class="b" id="nextBtn">下一句</button>
    </div>
    <div class="row2">
      <button class="loopbtn" id="loopBtn" aria-pressed="false">单句循环</button>
      <div class="rates">…4 个变速钮 data-rate="0.5|0.75|1|1.25"，1× 默认 aria-pressed="true"…</div>
      <label class="vol">音量 <input id="vol" type="range" min="0" max="100" value="80"></label>
    </div>
  </div>
</section>

<footer>…台词校准口径 + 版权说明…</footer>
<audio id="a" preload="auto"></audio>
```

### 1.2 数据模型（script 内）

```js
var CUES = [
  {who:"B", en:"英文原句", zh:"中文翻译", s:0.4, e:1.6,
   note:"解析 HTML：发音/连读（写音标或拟音）+ 词汇 + 常用词组（短语搭配，必写）+ 句型 + 文化梗 + 造句示例 1-2 个（六要素，v1.2 起强制，见 lesson-template.md Step 2）"}  // 硬规则：词组/句型/连读示例必须逐字来自该句原句；速查卡只列本课实际出现的词；扩展词显式标注"本课未出现，扩展知识",
  // who 取值：S=Stewie / B=Brian / O=军官（新说话人加 badge 类，见 2.3）
];
var FULL_END = 67.8;   // 音频总长（秒）
var AUDIO_B64 = "…";   // mp3 或 m4a 的 base64（本体！）
var VIDEO_B64 = "…";   // mp4 的 base64（本体！）
```

## 2. 视觉规范（CSS，模板已含完整样式，派生时禁止随意改）

### 2.1 配色（CSS 变量）

| 变量 | 值 | 用途 |
|---|---|---|
| `--bg` | `#f2f4f8` | 页面底 |
| `--card` | `#ffffff` | 卡片底 |
| `--ink` | `#1b2430` | 主文字 |
| `--sub` | `#5b6472` | 次要文字 |
| `--line` | `#e2e7ef` | 边框 |
| `--blue` | `#1d6fe0` | 主色（按钮/高亮） |
| `--blue-soft` / `--blue-line` | `#e8f0fe` / `#bcd3f6` | 选中背景/边框 |
| `--amber` | `#c2620a` | 精听进度/对比卡 B 侧 |
| `--ok` | `#1a7f4b` | 第三说话人 badge |

### 2.2 字体与基础

```css
body{font-family:-apple-system,BlinkMacSystemFont,"PingFang SC","HarmonyOS Sans SC","Microsoft YaHei","Segoe UI",sans-serif;line-height:1.6;-webkit-font-smoothing:antialiased}
```
h1 34px/800，shead h2 21px/800，正文 14-15.5px，注释 13-13.5px。`button{font:inherit;…}`，`:focus-visible{outline:2px solid var(--blue)}`。

### 2.3 组件要点

- **hero**：顶部渐变 `linear-gradient(180deg,#e7eefc,var(--bg))`；kicker 圆角 tag；steps 三个卡片按钮 `flex:1 1 150px;min-height:44px`。
- **盲听**：`bigbtn` 全宽 56px 蓝色大按钮，播放中变 `--ink` 并显示"暂停"；`bar` 6px 进度条由 `blindBar` 的 width 驱动；video 容器 `.vwrap` max-width 420px 居中，`video{aspect-ratio:3.05/1;object-fit:cover;border-radius:12px;background:#000}`（横版 16:9 剪辑按实际比例，竖版如 360x480 用 3/4）。
- **精讲卡片**：`.cue` 左侧 4px 色条，`.cue.active` 变蓝底；`badge` 三态——`.badge.S` 蓝(Stewie) / `.badge.B` 橙(Brian) / `.badge.O` 绿(其他)；`notebtn` 44px 方钮，展开后 `aria-expanded=true` 变黑底；`.cue.playing .en` 变蓝（播放中的句子高亮）。
- **发音对比卡**：`.pbox` 蓝底主发音 / `.pbox.alt` 橙底对比发音，`ipa` 用等宽字体。
- **精听面板**：`playbtn` 蓝色放大 1.6 倍宽；`loopbtn[aria-pressed=true]` 蓝底；rate 钮选中 `--ink` 底；进度条 `.pbar i` 用 `--amber`（与盲听蓝区分）。

## 3. 播放器双轨逻辑（JS 契约，模板已含完整实现）

- **audio 是主播放器**（精讲点句、精听、变速、循环都走 audio）；**video 只服务盲听**。两者音量联动（`vol` 滑块同时设 `audio.volume` 与 `bv.volume`）。
- **cue 时间轴防串句**（模板函数，勿改）：
  ```js
  function cueStart(i){ return clamp(CUES[i].s - 0.05, 0, CUES[i].s + 0.5); }
  function cueEnd(i){
    var e = CUES[i].e + 0.08;
    if (i + 1 < CUES.length) e = Math.min(e, CUES[i+1].s - 0.03);
    return clamp(e, CUES[i].e, FULL_END);
  }
  ```
  每句起点提前 0.05s 不丢字，终点多收 0.08s，且不越过下一句起点 0.03s。
- **模式互斥**：`mode = "full" | "cue"`；播 cue 先 `bv.pause()`，播 full 先 `audio.pause()`；`setPlaying(p)` 统一同步两个按钮文案与 `playing` 态。
- **timeupdate 驱动**：cue 模式到 `cueEnd` 时——`loopOn` 则回跳 `cueStart`，否则暂停复位；full 模式用 `bv.duration || FULL_END` 驱动盲听进度条（video duration 在部分环境读不出，兜底常量）。
- **变速**：`audio.playbackRate = rate`（0.5/0.75/1/1.25），rate 按钮 `data-rate` + `aria-pressed` 单选。
- **空格键**：播放/暂停当前模式；`e.target` 为 BUTTON/INPUT/VIDEO 时忽略（防止点按钮后空格误触）。
- 所有按钮挂真实 handler；`aria-label` 齐全；播放必须由点击触发。

## 4. 台词时间轴三重校准（±0.3s，产线必做）

1. **字幕站全文**：subslikescript / springfieldspringfield 搜全集脚本，确定场景起止、说话人、逐字台词。
2. **视频硬字幕 OCR**：下载 B 站剪辑后抽帧（1-5s 间隔），用图片识别读硬字幕英文——剪辑常跳句，硬字幕即"实际音频里有什么"的证据。
3. **云端 ASR 逐句校准（推荐必做）**：mediakit-cli `video asr-subtitles`（eng-US）对裁剪音频做识别，拿带时间戳的逐句文字；快速对吵、连读句 ASR 能听出**真实句子数和边界**——避免"漏句/对不上"。
4. **三方比对**：ASR 时间轴为主、硬字幕 OCR 校准内容、字幕站全文兜底说话人与上下文；以**实际音频**为准（剪辑删句时标注差异）。逐句 `s/e` 与音频对齐（±0.3s），页脚注明口径。
5. **说话人归属**：多说话人时按角色分配 who；ASR 的 speaker 字段可参考，最终以剧情上下文人工确认。

## 5. 片源查找与准备

1. **找片**：`site:bilibili.com 剧集名+场景/梗关键词` 搜剪辑或合集（常带硬字幕可作证据）；YouTube 备选。用 doubao-video-extract 下载（`social_video_to_minutes.py --media-mode video`）。
2. **定位场景**：对下载视频抽帧/OCR 定位目标场景时间轴；从 B 站合集里找目标片段（如第 2 课在 52 分钟合集的 1008.0–1075.8s）。
3. **裁字幕（源带硬字幕时）**：抽帧确认字幕条坐标——**顶部 UP 主中文旁白条和底部双语条都要查**（横版 640x360 示例：顶部 y0-45、底部 y255+，保留中间 → `crop=640:210:0:45`；竖版 360x480 示例：底部 y320-480 → `crop=360:320:0:0`）。
4. **裁视频（无字幕盲听片）**：
   ```bash
   ffmpeg -ss <起> -to <止> -i src.mp4 -vf "crop=<w>:<h>:<x>:<y>" -c:v libx264 -crf 26 -preset veryfast -pix_fmt yuv420p -c:a aac -b:a 96k -movflags +faststart blind.mp4
   ```
   **必须 `-movflags +faststart`（moov 前置），否则内嵌 base64 视频无法播放**。30s 片段目标 ≤1MB。
5. **裁音频**：同段 `-vn -acodec libmp3lame -q:a 4` 或 m4a（base64 更小用 m4a 亦可，模板 audio src 按实际改）。
6. **poster**：裁盲听视频首帧 `ffmpeg -i blind.mp4 -frames:v 1 poster.jpg`，压到 20-50KB。
7. 版权边界：只裁 1-2 分钟短片用于个人学习，不下载/不传播完整剧集。

## 6. base64 内嵌（重点防坑）

```js
var AUDIO_B64 = "…"; var VIDEO_B64 = "…"; var POSTER_B64 = "…";
audio.src = "data:audio/mpeg;base64," + AUDIO_B64;   // src 行写死，勿动
bv.src = "data:video/mp4;base64," + VIDEO_B64;
poster = "data:image/jpeg;base64," + POSTER_B64;
```

- **替换变量本体，禁止只改 src 前缀**（第 2 课真实事故：只改 src 会把新 base64 拼进上一课旧变量，学员听到上一课内容）。
- base64 从产物文件生成：`base64 -w0 blind.mp4`，嵌入后行内单引号包裹。

## 7. 生成后验证（三重 + 浏览器）

1. **字节一致**：从 HTML 提取 `AUDIO_B64`/`VIDEO_B64` 解码回文件，与磁盘源文件 `cmp` 逐字节一致。
2. **时长一致**：`ffprobe` 音频时长 = 视频时长 = `FULL_END`（误差 ±0.1s）。
3. **页面自检**：按 html Skill 跑 `shot.py`（桌面 + 移动截图、console 错误、横向溢出、资源错误），全部为空才交付。
4. **浏览器实测**：playwright 打开 `file://`，`audio.duration` 应为 FULL_END（video.duration 在该环境受 URL safety 限制读不出，属环境限制；视频正确性以"编码参数与上一课可播视频一致 + moov 前置 + 字节一致"兜底）。验证 `#盲听/精讲/精听` 锚点、变速钮、上下句、循环、音量联动。

## 8. 常见问题速查

| 现象 | 原因与修法 |
|---|---|
| 视频黑屏/无法播放 | moov 在文件尾 → 重新 ffmpeg 加 `-movflags +faststart` |
| 点句播的是上一课内容 | base64 变量本体没替换 → 替换 AUDIO_B64/VIDEO_B64 定义处 |
| 漏句/语音文字对不上 | 没做 ASR 三方校准 → 补 mediakit asr-subtitles，按真实句子数重拆 CUES |
| 盲听画面有字幕 | 只裁了底部没裁顶部旁白条 → 上下边缘都抽帧确认再 crop |
| video.duration 为 NaN | 环境 headless 限制，非文件问题；按编码参数兜底判断 |
| 三区锚点点了没反应 | steps href 与 section id 不一致 → 核对 `#blind/#explain/#intensive` |
