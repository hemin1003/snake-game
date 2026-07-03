# 用 OpenClaw + Claude Code + GitHub + Vercel 实现 Agent Coding

> 从飞书发一条消息，到贪吃蛇游戏上线——全程 Agent 自动化

**一句话概括：** 在飞书里用自然语言告诉 AI 助手「帮我开发一个贪吃蛇游戏，推送到 GitHub，部署到 Vercel」，全程无需打开 IDE、无需手动敲一行代码。

---

## 🏗 架构总览

```mermaid
flowchart TB
    U["👤 贺佬湿<br/>飞书用户"] -->|"「帮我开发贪吃蛇<br/>推GitHub部署Vercel」"| FS["💬 飞书"]
    FS -->|WebSocket| OC["📖 子书 (OpenClaw)<br/>本地 Mac 上的 AI 助手"]
    
    subgraph Mac["🖥 本地 Mac"]
        OC -->|"cd myclaudecode<br/>启动 claude"| CC["✳ Claude Code<br/>AI 编程 Agent"]
        CC -->|"读写文件<br/>执行命令"| FS2["📁 本地文件系统"]
        FS2 -->|"snake.html<br/>index.html"| CC
        OC -->|"直接执行"| SHELL["⚡ Shell"]
    end
    
    CC -->|"git push"| GH["🐙 GitHub<br/>hemin1003/snake-game"]
    OC -->|"部署"| VC["▲ Vercel<br/>myclaudecode.vercel.app"]
    GH -->|"GitHub Pages"| WEB["🌐 heminit.com/snake-game"]
    
    VC --> RESULT["🎮 在线可玩<br/>贪吃蛇游戏"]
    WEB --> RESULT
```

---

## 🔄 完整流程拆解

### 1. 飞书 → OpenClaw 接收指令

贺佬湿在飞书里直接发消息：`帮我开发个贪吃蛇游戏，推送到 GitHub，部署到 Vercel`。

飞书通过 WebSocket 连接把消息路由给本地 Mac 上的 OpenClaw 助手 **子书**，全程无需任何中间服务器。

> 💡 **关键点：** OpenClaw 自带飞书通道插件，配好 App ID/Secret 就能打通。消息通过 WebSocket 实时推送到本地 Gateway，延迟极低。

### 2. OpenClaw 理解意图 & 准备环境

子书（OpenClaw）解析需求，确定执行计划：

1. 创建 `myclaudecode` 工作目录
2. 启动 Claude Code 进行编码
3. 代码完成后推到 GitHub
4. 部署到 Vercel

```bash
# 子书执行的第一步
mkdir -p ~/.openclaw/workspace/myclaudecode
```

### 3. Claude Code 生成代码

子书通过终端 PTY 启动 Claude Code，把编程需求发过去：

```bash
cd ~/.openclaw/workspace/myclaudecode && claude
```

Claude Code 接到指令后：

- 分析需求（纯 HTML/CSS/JS 单文件、Canvas 渲染、键盘+触屏操控）
- Synthesizing（思考生成）约 1 分钟
- 直接写文件 `snake.html`（475 行）
- 等待确认后写入磁盘

> 💡 **关键点：** Claude Code 有自主文件读写权限，不再需要「复制粘贴代码」。它直接在当前目录创建、编辑文件，**所见即所得**。

### 4. Git 提交 & 推送 GitHub

Claude Code 写好代码后尝试推送 GitHub，但发现 `gh` CLI 未安装。子书接管后续：

```bash
# 安装 GitHub CLI
brew install gh

# 使用 token 认证
echo "ghp_xxx" | gh auth login --with-token

# 创建仓库 & 推送
gh repo create hemin1003/snake-game --public --push

# 添加 index.html 启用 GitHub Pages
cp myclaudecode/snake.html index.html
git add index.html && git commit && git push
```

> 💡 **关键点：** 当 Claude Code 遇到环境问题（缺 `gh` CLI），子书自动接手解决环境依赖，再继续推进。两个 Agent 配合互补。

### 5. GitHub Pages 自动上线

推送到 GitHub 后，子书通过 API 检查 Pages 状态，发现已自动构建：

