# 更新日志 / Changelog

版本遵循 [语义化版本](https://semver.org/lang/zh-CN/)。`0.3.x` 一批（2026-09-08）集中解决 **DSH 0.1.2+ 兼容性**与**提醒可靠性**；`0.3.15`（2026-10-06）把插件抬到 **DSH 0.2.0-rc.2**。

## [0.3.15]
### 修复（DSH 0.2.0-rc.2 适配，破坏性变更）
- **启动注入 `timer` 会让整个 `dsh web` 起不来**：0.1.7 起客户端 `client/runtime` 被拆掉后，浏览器 Cordis 树里**没有任何行提供 `timer`**（`cordis-plugin-timer` 只剩 host 行）。插件注入它会永久 `pending`，而启动审计（`boot-client.ts` 的 `assertEntriesActive`）把 pending 条目判为失败并抛错 → **整个前端白屏**。现改为**不注入任何服务**，提示条/看门狗/重复提醒/停滞检测改用 `ctx.effect(() => setTimeout/setInterval)` 自持原生定时器（随插件 fiber 释放）。
- **审批/提问提醒全哑**：`ctx.uiSession.pendingInteractions`（0.1.2 起的公开 observable）在 rc2 已变成私有实现，公开面是 **`sessionStatus`**（`HostObservable<Map<SessionId, { running, pendingInteraction, completionUnread }>>`）。现改读/订阅 `sessionStatus`，详情节字段（approval 的 `toolName`/`reason`、question 与 plan-review 的 `questions[]`）不变。
- **「仅当前会话」范围会完全不响**：rc2 的 `SessionListState` 已无 `current`；该事实改为会话行的 `retainedBy.mainView` 计数（主区域持有引用者即“当前会话”）。
- `running` 改以 **`sessionStatus.running` 为准**（list 行的 `running` 在 rc2 只是 Host 列表成员的展示回退），避免为不在 Host 列表里的会话误报完成。
- 「朗读输出」取最后一条助手回复时，**优先走 `ChatSnapshot.order` + `nodes.get(key)`** 这一 rc2 主路径，`legacy.nodes` 退为兜底。
- 清单：`dsh.client.inject` 里已消失的 `@deepseek-ai/dsh-client-runtime`（及不再使用的 `dsh-client-connection`）替换为**实际消费其服务**的四个包；补 `dsh.manifestVersion: 1` 与 `engines.dsh: ">=0.2.0-rc.2"`。

## [0.3.14]
### 发布链（无插件行为变更）
- 发布改用 **GitHub Actions + OIDC trusted publishing + staged publishing**：推 `v*` tag → CI 只把版本提交到 npm 暂存区 → 由维护者用 2FA 批准后才公开（包内代码与 0.3.13 一致）。
- `publishConfig.registry` 固定为 `registry.npmjs.org`（本机默认 registry 是淘宝镜像）。

## [0.3.13]
### 修复
- **提示音只有尾音**（老问题）：根因是**输出设备的启动延迟**——提醒间隔久，音频设备处于省电/空闲态，唤醒需 ~150ms，而首音只有 0.18s，于是开头被吞。修法：每次播提示音前排 **200ms 静音预热**再排真音符。

## [0.3.12]
### 文档
- 补 0.3.11 的 CHANGELOG 条目、统一日期标注、修正 `ROADMAP.md`「现状」段（五类、恢复默认、自定义音色键）。

## [0.3.11]
### 文档
- README（中/英）与代码对齐：五类提醒、**要求 DSH ≥ 0.1.2**、恢复默认设置、隐私/致谢的字段表述、通知类型的触发条件。
- 新增本 `CHANGELOG.md`；`ROADMAP.md` 的 v0.3 兼容性小节补齐 v0.3.3–0.3.10 脉络。

## [0.3.10]
### 修复
- 语音兜底**防重入**：看门狗触发时的 `speechSynthesis.cancel()` 会再次触发 `onerror`，不加标志会让兜底提示音播两遍。

## [0.3.9]
### 修复
- **完成提醒不再静音**：浏览器 `speechSynthesis` 会静默丢弃 utterance（`speak()` 已调用但 `onstart` 永不触发，既无声也无报错）。现加 **1 秒看门狗 + 各类提示音兜底**（审批→警醒 / 提问→轻点 / 完成→叮咚 / 错误·卡住→低沉）+ 启动时 `getVoices()` 预热。

## [0.3.8]
### 文档 / 体验
- 修正过时文案（底部提示原写“会用中文朗读”，实际随界面语言）与文件头注释（四类 → 五类）。
- 「恢复默认设置」加**二次确认**，避免误点清空。

## [0.3.7]
### 修复
- 修 v0.3.6 引入的**回归**：重建挂起基线时误重置 `running`，会吞掉当轮「完成/失败」（新增 `reseedPending()`，只刷新 pending、不碰 running）。

## [0.3.6]
### 修复（复查）
- 提示音队列的“释放”改为按**实际排程**计时，避免 context 晚启动时与下一条重叠。
- 挂起基线在 `uiSession` 就绪后重建，避免刷新页面时对既有挂起误报。
- 槽位注册改用 `ctx.inject(["slots"], …)`（消除同一类“服务晚到则永不注册”的隐患）。

## [0.3.5]
### 文档
- 「发生错误」加“**尚未经作者实测**”免责说明。
- 说明**只有打开「停滞检测」，第 5 类「卡住」提醒才会触发**。

## [0.3.4]
### 新增 / 文档
- 新增 **「恢复默认设置」** 按钮。
- 重写「朗读输出」说明，点明它**只对「语音」音色生效**。

## [0.3.3]
### 修复
- 等 `AudioContext.resume()` 完成再排程 + 加大启动余量（治“只播尾音”）。
- 语音播报前不再无条件 `cancel()`（会吞掉新句开头）。
- 新增**播放队列**：同一时刻的多条提醒串行播放，不再互相打断。

## [0.3.2]
### 修复
- **完成/失败完全不响**：会话列表订阅用 `ctx.get("sessions")` + 早退，服务未就绪时**订阅从未建立**；改用 `ctx.inject(["sessions"], …)`。

## [0.3.1]
### 修复
- 恢复“朗读最后回复”：DSH 0.1.2 移除了快照里的 `chat`，改从会话视图 `uiConversation.binding(id).target('chat')` 取最后一条助手文本。

## [0.3.0]
### 修复（DSH 0.1.2 兼容，破坏性变更）
- `SessionSummary.pendingInteraction` 与 `SessionSnapshot.pending` / `chat` 已被核心移除；审批/提问改读 **`ctx.uiSession.pendingInteractions`**，失败检测改用 `lastAgentError`。

## [0.2.1]
### 修复
- 包名改为 scoped（`@machine-126/dsh-alert-sound`）后，客户端 bundle 注册 id 未同步 → 修正 “loaded without registering” 报错。
- 规范化 `repository.url`。

## [0.2.0]
### 变更
- 完成 A–I 全部功能：i18n（zh/en）、语音朗读、重复提醒、浏览器系统通知、自定义音色上传、停滞检测（实验性）、勿扰时段、语音语速、悬浮提示开关。
- 包名改为 scoped 并发布到 npm。

## [0.1.0]
- 首个版本：四类提醒（审批 / 提问 / 完成 / 错误）+ 合成音色 + 设置页。
