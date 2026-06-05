# claude-video-translate

> 一句话把英文视频变成**带中文字幕的视频 + 中文文稿**。全程本地跑，转写不花一分钱 API 费。
> Turn any English video into a **Chinese-subtitled video + a clean transcript** with one sentence to Claude. Runs locally, no transcription API cost.

**语言 / Language:** [中文](#中文) ｜ [English](#english)

这是一套给 [Claude Code](https://claude.com/claude-code) 用的**技能（Skills）**。装好后，你不用记任何命令——直接对 Claude 说「把这个 YouTube 链接翻译成中文字幕视频」，它就会自动下载、转写、翻译、烧字幕、出文稿，一条龙做完。

---

## 中文

### 它能做什么

给它一个视频链接（YouTube / Bilibili / 抖音）或一个本地视频文件，它会：

1. **下载** 视频（或直接用你的本地文件）
2. **提取音频 → Whisper 转写**，生成**词级精确时间戳**的原文字幕
3. **翻译 + 润色** 成符合中文观看习惯的字幕（去标点、每行 ≤12 字、术语不乱翻、时间戳严格对齐）
4. **烧录字幕进画面**（可选水印），输出一个带中文字幕的视频
5. **同时生成一份 Markdown 文稿**（中文视频出中文稿；外文视频出原文稿 + 中文译稿）

字幕支持两种：**纯中文** 或 **中英双语**（中文大、英文小，主次分明）。

### 为什么用它

- **本地、免费、可离线**：转写用 [Whisper](https://github.com/openai/whisper)（Apple Silicon 走 MLX + Metal GPU 加速），翻译用你已经在用的 Claude，不调任何付费转写/翻译 API。
- **时间戳真的准**：用词级时间戳按「句子 + 停顿」切分，字幕不会跑在人前面、也不会半句甩到下一条。
- **字幕是专业级的**，不是机翻直出：自动纠正 ASR 听错的专有名词（Claude 被听成 cloud、MCP 被听成 NCP 之类）、按语义断句、术语保留英文。
- **双语字幕是真的双语**：用 ASS 字幕在同一条里做中文大、英文小的字号反差（SRT 做不到这个）。
- **一次编码搞定字幕 + 水印**，不掉画质。

### 演示案例

<!-- DEMO_PLACEHOLDER：此处将放入真实案例的「翻译前 / 后」对比帧、生成的中文 SRT 片段、以及输出视频截图。 -->

*（演示案例即将补充）*

### 环境要求

- **macOS**（推荐 Apple Silicon，转写最快；Intel/其他平台可用 faster-whisper 兜底，但烧字幕的字体命令针对 macOS 调过）
- **Claude Code**（这些是 Claude Code 技能。仓库也附带 `gemini-extension.json`，Gemini CLI 用户可作为 extension 加载）
- 命令行工具：`yt-dlp`、`ffmpeg`、`whisper-cpp`
- Python 转写引擎（任选其一）：`mlx-whisper`（Apple Silicon 首选）或 `faster-whisper`（通用兜底）
- 仅抖音需要：`patchright`（带登录态的浏览器自动化）

### 安装

**第一步：装系统依赖**

```bash
# 没装 Homebrew 的先装：https://brew.sh
brew install yt-dlp ffmpeg whisper-cpp

# 转写引擎（Apple Silicon 选 mlx-whisper）
pip3 install --break-system-packages mlx-whisper
# 非 Apple Silicon 用兜底引擎：
# pip3 install --break-system-packages faster-whisper
```

**第二步：装技能**

```bash
git clone https://github.com/xiaohuailabs/claude-video-translate.git
cd claude-video-translate
bash install.sh
```

`install.sh` 会把三个技能复制到 `~/.claude/skills/`，从模板生成 `config.json`，并检查依赖是否齐全。

**第三步：设置输出目录**

打开 `~/.claude/skills/xiaohu-video-md/config.json`，把 `output_dir` 改成你想要的**绝对路径**（中间文件进 `tmp/`，最终文稿进 `data/`）。

> Whisper 模型：MLX 引擎首次运行会自动从 HuggingFace 下载（约 1.5GB）；用 whisper-cpp 兜底则需手动下载模型，见各技能里的 `初始化.md`。

### 怎么用

装好后，**重启 Claude Code**，然后用大白话对它说就行：

| 你说的话 | 它做的事 |
|---------|---------|
| `把这个链接翻译成中文字幕视频 https://youtu.be/xxxx` | 全流程：下载→转写→翻译→烧中文字幕→出文稿 |
| `翻译这个视频，要中英双语字幕 https://...` | 同上，但字幕是中英双语（中文大英文小） |
| `把这个视频转成文字 https://...` | 只出 Markdown 文稿，不烧字幕 |
| `下载这个视频 https://...` | 只下载视频 + 外挂字幕 |
| `给我本地这个视频加中文字幕 ~/Movies/talk.mp4` | 本地文件直接转写翻译烧录 |
| `用快速模式转写 https://...` | 换更快但略低精度的模型 |
| `翻译时不要水印` | 关掉水印 |

抖音视频第一次用要先登录一次（运行一次 `python3 ~/.claude/skills/xiaohu-video-md/scripts/douyin_login.py`，弹出的浏览器里扫码登录，登录态只存在你本机）。

### 三个技能各管什么

| 技能 | 职责 |
|------|------|
| **xiaohu-video-md** | 编排器。下载 / 提音频 / Whisper 转写 / 调用字幕润色 / 烧录字幕 / 生成 Markdown |
| **xiaohu-subtitle-polish** | 字幕翻译与润色。纠错、翻译、断句、去标点、时间戳对齐、双语 ASS |
| **xiaohu-video-download** | 纯下载工具。下视频 / 下音频 / 下播放列表 / 给本地视频烧字幕 |

翻译管线由 `xiaohu-video-md` 编排，翻译环节自动调用 `xiaohu-subtitle-polish`。三个技能各自独立，可单独使用。

### 常见问题

- **YouTube 下载报 403 / SABR / PO Token？** 脚本会自动从你的浏览器读 cookies 重试（默认 Chrome）。还不行就挂代理：在对话里说明，或给脚本加 `--proxy http://127.0.0.1:7890`。
- **烧出来的中文字幕是方块？** macOS 的 fontconfig 索引不到苹方，水印用的是圆体（Yuanti SC）。脚本已处理；如果你改了字体，确认指定的是 `fontfile=` 绝对路径而不是字体名。
- **字幕跑在说话人前面 / 半句挤一起？** 本工具用词级时间戳按句子和停顿切，正常不会出现；若个别片段不对，多半是音频里 BGM 太响导致 Whisper 误判，可在对话里要求重转。
- **抖音报未登录？** 重跑一次 `douyin_login.py` 登录。

### License

[MIT](./LICENSE)。脚本里的字幕样式、水印默认值等可以随意改。注意：**不要把你自己的 `config.json` 或抖音登录态提交到任何公开仓库**（`.gitignore` 已默认排除）。

---

## English

A set of [Claude Code](https://claude.com/claude-code) **Skills** that turn any video into a Chinese-subtitled video plus a clean transcript. Once installed, you just tell Claude in plain language — no commands to memorize.

### What it does

Give it a video URL (YouTube / Bilibili / Douyin) or a local file, and it will:

1. **Download** the video (or use your local file)
2. **Extract audio → transcribe with Whisper**, producing **word-level accurate timestamps**
3. **Translate + polish** into natural Chinese subtitles (punctuation stripped, ≤12 chars/line, terms kept in English, timestamps strictly aligned)
4. **Burn the subtitles into the video** (optional watermark)
5. **Also generate a Markdown transcript** (original + Chinese translation for non-Chinese videos)

Subtitles can be **Chinese-only** or **bilingual** (Chinese large, English small — using real ASS, which SRT can't do).

### Why

- **Local, free, offline-capable.** Transcription runs on [Whisper](https://github.com/openai/whisper) (MLX + Metal GPU on Apple Silicon); translation uses the Claude you already have. No paid transcription/translation API.
- **Timestamps are actually accurate** — word-level cut by sentence + pause, so subtitles don't run ahead of the speaker.
- **Professional subtitles**, not raw machine output — fixes ASR mishears of proper nouns, breaks lines by meaning, keeps technical terms in English.
- **Burn-in + watermark in a single encode**, no quality loss.

### Requirements

- **macOS** (Apple Silicon recommended; faster-whisper works elsewhere, but burn-in font commands are tuned for macOS)
- **Claude Code** (a `gemini-extension.json` is also included for Gemini CLI users)
- CLI tools: `yt-dlp`, `ffmpeg`, `whisper-cpp`
- Python engine (pick one): `mlx-whisper` (Apple Silicon) or `faster-whisper` (fallback)
- Douyin only: `patchright`

### Install

```bash
# 1. System deps
brew install yt-dlp ffmpeg whisper-cpp
pip3 install --break-system-packages mlx-whisper   # or faster-whisper

# 2. Install the skills
git clone https://github.com/xiaohuailabs/claude-video-translate.git
cd claude-video-translate
bash install.sh
```

Then set `output_dir` (an absolute path) in `~/.claude/skills/xiaohu-video-md/config.json`. The MLX model auto-downloads from HuggingFace on first run.

### Usage

Restart Claude Code, then just say things like:

- *"Translate this link into a Chinese-subtitled video: https://youtu.be/xxxx"*
- *"Translate this with bilingual subtitles: https://..."*
- *"Just transcribe this video to text: https://..."*
- *"Add Chinese subtitles to my local file ~/Movies/talk.mp4"*
- *"Translate it, no watermark"*

### The three skills

| Skill | Role |
|-------|------|
| **xiaohu-video-md** | Orchestrator: download / audio / Whisper / call the polisher / burn-in / Markdown |
| **xiaohu-subtitle-polish** | Subtitle translation & polishing: fixes, translation, line-breaking, timestamp alignment, bilingual ASS |
| **xiaohu-video-download** | Pure downloader: video / audio / playlists / burn subs onto a local file |

### License

[MIT](./LICENSE). Don't commit your own `config.json` or any Douyin login state to a public repo (already excluded by `.gitignore`).
