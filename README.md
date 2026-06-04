# Hermes 飞书流式卡片插件

[中文](README.md) | [English](README.en.md)

<p align="center">
  <a href="https://github.com/baileyh8/hermes-feishu-streaming-card/stargazers"><img alt="GitHub stars" src="https://img.shields.io/github/stars/baileyh8/hermes-feishu-streaming-card?style=for-the-badge&logo=github&label=Stars&color=2f80ed"></a>
  <a href="https://github.com/baileyh8/hermes-feishu-streaming-card/releases"><img alt="Latest release" src="https://img.shields.io/github/v/release/baileyh8/hermes-feishu-streaming-card?style=for-the-badge&logo=githubactions&label=Release&color=22c55e"></a>
  <a href="https://github.com/baileyh8/hermes-feishu-streaming-card/actions/workflows/tests.yml"><img alt="Tests" src="https://img.shields.io/github/actions/workflow/status/baileyh8/hermes-feishu-streaming-card/tests.yml?branch=main&style=for-the-badge&label=Tests&logo=githubactions"></a>
  <img alt="Python 3.9+" src="https://img.shields.io/badge/Python-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="Feishu/Lark" src="https://img.shields.io/badge/Feishu%20%2F%20Lark-Streaming%20Cards-00D6B4?style=for-the-badge">
  <img alt="Sidecar only" src="https://img.shields.io/badge/Runtime-Sidecar--only-7C3AED?style=for-the-badge">
  <a href="LICENSE"><img alt="License" src="https://img.shields.io/github/license/baileyh8/hermes-feishu-streaming-card?style=for-the-badge&color=64748b"></a>
</p>

![Hermes Feishu Streaming Card 封面](docs/assets/readme-cover.png)

Hermes 飞书流式卡片插件把 Hermes Agent Gateway 的飞书/Lark 回复变成一张持续更新的交互式卡片：思考过程、工具调用、最终答案、授权确认、选项选择和运行统计都能收束在同一张飞书卡片里，而不是被拆成刷屏的灰色原生消息。

它重点解决飞书接入 Hermes 时最常见的痛点：流式内容漏字/乱序、长表格和代码块渲染成 raw markdown、工具调用过程不可见、approval/clarify 需要手工回复、sidecar 故障难排查、多 bot / 多 profile 难运维，以及升级 Hermes 后 hook 兼容不确定。

![飞书流式卡片真实效果截图](docs/assets/feishu-weather-card.png)

## 项目亮点

- **流式卡片体验**：`thinking.delta`、`answer.delta`、`tool.updated`、`message.completed` 聚合到同一张飞书卡片，减少刷屏和上下文断裂。
- **卡片内交互**：Hermes approval / clarify choices 渲染成飞书按钮，用户点击后原任务继续执行，原卡片继续更新。
- **长内容更稳**：长 Markdown 表格和 fenced code block 按结构边界切分，降低飞书 raw markdown 和半截代码围栏问题。
- **多 bot / 多 profile**：支持多飞书机器人、多 Hermes profile、群聊绑定、bot/profile 标题和路由诊断。
- **sidecar-only 架构**：Hermes hook fail-open，飞书发送/更新、状态机、重试、健康检查都在 sidecar 中独立运行。
- **安装和发布友好**：支持一行安装、Release 安装包、`doctor` 诊断、`start/status/stop` 进程管理和安全 restore/uninstall。

## 解决的真实痛点

| 痛点 | 项目能力 |
|------|----------|
| 飞书里只能看到一大段最终文本，看不到 Agent 思考和工具进度 | 思考、答案、工具状态、footer 统计持续更新在同一张卡片 |
| 模型调用工具时内容乱序、漏字、完成后又冒出灰色原生消息 | per-message 顺序、PATCH 合并、终态优先和原生 resend 抑制 |
| Hermes 请求授权或让用户选择选项时，需要手工输入编号 | 飞书卡片内按钮选择，点击后继续原任务 |
| 长表格/长代码块被飞书渲染成 raw markdown | Markdown-aware split，重复表头和完整 code fence |
| 多机器人、多群聊、多 profile 难确认路由 | `bindings.chats`、profile-aware session key、`/health.routing` 诊断 |
| sidecar 或 hook 出问题难定位 | `doctor`、`/health` metrics、fail-closed installer、restore/uninstall |

## V3.5.2 安装补丁

V3.5.2 是 V3.5.1 之后的安装补丁版，面向真实用户安装场景补齐“能下载、能安装、能诊断、能发 Release 包”这些关键体验：

