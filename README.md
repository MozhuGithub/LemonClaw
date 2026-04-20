# LemonClaw

LemonClaw 是一个基于 OpenClaw 的桌面 AI Agent 工作台。

## 当前阶段
当前项目处于 MVP 阶段，目标是先跑通最小可用闭环，而不是一开始做成完整平台。

当前重点闭环包括：

1. 打开桌面应用
2. 初始化或连接 OpenClaw Gateway
3. 配置模型 Provider / API Key
4. 创建会话并选择 Agent
5. 发起一个真实任务并得到结果
6. 保存并重新打开会话
7. 生成一次任务复盘或技能候选记录

## 技术方向
- Electron
- React
- TypeScript
- electron-vite
- Zustand

## 目录说明
- `src/`：应用源码
- `docs/`：产品、调研、架构、进度、日志等文档
- `references/`：外部参考项目和源码资料
- `data/`：默认 Agent 预设与技能模板
- `resources/`：应用静态资源
- `tests/`：测试代码

## 说明
- LemonClaw 复用 OpenClaw 作为底层能力基础
- MVP 阶段不重写 OpenClaw Gateway 或 Runtime
- 多渠道能力作为后续扩展方向保留，但不是当前主线
