# 安装指南 —— VibeStudio

---

## 环境要求

| 依赖 | 版本 | 说明 |
|---|---|---|
| Node.js | 18+ | 推荐 LTS；用 `node -v` 验证 |
| npm | 9+ | 随 Node.js 附赠；用 `npm -v` 验证 |
| WorkSwarm 服务器 | 任意当前版本 | 必须正在运行，且可通过 WebSocket 访问 |
| 现代浏览器 | Chrome 114+、Firefox 120+、Safari 17+ | 用于 Sandpack 浏览器内预览 |

---

## 第 1 步 —— 启动 WorkSwarm

VibeStudio 没有自己的后端。它连接到一个正在运行的 WorkSwarm 网关。请在打开 VibeStudio 前
启动它。

**选项 A —— 桌面安装程序（Windows / macOS / HarmonyOS）**

从 [openjiuwen.com](https://openjiuwen.com/en/jiuwenswarm) 下载并运行安装程序。
网关会在端口 19000 自动启动。

**选项 B —— pip**

```bash
pip install jiuwenswarm
jiuwenswarm-init          # 仅首次设置
jiuwenswarm-start
```

**选项 C —— 从源码**

```bash
cd jiuwenswarm            # openjiuwen/jiuwenswarm 仓库
uv venv && uv pip install -e .
jiuwenswarm-start
```

完成上述任一操作后，WebSocket 网关位于：

```
ws://localhost:19000/v1/ws
```

验证它已启动：

```bash
curl http://localhost:19000/v1/health
# 预期：{"status":"ok","protocol_version":"1",...}
```

---

## 第 2 步 —— 获取 VibeStudio 源码

如果你已有 `openjiuwenchannels` 仓库：

```bash
cd /path/to/openjiuwenchannels/jiuwenswarm-vibestudio
```

如果没有，则克隆：

```bash
git clone <repo-url> openjiuwenchannels
cd openjiuwenchannels/jiuwenswarm-vibestudio
```

---

## 第 3 步 —— 安装依赖

```bash
npm install
```

这也会从本地兄弟包 `../jiuwenswarm-sdk/packages/sdk` 安装 `@jiuwenswarm/sdk`。如果该路径
不存在，请改为从 npm 安装 SDK：

```bash
# 备选：从 npm 安装已发布的 SDK
npm install @jiuwenswarm/sdk
```

然后从 `package.json` 中移除 `file:` 引用，替换为已发布版本：

```json
"@jiuwenswarm/sdk": "^1.0.0"
```

---

## 第 4 步 —— 配置环境变量

```bash
cp .env.example .env
```

打开 `.env` 并至少设置：

```dotenv
# WorkSwarm 网关的 WebSocket URL
VITE_JIUWENSWARM_URL=ws://localhost:19000/v1/ws

# 可选的 bearer token——如果服务器无需鉴权则留空
VITE_AUTH_TOKEN=
```

**远程服务器：** 如果 WorkSwarm 运行在另一台机器上或位于 TLS 终止器之后，请相应更改 URL：

```dotenv
VITE_JIUWENSWARM_URL=wss://your-server.example.com:19000/v1/ws
VITE_AUTH_TOKEN=your-token-here
```

> 必须使用 `VITE_` 前缀——Vite 只向浏览器包暴露带此前缀的变量。请勿在此存储生产环境机密；
> 这些值会被嵌入编译后的 JavaScript。

---

## 第 5 步 —— 启动开发服务器

```bash
npm run dev
```

输出：

```
  VITE v5.x.x  ready in 300ms

  ➜  Local:   http://localhost:5174/
  ➜  Network: http://192.168.x.x:5174/
```

在浏览器中打开 **http://localhost:5174**。你应该看到 VibeStudio 仪表盘。

---

## 验证连接

1. 在浏览器中打开 VibeStudio。
2. 点击**创建**（或选择已有项目）打开 Studio。
3. 聊天面板顶部栏显示绿点和**「已连接到 WorkSwarm」**。
4. 如果点是红色（**「已断开——正在重连…」**）：
   - 确认 WorkSwarm 服务器正在运行：`curl http://localhost:19000/v1/health`
   - 确认 `.env` 中的 `VITE_JIUWENSWARM_URL` 与实际服务器地址一致。
   - 编辑 `.env` 后重启 Vite 开发服务器（`Ctrl+C` 然后 `npm run dev`）。

---

## 开发命令

| 命令 | 作用 |
|---|---|
| `npm run dev` | 在 `:5174` 启动带 HMR 的 Vite 开发服务器 |
| `npm test` | 运行 Vitest 单元测试（22 个测试） |
| `npm run typecheck` | 对 `src/` 进行 TypeScript 类型检查——无输出表示无错误 |
| `npm run build` | 编译 TypeScript 并打包到 `dist/` |
| `npm run preview` | 本地提供生产构建以供最终检查 |

---

## 生产构建

```bash
npm run build
```

输出位于 `dist/`。结果是一个纯静态站点——无服务端渲染，运行时无 Node.js。可从任意静态托管
服务提供：

**本地提供以进行测试：**

```bash
npm run preview
# 打开 http://localhost:4173
```

**部署到静态托管：**

```bash
# Vercel
npx vercel dist

# Netlify CLI
netlify deploy --dir dist --prod

# 任意 HTTP 服务器
npx serve dist
```

唯一的运行时依赖是一个可达的 WorkSwarm 网关。必须在运行 `npm run build` **之前**将
`VITE_JIUWENSWARM_URL` 设置为生产网关 URL，因为 Vite 会在构建时内联环境变量。

生产构建请使用显式环境变量：

```bash
VITE_JIUWENSWARM_URL=wss://gateway.prod.example.com:19000/v1/ws \
VITE_AUTH_TOKEN=prod-token \
npm run build
```

---

## 更新

拉取新源码后：

```bash
npm install       # 获取新的或更新的依赖
npm run dev       # 重启开发服务器
```

---

## 故障排查

**「无法创建项目：未连接。请先调用 connect()。」**
: SDK 在会话创建前无法打开 WebSocket。首次创建项目时会自动尝试连接。请检查
  `VITE_JIUWENSWARM_URL` 是否正确，以及网关是否正在运行。

**WebSocket 连接立即断开**
: 如果 `VITE_JIUWENSWARM_URL` 以 `ws://` 开头，但页面通过 `https://` 提供，浏览器会阻止
  混合内容请求。生产 HTTPS 部署请使用 `wss://` 和 TLS 终止网关。

**预览显示空白 iframe**
: Sandpack 需要从 `https://sandpack-cdn-v2.codesandbox.io` 获取运行时。请确保你的网络允许
  该域名。在受限环境中，可通过 `bundlerURL` prop 自定义 Sandpack 打包器 URL（第 2 阶段）。

**`npm install` 在 `@jiuwenswarm/sdk` 上失败**
: 本地 `file:` 引用要求 `jiuwenswarm-sdk/packages/sdk` 作为兄弟目录存在。请先在
  `jiuwenswarm-sdk/packages/sdk` 内运行 `npm install`，或按第 3 步所述切换到已发布的 npm
  包。

**端口 5174 已被占用**
: 更改 `vite.config.ts` 中的端口（`server.port`），或结束占用该端口的进程：
  `lsof -ti:5174 | xargs kill`。

**`tests/` 中的 `tsc` 类型错误**
: `tsconfig.app.json` 只覆盖 `src/`。测试文件使用 Vitest 全局变量（通过 `vite.config.ts`
  中的 `/// <reference types="vitest" />` 声明），不由 `npm run typecheck` 检查。请运行
  `npm test` 来执行它们。