- **一行安装入口**：新增 macOS/Linux `install.sh` 和 Windows `install.ps1`，README 首页直接给出一行安装命令。
- **Release 安装包**：新增 GitHub Actions release assets workflow，tag 发布时自动打包 macOS/Linux/Windows 安装包和 checksum。
- **macOS 安装更稳**：`install.sh` 不再直接 `source .env`，只白名单读取飞书/sidecar 变量；遇到 uv/PEP 668 externally managed Python 时自动重试 `--break-system-packages`。
- **安装材料完整**：新增 [README-install.md](README-install.md) 和 [V3.5.2 release notes](docs/release-notes-v3.5.2.md)，并把 V3.6.0 后续路线沉淀到 [docs/roadmap-v3.6.0.md](docs/roadmap-v3.6.0.md)。

## V3.5.2 解决了什么

| 场景 | V3.5.2 的变化 |
|------|---------------|
| 新用户不知道怎么装 | 首页提供 macOS/Linux 和 Windows 一行安装命令 |
| 用户不想 clone 仓库 | Release 会提供 `macos.tar.gz`、`linux.tar.gz`、`windows.zip` 和 checksum |
| macOS `.env` 里有带空格的 Chrome/浏览器路径 | 安装器白名单解析 `.env`，不会把无关变量当 shell 命令执行 |
| uv 管理的 Python 拒绝 `pip install --user` | 安装器检测 `externally-managed-environment` 后自动重试安全提示明确的安装路径 |
| 后续版本边界不清楚 | V3.6.0 roadmap 明确 repair、媒体/文件消息、多 profile 运维、E2E/发布矩阵 |

## V3.5.x 功能基线

**交互能力**

- 新增飞书按钮交互闭环：`interaction.requested` 渲染按钮，`/card/actions` 记录选择，Hermes hook 轮询 `/interactions/{interaction_id}` 后继续执行。
- 授权/选项按钮采用飞书 JSON 2.0 `button + behaviors.callback`，避免旧式 action 容器在飞书端更新失败。
- approval / clarify hook 透传 Hermes event loop，交互请求和流式 delta 共用同一条 message send lock，减少事件乱序。

**流式稳定性**

- 修复 issue #41：多条回复和新版 Hermes streaming flow 继续走卡片更新，最终回答不再从第二条开始退回原生 text。
- 修复 issue #39：`message.completed.answer` 为空时不再清空已通过 `answer.delta` 展示的内容，避免 DeepSeek V4 Pro 工具调用后卡片最终无内容。
- 修复 issue #31：同一张飞书卡片 PATCH 串行化，避免旧快照后到覆盖新内容。
- 修复 issue #25：Hermes v2026.5.7 的 `event_message_id` 作为显式 `message_id` 使用，保证 `message.started` 和 `message.completed` 落在同一张 fallback 卡片。
- 同一消息的 HTTP 事件发送按 message id 加锁；多线程 Hermes callback 产生的 sequence 不会互相踩踏。
- `answer.delta` / `thinking.delta` 保留原始边界空格，避免中文、英文、代码片段被拼坏。
- `thinking.delta(mode=append_block)` 以完整思考片段追加，修复思考过程句子漏字、截断和粘连。
- sidecar 非终态事件快速 ACK，卡片 PATCH 合并更新，刷新间隔降到 `0.2s`，终态事件优先落卡。
- queued follow-up 完成后抑制原生 resend，避免“卡片已完成但下面又冒出一大段灰色原生消息”。

**长内容与渲染**

- 长 Markdown 表格超过 `MAIN_CONTENT_CHUNK_CHARS` 后重复表头分块，仍保持合法表格。
- fenced code block 超长后拆成多个完整 fenced block，避免飞书显示半截代码围栏。
- 保留 V3.3.0 的飞书 5 表格限制保护，超限时自动截断并提示。

**兼容与安装**

- Hermes 0.13.0+/0.14.0、`v2026.5.16+` 使用 `gateway_run_013_plus`。
- 旧版本 Hermes（`v2026.4.23` 到 `v2026.4.x` / `0.12.x`）继续使用 `legacy_gateway_run`。
- `doctor` 输出 `version_source`、`version`、`hook_strategy`、`compatibility`、anchor/anchors 和 `reason`，便于安装前确认。
- 升级插件后必须重新安装 hook：运行 `install --hermes-dir ... --yes`，让 Hermes 使用匹配当前版本的 hook。
- 处理 PR #42：cron 卡片路由优先使用 `job['deliver']` 和 scheduler 解析出的 Feishu target。
- 多 profile / multi bot 体验补齐：issue #23、per-bot/profile title、cron final cards、attachment summaries + native media delivery、reply card context。

