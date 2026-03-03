# CTRL.ai — 部署指南

## 架构总览

```
手机/平板/电脑
     │
     ▼
Cloudflare Access (认证)
     │
     ▼
Cloudflare Pages (静态托管)
     │
     ├── OpenClaw API (AI对话) ──► 直接从浏览器调用
     └── GitHub Gist API (Tasks存储) ──► 直接从浏览器调用
```

---

## 1. 准备 GitHub Token

1. 访问 https://github.com/settings/tokens/new
2. Note: `CTRL.ai`
3. Scopes: 仅选 **gist**
4. 生成并复制 token（`ghp_...`）

---

## 2. 部署到 Cloudflare Pages

### 方法 A：直接上传（最快）

1. 登录 https://dash.cloudflare.com
2. 侧边栏 → **Workers & Pages** → **Create** → **Pages**
3. 选择 **Upload assets**
4. 把 `index.html`、`_headers`、`_redirects` 拖入上传
5. 项目名称：`ctrl-ai`（或自定义）
6. 点击 **Deploy site**
7. 得到 URL：`https://ctrl-ai.pages.dev`

### 方法 B：连接 GitHub 仓库（推荐，自动部署）

1. 将这个仓库 push 到你的 GitHub
2. Cloudflare Pages → **Connect to Git**
3. 选择仓库，Build settings 留空（纯静态）
4. Deploy

---

## 3. 配置 Cloudflare Access（认证保护）

1. Cloudflare Dashboard → **Zero Trust** → **Access** → **Applications**
2. **Add an application** → **Self-hosted**
3. Application name: `CTRL.ai`
4. Application domain: `ctrl-ai.pages.dev`（你的 Pages URL）
5. **Policies** → Add policy:
   - Policy name: `Owner only`
   - Action: **Allow**
   - Include: **Emails** → 填入你的邮箱（或用 Google OAuth）
6. Save

之后每次访问会弹出 Cloudflare 认证页面，输入邮箱收 OTP 码即可进入。

---

## 4. 配置 CTRL.ai

打开 `https://ctrl-ai.pages.dev`，点击 **SETTINGS**：

| 字段 | 值 |
|------|-----|
| API Endpoint | `https://api.openclaw.ai/v1` 或其他兼容接口 |
| API Key | 你的 API Key |
| 默认模型 | `gpt-4o` / `claude-3-5-sonnet-20241022` 等 |
| GitHub Token | 第1步生成的 `ghp_...` |
| Gist ID | 留空自动创建，或填已有 Gist ID |

点击 **Test Connection** 验证，然后就可以使用了。

---

## 本地开发

```bash
# 任意 HTTP 服务器，如：
npx serve .
# 或
python3 -m http.server 8080
```

访问 `http://localhost:8080`

---

## 文件说明

```
index.html    ── 主应用（单文件，包含所有逻辑）
_headers      ── Cloudflare Pages HTTP 响应头
_redirects    ── SPA 路由重定向规则
DEPLOY.md     ── 本文件
```

---

## 数据存储

- **API Keys / 配置**：存在 `localStorage`，不离开设备
- **Tasks**：存在 GitHub Gist（私有），跨设备同步
- **对话历史**：存在内存，刷新页面后清空（v0.1）
  - v0.2 计划：对话历史也存 Gist
