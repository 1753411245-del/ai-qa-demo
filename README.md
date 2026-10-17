# 农业助手 · AI 问答单页 Demo

一个零依赖、单文件的 AI 问答网页演示，用来快速展示"客户问一句话 → 立刻拿到标准答案"的效果。
双击就能打开，不需要登录、不需要后端、不需要安装依赖。

适合：产品演示、客户汇报、给领导看效果、作为接入真实大模型前的交互原型。
内置示例围绕农业种植与农资使用场景（用法用量、混用禁忌、效果判断、购买渠道），换成其他行业只需替换知识库。

## 特性

- **单文件**：HTML + CSS + JS 全在 `index.html` 里，无构建、无外部依赖、可离线运行
- **完整聊天体验**：气泡式消息、多轮对话、Enter 发送 / Shift+Enter 换行、输入框自动增高、自动滚动到底
- **真实感细节**："正在思考"三点动画 + 打字机逐字输出
- **模拟知识库**：11 条内置问答，按关键词命中返回；未命中自动走兜底转人工
- **风控演示**：命中"烧苗/药害/索赔"等高风险词时，触发转人工提示
- **响应式**：桌面与手机浏览器自适应，移动端适配安全区
- **预留真实接口**：顶部 `CONFIG` 一块配置，改三行即可切换到大模型

## 快速开始

```bash
# 方式一：直接双击 index.html 用浏览器打开（演示模式可完全离线）

# 方式二：起一个本地服务（手机同 WiFi 可访问）
python -m http.server 8000
# 打开 http://localhost:8000
```

想在手机上试：电脑与手机连同一个 WiFi，用电脑内网 IP 访问，例如 `http://192.168.1.20:8000`
（Windows 查 IP：`ipconfig` → IPv4 地址）。

## 部署成在线链接

纯静态，任意静态托管都能放，把整个目录传上去即可：

- GitHub Pages：Settings → Pages → 选择分支根目录
- Vercel / Netlify：导入仓库，无需构建命令，输出目录填 `.`
- 对象存储（腾讯云 COS / 阿里云 OSS）：开启静态网站功能

## 接入真实大模型

打开 `index.html`，修改顶部的 `CONFIG`：

```js
const CONFIG = {
  useRealAPI: false,           // 改成 true 即走真实接口
  apiKey: 'YOUR_API_KEY_HERE', // 在这里填 Key
  apiUrl: 'https://api.deepseek.com/chat/completions',
  model: 'deepseek-chat'
};
```

接口采用 OpenAI 兼容格式，DeepSeek、通义千问、智谱、Kimi、自建 Dify 等都可以，换 `apiUrl` 与 `model` 即可。
请求失败时错误信息会直接显示在对话里，便于排查。

## 安全与隐私

本项目**不包含任何真实的 API Key、账号、密码、手机号或邮箱**，代码中只有占位符 `YOUR_API_KEY_HERE`。

但请务必注意：**把 Key 写在前端等于公开**，任何人打开页面按 F12 都能看到。演示阶段可以，正式上线请改成后端代理：

```
浏览器  →  你的后端（保存 Key，转发请求）  →  大模型厂商
```

最小可用的 Node 代理示例：

```js
const http = require('http');
const PORT = 3000, KEY = process.env.API_KEY;   // Key 放环境变量，不写进代码
http.createServer(async (req, res) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  if (req.method === 'OPTIONS') { res.writeHead(204); return res.end(); }
  let body = ''; req.on('data', c => body += c);
  req.on('end', async () => {
    const r = await fetch('https://api.deepseek.com/chat/completions', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', Authorization: 'Bearer ' + KEY },
      body
    });
    res.writeHead(r.status, { 'Content-Type': 'application/json' });
    res.end(await r.text());
  });
}).listen(PORT);
```

启动：`API_KEY=你的key node server.js`，然后把 `CONFIG.apiUrl` 指向 `http://localhost:3000`，`apiKey` 留空。

`.gitignore` 已排除 `.env`、`.workbuddy/`、`node_modules/` 等，避免误提交本地配置与工作区文件。

## 目录结构

```
.
├── index.html    # 全部代码：界面 + 样式 + 逻辑 + 模拟知识库 + 接口配置
├── README.md
├── LICENSE
└── .gitignore
```

## 自定义

- **换品牌/文案**：改 `<header>` 里的标题、副标题，以及开场白 `addBubble(...)`
- **换知识条目**：改 `KB` 数组，每项含 `id`、`keys`（关键词）、`text`（答案）
- **换快捷问题**：改 `SUGGESTS` 数组
- **调速度**：`CONFIG.thinkDelay`（思考延迟）、`CONFIG.typeSpeed`（打字速度）

## 演示技巧

1. 别自己手打，点下方的快捷问题条，命中率最高
2. 每条答案下方会标出「知识库条目 X-001」，用来说明答案有出处、不是模型瞎编
3. 故意问一句"打多了烧苗怎么办"，会触发转人工提示，适合展示风险控制

## 免责声明

内置问答内容为演示示例，仅用于展示交互效果，不构成任何使用建议。
涉及农药、肥料等农资产品的用法用量，请以产品最新标签、登记证信息及当地技术人员指导为准。

## License

[MIT](./LICENSE) © 2026 社恐腼腆大男