**配置与运行安全**

- `--config` 指向的配置文件同目录存在 `.env` 时，会自动读取 `FEISHU_APP_ID`、`FEISHU_APP_SECRET`、`HERMES_FEISHU_CARD_HOST`、`HERMES_FEISHU_CARD_PORT`。
- 真实进程环境变量优先级高于同目录 `.env`。
- 多 Profile 模式下继续要求每个 profile 显式配置 Feishu 凭据，顶层环境变量不会覆盖 profile 凭据。
- 无凭据时 sidecar 使用 no-op client，只做本地状态，不会向真实飞书发卡；`/health.routing.bot_count` 可用于确认是否加载到真实 bot。

## 一行安装

macOS / Linux：

```bash
curl -fsSL https://raw.githubusercontent.com/baileyh8/hermes-feishu-streaming-card/main/install.sh | bash
```

Windows PowerShell：

```powershell
irm https://raw.githubusercontent.com/baileyh8/hermes-feishu-streaming-card/main/install.ps1 | iex
```

安装脚本会自动安装/升级插件、读取或提示填写飞书凭据、写入 `~/.hermes/.env`，并调用整合安装器：

```bash
python3 -m hermes_feishu_card.cli setup --hermes-dir ~/.hermes/hermes-agent --config ~/.hermes/config.yaml --yes
```

安装完成后可以检查 sidecar 状态：

```bash
python3 -m hermes_feishu_card.cli status --config ~/.hermes/config.yaml
```

常用环境变量：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `HFC_VERSION` | `latest` | 指定安装版本，例如 `v3.5.2` 或 `main` |
| `HERMES_DIR` | `~/.hermes/hermes-agent` | Hermes Agent Gateway 目录 |
| `HFC_CONFIG` | `~/.hermes/config.yaml` | sidecar 配置路径 |
| `HFC_ENV_FILE` | `HFC_CONFIG` 同目录 `.env` | 飞书凭据保存位置 |
| `HFC_SKIP_START` | `0` | 设为 `1` 时只安装 hook，不启动 sidecar |
| `HFC_NO_PROMPT` | `0` | 设为 `1` 时禁止交互式输入，适合自动化安装 |

也可以从 Release 下载 `hermes-feishu-card-<version>-macos.tar.gz`、`hermes-feishu-card-<version>-linux.tar.gz` 或 `hermes-feishu-card-<version>-windows.zip`，解压后运行包内的 `install.sh` / `install.ps1`。完整安装包说明见 [README-install.md](README-install.md)。

## 手动安装

```bash
git clone https://github.com/baileyh8/hermes-feishu-streaming-card.git
cd hermes-feishu-streaming-card
pip install -e ".[test]"

export FEISHU_APP_ID=cli_xxx
export FEISHU_APP_SECRET=xxx

python3 -m hermes_feishu_card.cli setup --hermes-dir ~/.hermes/hermes-agent --yes
```

`setup` 是整合安装器：自动生成配置、检查 Hermes 版本和代码 anchor、安装 hook、启动 sidecar 并做健康检查。它支持 `v2026.4.23` 起的旧版 Hermes，也支持 Hermes 0.13.0+/0.14.0 与 `v2026.5.16+` 新版 anchor。

如果你使用 Hermes 默认目录，也可以把凭据放在 `~/.hermes/.env`：

```bash
FEISHU_APP_ID=cli_xxx
FEISHU_APP_SECRET=xxx
FEISHU_CONNECTION_MODE=websocket
FEISHU_HOME_CHANNEL=oc_xxx
```

之后使用：

```bash
python3 -m hermes_feishu_card.cli start --config ~/.hermes/config.yaml
```

V3.5.2 会自动读取 `~/.hermes/config.yaml` 同目录的 `~/.hermes/.env`。

## 升级

从 V3.2.x/V3.3.0/V3.4.x/V3.5.x 升级到 V3.5.2 向后兼容，**单 Profile 配置无需任何修改**。升级后请重新安装 hook；只重启 sidecar 不足以让 Hermes 使用新的 patch。

