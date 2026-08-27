# jiuwenswarm-vibestudio

**VibeStudio** —— 通过自然语言对话构建全栈 Web 应用，由 WorkSwarm 驱动。

描述你想要的。一队协调的 WorkSwarm 智能体负责写代码。浏览器内实时预览立即出现。
无需终端、无需配置、无需任何编程知识。

---

## 它是什么

VibeStudio 是一个 OpenJiuwen 频道——WorkSwarm 智能体运行时的薄客户端。它使用
`@jiuwenswarm/sdk` 作为与 WorkSwarm 网关通信的唯一传输层。

智能体侧运行在服务器上：
- **架构师** 智能体将请求分解为组件
- **前端** 智能体编写 React / TypeScript / Tailwind
- **后端** 智能体在需要时编写 API 路由
- **数据库** 智能体编写 schema 和查询
- **QA** 智能体在输出到达用户前审查每个产出

浏览器侧渲染：
- **聊天面板**，带实时流式 token 输出和快捷操作按钮
- **实时预览** —— 生成的 app 由开发服务器（esbuild）离线打包，因此无需访问 CodeSandbox 即可工作
- **代码抽屉** —— 预览下方并排显示文件浏览器和编辑器（不挤压预览宽度）；点击文件即可打开
- **聊天内联的 Swarm 活动** —— 一个可折叠的、实时的逐智能体活动流，智能体工作时打开、完成后关闭
- **模板选择器**，含 6 个现成模板
- **重连 toast**，WebSocket 恢复时自动消失

---

## 快速开始

```bash
# 0. 启动 WorkSwarm（必须先运行）
jiuwenswarm-start           # 或使用桌面安装程序

# 1. 安装依赖
npm install

# 2. 复制并填写环境变量
cp .env.example .env
# 设置 VITE_JIUWENSWARM_URL=ws://localhost:19000/ws  （已是默认值）

# 3. 启动开发服务器
npm run dev
# 打开 http://localhost:5174
```

完整的前置条件、远程服务器设置、生产构建和故障排查请参见
**[`VibeStudioInstall.md`](VibeStudioInstall.md)**。

---

## 架构

```
浏览器（VibeStudio SPA）
├── @jiuwenswarm/sdk          ← 与 WorkSwarm 通信的唯一传输层
│   └── RpcClient             ← WebSocket、会话、streamEvents、回退
├── Zustand stores            ← 项目文件、聊天消息、智能体日志、会话列表
├── Sandpack                  ← 浏览器内 React/TS 预览 + 可选代码编辑器
└── React Router              ← /（仪表盘）和 /studio/:sessionId

WorkSwarm 服务器
└── 智能体团队（架构师 + 前端 + 后端 + 数据库 + QA）
    └── openjiuwen.core 运行时（记忆、检索、工具执行）
```

会话即项目。生成的文件持久化到 `localStorage`，因此关闭并重新打开项目会恢复上次状态，
无需重新生成。

---

## 项目结构