- GitHub Pages 自动检测到仓库推送
- 构建静态站点（源自定义域名 `heminit.com`）
- 上线地址：[heminit.com/snake-game](http://heminit.com/snake-game/)

### 6. Vercel 部署

用户觉得 GitHub Pages 不够快，要求部署到 Vercel。子书直接：

```bash
# 使用 Vercel Token 一键部署
npx vercel deploy --token=vcp_xxx --prod --yes

# 输出
Production: myclaudecode.vercel.app  ✅
Ready in 7s
```

- 零配置，自动检测静态站点
- 添加 `index.html` 后重新部署
- 7 秒完成构建 + CDN 分发

---

## 🎯 最终成果

| 项目 | 地址 |
|------|------|
| **GitHub 仓库** | [github.com/hemin1003/snake-game](https://github.com/hemin1003/snake-game) |
| **GitHub Pages** | [heminit.com/snake-game](http://heminit.com/snake-game/) |
| **Vercel** | [myclaudecode.vercel.app](https://myclaudecode.vercel.app) |
| **耗时** | 从发消息到上线 ≈ 5 分钟 |

---

## 🧩 核心角色分工

```mermaid
graph LR
    OC["📖 子书 (OpenClaw)<br/>────────<br/>• 消息接收 & 路由<br/>• 环境准备 & 工具安装<br/>• 流程编排 & 错误处理<br/>• 多平台部署"]
    CC["✳ Claude Code<br/>────────<br/>• 代码生成<br/>• 文件创建与编辑<br/>• Git 操作<br/>• 终端命令执行"]
    GH["🐙 GitHub<br/>────────<br/>• 代码托管<br/>• Pages 静态部署<br/>• 版本管理"]
    VC["▲ Vercel<br/>────────<br/>• 极速构建<br/>• CDN 全球分发<br/>• 自动 HTTPS"]
    
    OC -->|"启动 & 传参"| CC
    OC -->|"直接推送"| GH
    OC -->|"直接部署"| VC
    CC -->|"生成代码到"| OC
    GH -->|"Pages 服务"| WEB["🌐 在线访问"]
    VC -->|"CDN 服务"| WEB
```

---

## 📋 时序图

```mermaid
sequenceDiagram
    actor 用户 as 👤 贺佬湿
    participant 飞书 as 💬 飞书
    participant 子书 as 📖 子书(OpenClaw)
    participant CC as ✳ Claude Code
    participant GitHub as 🐙 GitHub
    participant Vercel as ▲ Vercel

    用户->>飞书: 「帮我开发贪吃蛇游戏<br/>推到GitHub，部署到Vercel」
    飞书->>子书: WebSocket 推送消息
    
    子书->>子书: 解析意图 & 制定计划
    子书->>子书: mkdir myclaudecode
    
    子书->>CC: 启动 Claude Code (PTY)
    子书->>CC: 发送编程需求
    
    CC->>CC: Synthesizing… (1min)
    CC->>子书: snake.html 生成完毕 (475行)
    子书->>CC: 确认写入文件 ✅
    
    CC->>CC: git add & git commit
    Note over 子书,CC: gh CLI 未安装！
    
    子书->>子书: brew install gh
    子书->>GitHub: gh auth login (token)
    子书->>GitHub: 创建仓库 snake-game
    子书->>GitHub: git push origin main ✅
    
    子书->>子书: cp snake.html → index.html
    子书->>GitHub: push index.html (Pages)
    GitHub-->>用户: heminit.com/snake-game 🎉
    
    用户->>子书: 「部署到 Vercel」
    子书->>子书: npx vercel deploy --prod
    
    Vercel-->>子书: 构建完成 (7s)
    Vercel-->>用户: myclaudecode.vercel.app 🎉
    
    Note over 用户,Vercel: ⏱ 全程约 5 分钟<br/>💻 零手动编码
```

---

## 🔑 关键技术点

### 1. OpenClaw 的多通道消息路由

OpenClaw 通过插件体系支持飞书、微信、钉钉、Discord 等多个通道。每个通道的消息通过 Gateway 统一路由给本地 Agent，Agent 回复自动返回对应通道。

```json
// openclaw.json 中的飞书通道配置
"feishu": {
  "enabled": true,
  "appId": "cli_xxx",
  "connectionMode": "websocket"
}
```

### 2. OpenClaw 启动外部 Agent (Claude Code)

OpenClaw 通过 PTY（伪终端）启动 Claude Code，可以发送指令、读取输出、确认交互。即使 Claude Code 遇到权限确认，OpenClaw 也能代为处理。

```
exec("cd myclaudecode && claude", { pty: true })
process.sendKeys(["enter"])  // 确认文件写入
process.paste("编程需求...")
process.sendKeys(["enter"])  // 发送
```

### 3. 双 Agent 协作模式

当 Claude Code 发现环境缺少工具时（如 `gh` CLI），子书（OpenClaw）主动接管环境准备工作，而不是让 Claude Code 卡住。

### 4. Token 驱动的 API 操作

GitHub 和 Vercel 的操作都通过 Token 认证，无需手动在浏览器操作。用户只需提供一次 Token，Agent 负责后续所有调用。

---

## 🚀 总结

> **这个流程的本质：**
>
> 将「人在 IDE 里写代码 → git 推送 → 手动部署」的传统开发流程，变成「**在聊天框里说一句话 → Agent 自动完成全部**」。

**核心能力栈：**

- **OpenClaw** — 消息路由 + 环境编排
- **Claude Code** — 代码生成 + 文件操作
- **GitHub** — 版本管理 + Pages 部署
- **Vercel** — 极速构建 + CDN 分发

你不需要打开终端、不需要写一行代码、不需要手动 git commit。你只需要在飞书里说一句话，剩下的——创建目录、安装工具、写代码、提交推送、部署上线——全是 Agent 的事。

---

<p align="center">
  <sub>📅 2026-07-03 · 子书 (OpenClaw) + Claude Code · 🎮 <a href="https://myclaudecode.vercel.app">试玩贪吃蛇</a></sub>
</p>