```bash
# 1. 停止 sidecar
python3 -m hermes_feishu_card.cli stop --config ~/.hermes_feishu_card/config.yaml

# 2. 更新代码
cd /path/to/hermes-feishu-streaming-card
git checkout v3.5.2
pip install -e ".[test]" --upgrade

# 3. 诊断 Hermes hook strategy 与 anchors
python3 -m hermes_feishu_card.cli doctor \
  --config ~/.hermes_feishu_card/config.yaml \
  --hermes-dir ~/.hermes/hermes-agent

# 4. 重新安装 hook
python3 -m hermes_feishu_card.cli install --hermes-dir ~/.hermes/hermes-agent --yes

# 5. 启动 sidecar
python3 -m hermes_feishu_card.cli start --config ~/.hermes_feishu_card/config.yaml
```

`doctor` 会从 `VERSION` 或 Git tag `v2026.4.23+` 判断 Hermes 支持状态。Hermes 0.13.0+/0.14.0 与 `v2026.5.16+` 应命中 `gateway_run_013_plus`；旧版本 Hermes 应命中 `legacy_gateway_run`。

## 核心功能

- **飞书流式卡片**：`message.started`、`thinking.delta`、`answer.delta`、`tool.updated`、`message.completed`、`message.failed` 汇聚到同一张卡片。
- **授权/选项按钮**：Hermes approval 和 clarify choices 在同一张卡片里显示为按钮，用户点击后继续原任务。
- **多 bot 与群聊绑定**：`bots.items` 注册多个飞书机器人，`bindings.chats` 按 `chat_id` 路由，`group_rules` 预留群聊策略。
- **多 Profile 进程内隔离**：一个 sidecar 服务多个 Hermes profile，使用 `profile_id:message_id` 隔离 session。
- **Profile / Bot 卡片标题**：全局、profile、bot 均可设置标题，bot 级优先。
- **Cron 最终卡片与回复上下文**：cron 任务可发送最终卡片，reply card context 保留必要上下文。
- **附件摘要与原生媒体投递**：attachment summaries + native media delivery，卡片内展示摘要，媒体按飞书原生能力投递。
- **DeepSeek 思维链兼容**：过滤 `<think>`/`</think>` 与 `<thinking>`/`</thinking>` 标签。
- **工具调用跟踪**：累计工具调用次数和每个工具的当前状态。
- **运行统计 footer**：显示耗时、模型、token、上下文占比；非终态卡片显示旋转生成中状态。
- **故障隔离**：sidecar 不可用时 hook fail-open，Hermes 原生文本继续运行。
- **安全安装/恢复**：安装器 fail-closed，`restore`/`uninstall` 检测文件改动后拒绝覆盖。

## 配置

复制 `config.yaml.example` 到本地使用，不要提交真实凭据。

**单 Profile 最小配置**

```yaml
server:
  host: 127.0.0.1
  port: 8765
feishu:
  app_id: ""
  app_secret: ""
card:
  title: Hermes Agent
  footer_fields: [duration, model, input_tokens, output_tokens, context]
```

**单 Profile + 多 Bot / 群聊**

```yaml
server:
  host: 127.0.0.1
  port: 8765
feishu:
  app_id: ""
  app_secret: ""          # fallback
bots:
  default: default
  items:
    sales:
      app_id: "cli_sales_xxx"
      app_secret: "xxx"
    support:
      app_id: "cli_support_yyy"
      app_secret: "yyy"
bindings:
  fallback_bot: default
  chats:
    oc_5cc6a25d8815790fa890dd0226005e83: sales
  group_rules:
    enabled: false
card:
  title: Hermes Agent
  footer_fields: [duration, model, input_tokens, output_tokens, context]
```

**多 Profile**

```yaml
server:
  host: 127.0.0.1
  port: 8765
profiles:
  engineering:
    feishu:
      app_id: "cli_eng_xxx"
      app_secret: "xxx"
    bots:
      default: default
      items:
        default:
          app_id: "cli_eng_xxx"
          app_secret: "xxx"
    bindings:
      fallback_bot: default
      chats: {}
  sales:
    feishu:
      app_id: "cli_sales_xxx"
      app_secret: "xxx"
    bots:
      default: default
      items:
        default:
          app_id: "cli_sales_xxx"
          app_secret: "xxx"
    bindings:
      fallback_bot: default
      chats: {}
card:
  title: Hermes Agent
  footer_fields: [duration, model, input_tokens, output_tokens, context]
```

多 Profile 模式下，`FEISHU_APP_ID` / `FEISHU_APP_SECRET` 不会覆盖 profile 内的 `feishu` 配置。`footer_fields` 支持 `duration`、`model`、`input_tokens`、`output_tokens`、`context`。

