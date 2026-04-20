# LemonClaw MVP 1.0 Steps 1-5 踩坑复盘

> 沉淀 Steps 1-5（04-16 ~ 04-19）开发过程中的经验教训。
> 状态：**已填写** · 最后更新：2026-04-20

---

## 背景

LemonClaw MVP 是基于 OpenClaw Runtime 的个人 AI 助手桌面应用。采用 Vendor 子进程模式（Electron + OpenClaw Gateway），参考了 RivonClaw、HomiClaw、Hermes、ClawX 四个开源项目。

04-16 ~ 04-19 四天内完成 Step 1（Electron 骨架）到 Step 5（LLM 接通），期间积累了大量与 OpenClaw 协议/配置相关的踩坑经验。

---

## 当时的目标

Phase 1 MVP 核心路径：Electron 桌面壳 → Gateway 集成 → Chat UI → LLM 真实回复 → Settings → 会话持久化。

Step 1-5 的具体目标：

- Step 1: Electron 安全骨架
- Step 2: 前端框架 + shadcn/ui + 深色主题
- Step 2.5: 架构决策（Vendor 子进程模式 v3.0.0）
- Step 3: Gateway 集成层（5 个核心模块）
- Step 4: Chat UI Mock 模式
- Step 5: LLM 接通（Minimax 流式回复）

---

## 实际做到的程度

Steps 1-5 全部完成。用户能在桌面应用中发消息 → Gateway 调 Minimax API → 流式真实回复显示。

遗留未完成：

1. SettingsPage 默认值前后端不一致
2. Agent 切换未调用 sessions.patch RPC
3. launcher.ts 中 probeSidecar() 孤立代码（~90 行）
4. chat-store.ts 约 15 处 console.log 待清理

---

## 主要问题 / 阻塞点

1. **OpenClaw 协议/配置格式无文档** — 最大阻塞源。OpenClaw 的 auth-profiles.json、openclaw.json、RPC 握手协议均无公开文档，靠源码逆向 + 试错。Step 3-4 花费大量时间在"猜格式"上。
2. **Windows 环境兼容性** — ELECTRON_RUN_AS_NODE、taskkill 语法、spawn curl 控制台窗口、SIGUSR1 不可用等问题，每个都是独立排查。
3. **Gateway 启动时序不确定** — stdout 输出"listening"时 RPC 未就绪，需等到"embedded acpx runtime backend ready"才能连接，这个时序差异导致多次"连不上"的假象。

---

## 踩坑记录

| #    | 坑                                                           | 影响                       | 是否已解决                                                   |
| ---- | ------------------------------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| 1    | **ELECTRON_RUN_AS_NODE=1**：Claude Code 自身基于 Electron，自动设置此变量，导致 Electron 以纯 Node 模式运行，所有内置模块 undefined | 开发环境完全无法启动       | ✅ `env -u` 移除                                              |
| 2    | **Config Bridge 配置格式偏差**：auth-profiles.json 格式、models.providers 嵌 apiKey、plugins 用 entries 而非 allow/deny — 全部靠猜 | Gateway 反复重启/启动失败  | ✅ 对照用户可用配置修正                                       |
| 3    | **RPC 握手链**：`missing scope: operator.write` → token 认证问题 → `origin not allowed` → Control UI origin 检查 → `requires device identity` → 需要 dangerouslyDisableDeviceAuth | 连接 Gateway 花了一整天    | ✅ openclaw-control-ui + origin + dangerouslyDisableDeviceAuth |
| 4    | **plugins 引用不存在的插件名**（如 `minimax`）               | Gateway 校验失败反复重启   | ✅ 改用 entries 格式                                          |
| 5    | **Gateway 启动后设 running 太早**：stdout "listening" 时 RPC 未就绪 | 前端连接 RPC 失败          | ✅ 等 RPC 连接成功后才通知                                    |
| 6    | **Windows taskkill 语法**：Git Bash 下 `taskkill //F //PID` 失败 | 无法自动化重启             | ✅ 用 node 间接调用                                           |
| 7    | **Gateway spawn curl**：Windows 上 DEP0190 警告 + 可见控制台窗口 | 用户体验差                 | ⬜ OpenClaw 内部行为，无法完全消除                            |
| 8    | **research/ 文件夹误删**：`rm -rf` 误删未 git 跟踪的参考文档 | 参考文档丢失，需从源码重建 | ⚠️ 已重建但非原始版本                                         |
| 9    | **SettingsPage 默认值不一致**：前端 provider=openai/model=glm-5.1，后端=minimax-portal/MiniMax-M2.7-HighSpeed | 用户看到错误的默认配置     | ⬜ Step 6 修复                                                |
| 10   | **prompt 过长触发 BOOTSTRAP.md**：Gateway 发送引导内容       | 用户看到无关内容           | ⚠️ 属正常行为，需优化 system prompt                           |

---

## 不建议延续的思路

1. **猜 OpenClaw 配置格式** — 不要自己发明配置格式，严格对照用户本地可用的 openclaw.json。有疑问时直接读源码或对照参考项目。
2. **不参考源码直接写集成代码** — Steps 1-2 还好，Step 3 开始必须对照 OpenClaw/HomiClaw/RivonClaw 源码写，否则格式必定出错。
3. **大 Step 不拆分** — Step 3 和 Step 4 都是因为拆成了小步骤才最终跑通。一个"接通 Gateway"的大任务实际包含 5+ 个独立问题。
4. **RivonClaw 方案照搬** — RivonClaw 是个人项目，部分实现不够成熟（如 keychain 在某些平台的表现），需批判性参考。

---

## 可保留的经验

1. **Vendor 子进程模式** — OpenClaw 作为子进程、零源码侵入，解耦清晰。这个核心架构决策被验证可行。
2. **端口隔离** — LemonClaw 3212 vs 用户本地 OpenClaw 18789，避免冲突。
3. **系统 Node v24.x** — 不用 Electron 内置 v20.x，版本够用且稳定。
4. **参考文档先行** — 四个参考项目的源码分析文档（docs/reference/）为后续开发提供了重要支撑。
5. **API Key 自动播种** — 从用户 ~/.openclaw/openclaw.json 读取已配置的 key，零配置启动。
6. **Zustand 状态管理** — 比 MST 轻量，适合 LemonClaw 的单进程架构。
7. **严格对照可用配置** — 对照用户本地可用 openclaw.json 而非文档/猜测。

---

## 对当前版本的启发

1. **每个新功能先验证协议** — 接 OpenClaw 任何新 RPC/配置前，先用 curl 或 wscat 手动验证协议格式，再写代码。
2. **Step 6+ 保持小步快跑** — 每步一个独立可验证的目标，不跨步。
3. **Windows 兼容性要持续关注** — SIGUSR1、spawn 控制台窗口、路径分隔符、taskkill 语法等。
4. **Git 跟踪要及时** — 新建文件立即 git add，避免误删无法恢复。
5. **Fork openclaw 尽早执行** — Vendor 版本锁定很重要，避免上游更新引入不兼容。

---

## 旧源码归档

当前代码即 Step 1-5 的实现，无需额外归档。

---

