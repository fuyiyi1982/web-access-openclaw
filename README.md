# 🌐 web-access-openclaw

给 OpenClaw 装上完整的 **CDP 浏览器自动化**能力。

OpenClaw 内置的 browser 工具适用于常规网页操作，但在面对反爬站点（小红书、微信公众号等）、需要用户登录态、或需要精细 JS 控制的场景时力不从心。这个 skill 通过 CDP Proxy 直连用户日常 Chrome，补上了：

- **联网策略**：Brave Search + Jina + curl + CDP，按场景自主判断
- **CDP 浏览器操作**：直连用户 Chrome，天然携带登录态，支持动态页面、交互操作、视频截帧
- **三种点击方式**：`/click`（JS click）、`/clickAt`（CDP 真实鼠标事件）、`/setFiles`（文件上传）
- **站点经验积累**：按域名存储操作经验，跨 session 复用

> 基于 [一泽 Eze](https://github.com/eze-is) 的 [web-access](https://github.com/eze-is/web-access) 项目改编，适配 OpenClaw 平台。

---

## 安装

### 方式一：让 OpenClaw 自己安装（推荐）

直接在聊天中告诉 OpenClaw：

```
帮我安装这个 skill：https://github.com/fuyiyi1982/web-access-openclaw
```

OpenClaw 会自动将仓库克隆到 workspace 的 skills 目录下。

### 方式二：手动安装

```bash
git clone https://github.com/fuyiyi1982/web-access-openclaw ~/.openclaw/workspace/skills/web-access
```

安装后重启 session（发送 `/new`）或重启 Gateway 即可生效：

```bash
openclaw gateway restart
```

验证是否加载成功：

```bash
openclaw skills list
```

应该能看到 `web-access` 出现在列表中。

---

## 前置配置

CDP 模式需要 **Node.js 22+** 和 Chrome 开启远程调试：

1. Chrome 地址栏打开 `chrome://inspect/#remote-debugging`
2. 勾选 **Allow remote debugging for this browser instance**（可能需要重启浏览器）

---

## 使用

安装后直接让 OpenClaw 执行联网任务，skill 自动接管：

- "帮我读一下这个页面：[URL]"
- "去小红书搜索 xxx 的账号"
- "帮我在创作者平台发一篇图文"
- "同时调研这 5 个产品的官网，给我对比摘要"

Skill 会根据任务场景自主选择最合适的工具：
- 搜索 → Brave Search（需已安装 Brave Search 插件）
- 快速读取网页 → Jina（Markdown 化，省 token）
- 原始 HTML → curl
- 反爬站点 / 登录态 / 交互操作 → CDP 浏览器

---

## CDP Proxy API

Proxy 通过 WebSocket 直连 Chrome（兼容 `chrome://inspect` 方式），提供 HTTP API：

```bash
# 启动（自动检测环境并启动 Proxy）
bash ~/.openclaw/workspace/skills/web-access/scripts/check-deps.sh

# 页面操作
curl -s "http://localhost:3456/new?url=https://example.com"     # 新建 tab
curl -s -X POST "http://localhost:3456/eval?target=ID" -d 'document.title'  # 执行 JS
curl -s -X POST "http://localhost:3456/click?target=ID" -d 'button.submit'  # JS 点击
curl -s -X POST "http://localhost:3456/clickAt?target=ID" -d '.upload-btn'  # 真实鼠标点击
curl -s "http://localhost:3456/screenshot?target=ID&file=/tmp/shot.png"     # 截图
curl -s "http://localhost:3456/scroll?target=ID&direction=bottom"           # 滚动
curl -s "http://localhost:3456/close?target=ID"                             # 关闭 tab
```

---

## 设计哲学

> Skill = 哲学 + 技术事实，不是操作手册。讲清 tradeoff 让 AI 自己选，不替它推理。

详见 [SKILL.md](./SKILL.md) 中的浏览哲学部分。

---

## 致谢

- 原项目：[web-access](https://github.com/eze-is/web-access) by [一泽 Eze](https://github.com/eze-is)
- 平台：[OpenClaw](https://github.com/openclaw/openclaw)

## License

MIT