## 飞书应用配置

```bash
export FEISHU_APP_ID=cli_xxx
export FEISHU_APP_SECRET=xxx

# 真实飞书 smoke 测试
python3 -m hermes_feishu_card.cli smoke-feishu-card \
  --config config.yaml.example \
  --chat-id oc_xxx
```

如果凭据未配置，sidecar 会使用 no-op client；这适合本地单元测试，但不会向真实飞书发卡。真实联调前请检查：

```bash
python3 -m hermes_feishu_card.cli status --config ~/.hermes/config.yaml
```

确认 `/health.routing.bot_count` 大于 0，且 `last_route_error` 为空。

## Hermes Gateway 流式配置

确保 Hermes `config.yaml` 中启用流式编辑：

```yaml
streaming:
  enabled: true
  transport: edit
```

不要设置 `display.platforms.feishu.streaming: false`。也不要把 `display.show_reasoning` 当成本插件的必需开关；它可能在最终回复中追加 reasoning 代码块，反而干扰卡片流式体验。若模型只返回最终答案、没有 thinking 增量，卡片会直接显示最终答案。

## CLI 命令

| 命令 | 说明 |
|------|------|
| `setup --hermes-dir ... --yes` | 一键安装：配置、检测、hook、sidecar、健康检查 |
| `doctor --config ... --hermes-dir ...` | 诊断 Hermes 版本、`hook_strategy`、`compatibility`、anchors 和原因 |
| `install --hermes-dir ... --yes` | 安装 hook 到 Hermes |
| `restore --hermes-dir ... --yes` | 恢复原始 Hermes 文件 |
| `uninstall --hermes-dir ... --yes` | 卸载并恢复 |
| `start --config ...` | 启动 sidecar |
| `stop --config ...` | 停止 sidecar；校验 PID/token 与 `/health` 的 `process_pid/process_token` 后才停止 |
| `status --config ...` | 查看 sidecar 状态、routing 与 metrics |
| `bots list|show|add|remove --config ...` | 管理飞书 Bot 注册 |
| `bots bind-chat|unbind-chat --config ...` | 管理 `bindings.chats` 群聊绑定 |

## 架构

```text
Hermes Gateway
  └─ minimal hook in gateway/run.py
       └─ hermes_feishu_card.hook_runtime
            └─ HTTP POST /events ——→  sidecar server
                                      ├─ CardSession 状态机
                                      ├─ render_card() 卡片渲染
                                      ├─ Feishu CardKit HTTP client 已实现
                                      ├─ tenant token / send / update
                                      ├─ 节流、合并、重试、锁、诊断
                                      └─ /health 指标
```

Hermes hook 将事件 fail-open 转发给 sidecar。sidecar 持有完整会话状态和飞书边界，可独立测试、重启、诊断。历史实现集中归档在 `legacy/`（`installer_v2.py`、`gateway_run_patch.py`、`patch_feishu.py` 等），不是 active runtime；当前主线以 `hermes_feishu_card/` 为准。迁移说明见 [docs/migration.md](docs/migration.md)。

## 常见问题

- **卡片没有思考/不流式**：检查 `streaming.enabled: true` 与 `streaming.transport: edit`，确认模型确实输出 `thinking.delta`。
- **真实飞书没有卡片**：检查凭据是否进入 sidecar；没有凭据时是 no-op client。V3.5.2 会读取 config 同目录 `.env`，真实环境变量仍优先。
- **卡片停在“思考中”**：看 `/health.diagnostics.last_terminal_event` 和 `feishu_update_failures`，确认终态事件是否到达、飞书 PATCH 是否成功。
- **出现灰色原生文本**：通常说明 sidecar 未成功接收或更新终态；V3.5.x 已补 queued follow-up suppression 和终态优先更新。
- **思考过程漏字/截断**：V3.5.x 已用 ordered send、append_block、PATCH 合并与终态优先修复；若仍出现，先检查 `/health.metrics.feishu_update_failures`。
- **长表格/代码显示成 raw markdown**：V3.5.x 会结构化拆分；如果仍异常，减少单个表格列宽或代码块长度。
- **重复卡片**：检查 `/health` metrics（`events_received`、`events_applied`、`feishu_send_successes`）。多 Profile 下 session key 为 `profile_id:message_id`。
- **Hermes 0.13.0+/0.14.0 升级后无卡片**：先跑 `doctor --config ... --hermes-dir ...`，确认 `hook_strategy` 为 `gateway_run_013_plus`，再重新安装 hook。
- **恢复失败**：`restore`/`uninstall` 检测到文件改动会拒绝覆盖，先备份再人工确认差异。
- **只想验证本地 sidecar**：可以用 no-op client 跑测试；真实飞书 smoke 需要真实 App ID/Secret 和 chat id。