```
jiuwenswarm-vibestudio/
├── src/
│   ├── config.ts                       # 环境变量访问
│   ├── main.tsx                        # React 根
│   ├── App.tsx                         # 路由
│   ├── index.css                       # Tailwind + CSS 变量主题（浅色/深色）
│   │
│   ├── lib/
│   │   ├── client.ts                   # RpcClient —— WebSocket、会话、streamEvents、回退
│   │   ├── streamParser.ts             # @@FILE…@@END_FILE 提取
│   │   ├── agentMode.ts                # 模式选择（团队/智能体）+ 意图检测
│   │   └── export.ts                   # ZIP 导出 + Markdown 聊天导出（fflate）
│   │
│   ├── store/
│   │   ├── project.ts                  # 文件、聊天消息、智能体日志、回退栈
│   │   ├── session.ts                  # 项目列表 + 文件快照（localStorage）
│   │   ├── layout.ts                   # 聊天宽度 + 代码抽屉高度（localStorage）
│   │   └── theme.ts                    # 深色/浅色偏好（localStorage）
│   │
│   ├── components/
│   │   ├── Resizer.tsx                 # 可调整大小面板的拖拽手柄
│   │   ├── Chat/
│   │   │   ├── ChatPanel.tsx           # 提示输入 + 流式消息 + 快捷操作
│   │   │   └── MessageBubble.tsx       # 用户 / 助手 / 状态气泡
│   │   ├── Preview/
│   │   │   └── SandpackPreview.tsx     # 实时预览 + 下方可调整大小的代码抽屉
│   │   ├── FileExplorer/
│   │   │   └── FileTree.tsx            # 嵌套文件树（在编辑器中打开文件）
│   │   ├── Swarm/
│   │   │   └── SwarmActivity.tsx       # 内联可折叠的智能体活动流（在聊天中）
│   │   ├── TemplateModal.tsx           # 模板选择器浮层（6 个模板）
│   │   ├── ReconnectToast.tsx          # 浮动断连横幅
│   │   └── ErrorBoundary.tsx           # 渲染错误恢复界面
│   │
│   └── pages/
│       ├── Dashboard.tsx               # 项目列表、创建、重命名、删除、模板
│       └── Studio.tsx                  # 主工作区（聊天 + 预览 + 抽屉，移动端标签页）
│
├── tests/
│   ├── setup.ts                        # Vitest + jest-dom 引导
│   ├── streamParser.test.ts            # @@FILE 提取、@@DELETE、空白
│   ├── agentMode.test.ts               # 意图检测、模式选择、流式选项
│   ├── projectStore.test.ts            # 聊天消息、智能体日志、回退、重置
│   ├── export.test.ts                  # Markdown 导出、文件名清洗、下载触发
│   └── session.test.ts                 # 会话增删改查、文件持久化、活动项目
│
├── docs/
│   ├── en/vibestudio/VibeStudio.md        # 概览（本文档）
│   ├── en/vibestudio/VibeStudioGuide.md   # 用户指南（模板、面板、移动端、导出）
│   └── en/vibestudio/VibeStudioInstall.md # 安装指南
│
└── examples/
    └── prompts/
        ├── counter-app.md              # 最小计数器——最简单的应用
        ├── todo-app.md                 # 带优先级、标签、深色模式的待办应用
        ├── landing-page.md             # SaaS 落地页
        ├── dashboard.md                # 带图表的分析仪表盘
        ├── kanban-board.md             # 拖拽看板
        ├── chat-ui.md                  # 精致的聊天界面
        └── auth-form.md                # 带校验的登录/注册表单
```

---

## 智能体输出如何工作

WorkSwarm 智能体使用哨兵标记将生成的文件嵌入 token 流：

```
@@FILE: src/components/Button.tsx
```tsx
export function Button({ label }: { label: string }) {
  return <button>{label}</button>;
}
```
@@END_FILE
```

`src/lib/streamParser.ts` 在流完成时提取这些块，并将其作为文件增量应用到项目 store。
删除使用 `@@DELETE: path`。

系统提示前缀（`buildAgentSystemPrefix()`）指示智能体遵循此约定。

---

## 智能体模式选择

| 意图 | 模式 | 何时 |
|---|---|---|
| `generate` | `"team"` | 构建新应用或主要功能 |
| `refine` | `"agent"` | 对已有文件的定向修改 |
| `explain` | `"agent"` | 关于代码库的问题 |
| `fix` | `"agent"` | 修复 bug 或纠正错误 |

`inferIntent()` 对用户消息文本应用简单启发式。用户也可以通过快捷操作按钮
（**生成 / 修复 / 解释 / 重构**）覆盖。

---

## 开发

```bash
npm run dev        # 在 :5174 启动 Vite 开发服务器
npm test           # 运行 Vitest 单元测试（5 个套件共 42 个测试）
npm run coverage   # 使用 V8 覆盖率报告运行测试
npm run typecheck  # TypeScript 检查（不产出）
npm run build      # 生产构建到 dist/
```
