# 粤普同传

粤语与普通话双向翻译的 Windows 桌面实验版，默认使用本地模型，可选联网识别与翻译，提供双栏字幕、本地会话历史和字幕导出。

[查看更新日志](docs/CHANGELOG.md) · [仓库、分支和发布说明](CONTRIBUTING.md#仓库分支和发布)

**当前为开发预览。** Windows 已有包含私有 Python 和离线模型的本地 Inno Setup 安装包构建流程，见 [Windows 打包与验证说明](installer/windows/README.md)；尚未完成全新系统安装与公开发布验收。Android 可构建 ARM64 调试 APK，真机收音、性能和音质尚待验收，参见 [Android 开发说明](android/README.md)。Windows pro 复核与联网增强为实验功能。联网接口已接入并验证本地模拟服务及本地回退，真实百炼账户的权限、准确率和延迟仍需实测。

Android 版提供离线文字和麦克风翻译、本机会话历史及深浅主题。手机使用底部输入区和会话弹层，宽窗口可展开会话栏。已导入的离线模型会在打开应用时后台预加载。参见 [Android 使用与构建说明](android/README.md)。

## 已有功能

- 文字翻译和实时收音共用一个输入区，收音选项集中在输入区的下拉菜单。麦克风连续收音可生成粤语 ⇄ 普通话字幕及译音。**pro** 保留原段落检测和断句，轻量模型先显示暂定原文和粗略译文，句末较大模型复核后固定原文、修正译文并播报。原轻量会话模式保留为 **flash**。
- Windows 默认输出设备的声音翻译，适用于浏览器、视频及网课等播放内容。收音期间只显示字幕，停止后可播放译音，避免再次识别自己的声音。
- 处理粤语、普通话和英文识别结果；遇到日语、韩语标签或相关文字会按所选方向重试或拒绝。底层模型本身仍是多语言模型。
- 原文、译文按条配对并按会话自动保存到本机，重新打开软件可从左侧历史回看。每个会话最多保留 10,000 条、合计 2,000,000 字符，达到任一上限后移除该会话最早的记录。
- 向上滚动时暂停跟随新字幕；可以返回最新位置、复制及手动导出。停止和切换方向会保留当前会话。
- pro：轻量草稿不提前固定，可先显示粗略译文；Qwen3-ASR-1.7B 在原有检查点保留跨窗前文，停顿或原分段上限到达后确认当前段落，确认后修正译文，已有确认前文不重译。麦克风和设备声音均可使用。
- 设置可保存默认模式、方向、音源、播报与阅读选项，以及临时缓存和字幕导出目录（原段落修订首选项自动对应 pro）。临时缓存默认使用系统临时目录，更换位置不搬动或删除旧文件。
- 实时收音提供“标准／灵敏”两档并记住选择：标准默认用于嘈杂环境，灵敏用于手机外放、轻声和短句。pro/flash 与两种音源均可使用，停止收音后切换；不能只锁定指定说话人。
- 歌曲文件支持最多 10 分钟 / 100 MB 的 MP3、WAV、FLAC、OGG；设置中的人声分离为可选高质量处理，只用于文件，不增加实时收音开销。
- 可选百炼联网增强：Qwen3-ASR-Flash 识别、Qwen-MT-Flash 翻译，支持麦克风、设备声音、歌曲文件及短句输入。用户自行提供密钥并承担费用；本次任务第一次云端失败后转为本地处理，不自动重试。

会话历史只保存文字字幕及配对所需的信息。需要独立的文本文件时可手动导出。设备声音输入采集默认输出的混合声音，暂不支持只选择一个应用。

## 开发环境与启动

当前支持 Windows x64、Python **3.12**。首次准备需要联网，准备完成后可离线推理。源码初始化按所需功能选择模型，Python 依赖和本地推理运行库另计。

安装 Python 3.12 x64 后，双击 **`Initialize-YuePu.cmd`**。选 T 只装文字翻译（模型约 1.06 GiB），选 F 加上基础收音与双向播报（约 1.96 GiB），选 P 再加 pro 复核（约 4.31 GiB，默认）。这些数字为本机模型的解压后大小。也可运行 `.\Initialize-YuePu.cmd --plan text` 预览清单，或将 `text`、`flash`、`pro` 作为参数直接初始化。升级档位时会复用已校验的文件。看到 `Setup complete` 后，双击 **`Start-YuePu.cmd`**。

也可在仓库根目录打开 PowerShell，手动创建环境并安装依赖。

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -e ".[offline,dev]"
```

基础模型和默认 pro 模式的复核模型要分两次下载。每次下载上限为 4 GiB，两次都显示 `Prepared and verified requested assets.` 后再启动。

```powershell
.\.venv\Scripts\python.exe experiments\offline-poc\prepare_models.py asr canto hymt2 runtime mandarin vad
.\.venv\Scripts\python.exe experiments\offline-poc\prepare_models.py song-asr-large
```

准备完成后，双击根目录的 `Start-YuePu.cmd`，也可以在 PowerShell 中启动。

```powershell
.\.venv\Scripts\python.exe -m yuepu_translate
```

已有 `experiments/offline-poc/.venv` 的本地环境可继续使用 `Start-YuePu.cmd`，并复用已准备的模型。

数据目录可用 `YUEPU_DATA_DIR` 指定，必须为绝对路径。源码运行默认使用 `experiments/offline-poc/`，独立安装的 Python 包默认使用 `%LOCALAPPDATA%/YuePuTranslate/`。如果在 PowerShell 中临时设置了这个变量，请在同一个窗口下载模型并启动应用，让两者使用同一目录。

```powershell
$env:YUEPU_DATA_DIR = "D:\YuePuData"
```

操作与故障处理见 [Windows 使用说明](docs/windows.md)。依赖锁定的是直接依赖版本，尚未提供完整的传递依赖锁文件；全新机器安装和运行仍需持续验证。

## 联网增强（实验）

准备基础模型及 `song-asr-large` 后，打开“设置 → 联网设置…”，填写百炼 API Key，选择对应的北京或新加坡地域。需要时填写业务空间 ID。勾选“启用联网增强”并保存，再开始收音、打开音频或输入文字。

默认离线；即使加密记住密钥，每次启动也须重新开启联网。仅在用户发起处理后发送近期音频和待翻译文字，连接测试只发送合成静音及测试短句。滚动修订可能重复提交同一部分内容并产生费用。译音仍在本地生成。详细步骤、上传范围和计费边界见 [联网增强说明](docs/online.md)。

## 项目结构

```text
src/yuepu_translate/          应用包：Qt 窗口、音频引擎、字幕、模型加载
tests/                       不需要模型或音频设备的回归检查
android/                     独立离线 Android 工程、JVM 检查和模型准备工具
experiments/offline-poc/      模型准备和可选的真实模型诊断；旧启动入口
docs/                        用户及开发说明
.github/                     CI、Issue 和 PR 模板
```

模型、运行时二进制、录音、导出内容、测试输出、环境缓存和内部开发笔记均不纳入 Git。实验脚本不属于稳定 API，也不作为应用运行依赖。

## 本地检查

```powershell
.\.venv\Scripts\python.exe -m ruff check .
.\.venv\Scripts\python.exe -m ruff format --check .
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
.\.venv\Scripts\python.exe -m build
```

CI 只运行静态检查、无模型测试和打包；联网契约测试仅连接本机 HTTP 模拟服务，不调用真实云端、不使用真实密钥、不下载模型、不采集声音。真实模型验证方法及证据边界见 [开发说明](docs/development.md)。

## 模型与限制

flash 链路为 WenetSpeech-Yue U2++ CTC INT8 → Hy-MT2-1.8B Q4_K_M → Canto-TTS-Nano / MeloTTS 普通话，使用 Silero VAD 断句。基础识别与普通话音色已更换为许可明确的模型，旧环境运行 `python experiments/offline-poc/prepare_models.py asr mandarin` 准备；不会回退到旧 SenseVoice 或 Huayan 权重。新 CTC 不提供音频语言识别标签，翻译方向由用户选择，无法保证排除误听成中文的其他语言音频。文件来源和固定版本见 [第三方组件说明](THIRD_PARTY.md) 及模型准备脚本。

pro（原段落修订）为默认选项，需准备额外模型；仅用 flash 时无需此模型：

```powershell
.\.venv\Scripts\python.exe experiments\offline-poc\prepare_models.py song-asr-large
# 可选，仅用于“设置 → 首选项… → 阅读与播报 → 人声分离”的歌曲文件处理
.\.venv\Scripts\python.exe experiments\offline-poc\prepare_models.py song-separator
```

flash 收音约在首个 0.8 秒音频窗口后尝试显示原文，之后约每 0.64 秒更新；这是采样节奏，实际等待还包含人声检测、排队和模型推理，不能作为延迟保证。短暂结果可能改字；停顿或连续约 8 秒后确认一条，开始下一条。翻译只保留最新待处理草稿和有界的确认任务，迟到的旧译文不会覆盖已更新的原文。此模式复用基础模型，无需 `song-asr-large`；文件输入仍按短音频一次处理。

原文及译文用正文颜色标出已确认内容，灰色表示暂定内容，并保留文字状态提示。pro 的轻量草稿和粗略译文不锁定；粗译限频并合并旧任务，句末复核后生成这条记录的确认译文，已有确认前文不改写、不重译。flash 保留原有稳定前文和暂定译文缓存。自动确认不能代替人工核对准确性。

再次点击收音按钮会停止采集并确认末尾段落，也可按 Esc 立即取消未完成处理。字幕会标明暂定、前文已固定或已确认。句末由人声停顿判断，仍可能误判；标点复核不保证歌词正确。歌曲文件处理不自动播报，可完成后手动播报选中译文。

目前使用 CPU 推理；速度随设备和内容变化。对话、歌词、伴奏和混响的效果差异明显，未提供通用准确率、手机延迟或长期运行保证。flash 模式文件输入仅支持不超过 15 秒的 WAV，单次文本输入最多 120 字符。pro 保留原段落方案：停顿阈值约 2 秒、2 秒前导、8 秒识别上下文及 2 秒重叠，连续约 15 秒后另起一条；仅轻量草稿改为首个约 0.8 秒窗口、之后约每 0.64 秒更新，文件处理仍使用 30 秒窗口。这不是模型原生流式识别；固定前文留作显示和保存，旧音频退出窗口后不再进入识别。人声分离耗时较长。离线保护包含 Python 外部套接字拦截和 llama.cpp 离线参数，尚不能代替完整的操作系统断网审计。

Windows 在首次使用时加载对应模型。文字短句和实时字幕共用 Hy-MT2，切换粤普方向也可复用。开启播报或重播译文时才加载目标音色。收音遇到持续积压时，应用处理到最后接收的位置，并在会话和导出中标明中断边界。

## 参与和授权

欢迎提交可复现的问题和小范围改进，请先阅读 [贡献指南](CONTRIBUTING.md)、[行为准则](CODE_OF_CONDUCT.md) 和 [安全说明](SECURITY.md)。请勿在公开 Issue 上传私人录音、密钥或未获授权的歌曲。

本项目原创源码按 [MIT](LICENSE) 许可，可修改和商用。第三方模型、库和运行时保留各自许可。模型来源、许可及作者授权证据见 [模型许可目录](licenses/models/README.md)。SenseVoice 与 Android 原粤语权重已退出当前运行／发布配置；Android 粤语改用用户合法安装的离线系统音色。两个平台的当前模型许可检查均可通过，但这不是整个软件的发布批准；旧安装附件仍须重建并核验。