## 版本历史

| 版本 | 日期 | 主要变更 |
|------|------|---------|
| [v3.5.2](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.5.2) | 2026-06 | 跨平台一行安装、Release 安装包、macOS `.env` 安全解析、uv/PEP 668 Python 安装适配、Windows installer CI 解析验证 |
| [v3.5.1](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.5.1) | 2026-06 | 流式更新排序与合并、飞书 JSON 2.0 按钮修复、queued follow-up 原生消息抑制、`.env` 凭据回退、README 首页重整 |
| [v3.5.0](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.5.0) | 2026-06 | 飞书按钮交互闭环、issue #41、PR #42、长表格/代码块结构拆分、thinking 漏字修复 |
| [v3.4.3](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.4.3) | 2026-05 | issue #39、Markdown 结构化切分、Hermes v0.14.0 / v2026.5.16+ 验证 |
| [v3.4.2](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.4.2) | 2026-05 | issue #31，避免并发 PATCH 和 sequence 竞争导致内容回退/漏字 |
| [v3.4.1](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.4.1) | 2026-05 | issue #25，Hermes v2026.5.7 fallback message id 生命周期一致 |
| [v3.4.0](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.4.0) | 2026-05 | Hermes 0.13.0+ 兼容、旧版本 strategy、issue #23、多 profile/multi bot、附件与回复上下文 |
| [v3.3.0](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.3.0) | 2026-05 | 多 Profile、DeepSeek 兼容、表格保护、Footer 动画、平台判断修复 |
| [v3.2.1](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.2.1) | 2026-04 | Accept-Encoding 修复 |
| [v3.2.0](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.2.0) | 2026-04 | 多 Bot 路由、群聊绑定、Bot CLI、路由诊断 |
| [v3.1.0](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.1.0) | 2026-04 | Sidecar 架构、流式卡片、健康端点、安装向导 |
| [v3.0.0](https://github.com/baileyh8/hermes-feishu-streaming-card/releases/tag/v3.0.0) | 2026-04 | Sidecar-only 初始发布 |

完整更新日志：[CHANGELOG.md](CHANGELOG.md)。

## 测试与验收

```bash
python3 -m pytest -q
```

本版本自动化覆盖配置加载、hook patch、Hermes runtime event、sidecar session、飞书按钮 callback、长表格/代码块切分、queued follow-up completion、cron deliver precedence 和文档约束。

已完成的人工/集成验收包括：真实 Feishu E2E 主链路、真实 Hermes Gateway E2E、真实飞书应用卡片验证、16k 长卡压力测试、`doctor -> install -> restore` 闭环、多 Profile 路由、DeepSeek 标签过滤。Feishu CardKit HTTP client 已实现，并通过 mock Feishu server 和真实飞书 smoke 验证。

## 文档

- 架构说明：[中文](docs/architecture.md) / [English](docs/architecture.en.md)
- 事件协议：[中文](docs/event-protocol.md) / [English](docs/event-protocol.en.md)
- 安装安全：[中文](docs/installer-safety.md) / [English](docs/installer-safety.en.md)
- 迁移说明：[中文](docs/migration.md) / [English](docs/migration.en.md)
- 端到端验证：[中文](docs/e2e-verification.md) / [English](docs/e2e-verification.en.md)
- 发布准备：[中文](docs/release-readiness.md) / [English](docs/release-readiness.en.md)
- 测试说明：[中文](docs/testing.md) / [English](docs/testing.en.md)

## 贡献者

感谢以下贡献者对项目的改进：

- [gischuck](https://github.com/gischuck) — [PR #12](https://github.com/baileyh8/hermes-feishu-streaming-card/pull/12) Accept-Encoding 修复（V3.2.1 brotli 兼容）
- [fengs2021](https://github.com/fengs2021) — [PR #17](https://github.com/baileyh8/hermes-feishu-streaming-card/pull/17) 锁架构优化与更新间隔改进（V3.3.0）

## 安全说明

不要把 App Secret、tenant token、真实 chat_id 提交到仓库。效果图仅用于展示卡片效果，生产凭据保存在本机配置或环境变量中。

## License

MIT License，详见 [LICENSE](LICENSE)。
