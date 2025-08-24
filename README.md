# astrabot_plugin_tts_emotion_router

按情绪路由到不同音色与语速的 TTS 插件（硅基流动 API，其他OpenAI语音接口api同样适配）。
可配合STT插件：https://github.com/NickCharlie/Astrbot-Voice-To-Text-Plugin 实现和Bot全语音对话。

## 功能特性
- 将 LLM 文本回复按概率/长度/冷却等门控转换为语音（Record），失败回退文本。
- 情绪分类仅采用两级策略：
  - 隐藏情绪标记（推荐）：在回复末尾追加形如 `[EMO:happy|sad|angry|neutral]` 的隐藏标记；插件自动解析并移除；
  - 启发式规则：无标记时，用关键词/标点等规则推断情绪；
- 情绪 → 音色与语速联动：`voice_map`、`speed_map`；
- 会话控制指令：`tts_global_on/off`、`tts_on/off`、`tts_prob`、`tts_limit`、`tts_cooldown`、`tts_status`；
- 过滤与门控：文本长度、混合内容开关、链接/文件提示过滤、触发概率与冷却；
- 硅基流动 TTS：POST `{api.url}/audio/speech`，带重试与缓存；音频落地到 `data/plugins/tts_emotion_router/temp/<sid>/`。


## 安装
1. 将本目录放入 `data/plugins/tts_emotion_router/`。
2. 安装依赖（项目根目录）：`uv sync`（已包含 `requests`）。
3. 启动 AstrBot：`uv run main.py`（WebUI 默认 http://localhost:6185）。
4. 首次使用请先安装 ffmpeg，并将其加入 PATH 环境变量。

## 面板配置
- API（必填）
  - `api.url`: 例如 `https://api.siliconflow.cn/v1`
  - `api.key`: 你的 API Key（不要提交到公共仓库）
  - `api.model`: 例如 `FunAudioLLM/CosyVoice2-0.5B`
  - `api.format`: `mp3`（默认，跨端播放更稳）
  - `api.speed`: 默认语速（1.0）
- 情绪策略（无需小模型）
  - `emotion.marker.enable`: 启用隐藏标记解析
  - `emotion.marker.tag`: 默认 `EMO`，对应标记 `[EMO:happy]`
  - `emotion.marker.prompt_hint`: 可复制进人格/系统提示的说明
- 路由
  - `voice_map.neutral/happy/sad/angry`: 为不同情绪配置音色，至少填写 `neutral`
  - `speed_map.neutral/happy/sad/angry`: 情绪语速倍率（如 happy=1.2, sad=0.85）
- 其他
  - `global_enable`、`enabled_sessions`、`disabled_sessions`
  - `prob`、`text_limit`、`cooldown`、`allow_mixed`
  -<img width="980" height="2309" alt="7cc5891f4c28a5d4c80ac433a9801922" src="https://github.com/user-attachments/assets/263b7a93-096d-4b95-b418-796bcf30a9cd" />


## 使用建议
- 在人格或系统提示末尾加入：
  - “请在每次回复末尾追加形如 `[EMO:<happy|sad|angry|neutral>]` 的隐藏标记。该标记仅供系统解析，不会展示给用户。”
- 开启 `emotion.marker.enable`；填写 `voice_map` 与 `speed_map`；保存并重载插件。
- 发送短句测试不同情绪，验证音色/语速切换。

## 硅基流动一键上传音色工具
不会上传硅基cosyvoice音色？直接到 [Releases](https://github.com/muyouzhi6/astrabot_plugin_tts_emotion_router/releases/tag/v0.1.1) 页面下载 exe 附件，双击即可用！
需要准备的东西：5Mb以下的10s左右的清晰干净的人声素材，音频所对应的文本，一个硅基流动apikey（音色绑定apikey，建议单独开一个apikey）
没有素材也可以用硅基官方音色：FunAudioLLM/CosyVoice2-0.5B:anna

## 指令速查
- `tts_global_on` / `tts_global_off` — 全局开关（黑名单/白名单模式）
- `tts_on` / `tts_off` — 当前会话开/关
- `tts_prob <0~1>` — 触发概率
- `tts_limit <非负整数>` — 文本长度上限
- `tts_cooldown <秒>` — 触发冷却
- `tts_status` — 查看当前会话与参数

## 常见问题
- 没有音频但有文本：
  - 检查 `api.url/key/model` 是否正确；
  - 查看日志是否 429/5xx 被重试后仍失败；
  - 确认 `voice_map.neutral` 已配置；
- 情绪不切换：
  - 优先使用隐藏标记；若没有标记则使用启发式，可能不如小模型细腻；
- 路由不生效：
  - 检查 `prob`/`text_limit`/`cooldown` 是否限制触发；

## 版本与作者
- 作者：木有知
- 版本：0.1.1
- 许可证：MIT（随 AstrBot 项目）


