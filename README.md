# 🚀 Cloudflare Worker 智能部署中控 (Worker Manager Pro V9.9.1)

> 全部代码为claude code 完成
> 自行修改延伸功能


> **当前版本**: V9.9.1 Pro
> **核心特性**: 批量部署 / 多账号管理 / 自动域名绑定 / 版本回滚 / 智能修复

这是一个运行在 Cloudflare Worker 上的**高级中控面板**，专为管理和批量部署复杂的 Worker 项目（如 EdgeTunnel, 少年你相信光吗, ECH Proxy 等）而设计。彻底解决了传统部署繁琐、配置难同步、资源难清理的痛点。

---

## ✨ 核心功能特性 (V9.9.1 更新)

* **⚡ 批量极速部署**
* **一键分发**：支持同时向多个 Cloudflare 账号部署新项目。
* **KV 智能开关**：新增 **KV 绑定开关**。针对 `Joey` 等无需 KV 的项目，可选择纯变量模式部署，节省资源。
* **自动配置**：自动处理代码上传、变量注入、UUID 生成和依赖绑定。


* **🔄 双向同步与修复**
* **同步修复 (Fix Sync)**：删除 Worker 时，自动同步清理中控端的账号配置，防止“幽灵项目”残留。
* **防崩溃保护**：自动检测并注入必要的环境变量（如 UUID），防止新项目启动失败。
* **上传修复 (Fix 808)**：内置修正版 `FormData` 协议，确保代码能正确推送到 CF 接口。


* **🧹 资源安全清理**
* **级联删除**：执行删除操作时，遵循 `获取绑定 -> 删除 Worker -> 删除 KV 空间` 的安全顺序，防止资源被占用导致删除失败。


* **🌐 域名深度管理**
* **预设域名读取**：自动读取账号下的 Zone（根域名）。
* **自定义前缀**：部署时只需输入前缀（如 `hk`），系统自动生成 `hk.你的域名.com` 并绑定。
* **隐私保护**：支持一键禁用 `*.workers.dev` 默认长域名，防止被扫描。


* **📜 版本控制大师**
* **时间轴对比**：直观显示 **上游最新提交时间** vs **本地部署时间**。
* **一键回滚**：支持查看 GitHub 历史并回滚到任意旧版本。
* **收藏夹**：将稳定版本加入收藏（Locked），防止误触更新。



---

## 🛠️ 部署教程 (保姆级)

只需简单 4 步，即可拥有自己的 Worker 中控台。

### 1️⃣ 第一步：创建主控 Worker

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2. 进入 **Workers & Pages** -> **Overview** -> **Create Application** -> **Create Worker**。
3. 命名为 `manager` (建议)，点击 **Deploy**。
4. 点击 **Edit code**，将本项目提供的 `worker.js` (V9.9.1) **完整代码** 粘贴覆盖。
5. 点击 **Save and deploy**。

### 2️⃣ 第二步：绑定 KV 存储 (⚠️ 核心)

**中控本身需要一个 KV 来存储账号数据，不绑定无法启动！**

1. 在 Worker 编辑页面的 **Settings** (设置) -> **Variables** (变量)。
2. 找到 **KV Namespace Bindings**，点击 **Add binding**。
3. **Variable name**: 填写 `CONFIG_KV` (**必须大写，完全一致**)
4. **KV Namespace**: 点击 "Create new KV namespace"，命名为 `manager_data`，点击 **Add**。
5. 点击 **Save and deploy**。

### 3️⃣ 第三步：设置安全密码

1. 同样在 **Settings** -> **Variables** -> **Environment Variables**。
2. 点击 **Add variable**：
* **Variable name**: `ACCESS_CODE`
* **Value**: 设置你的登录密码（如 `admin888`）。


3. *(可选但推荐)* 防止 GitHub API 限流：
* **Variable name**: `GITHUB_TOKEN`
* **Value**: 你的 GitHub PAT (获取方式见下文)。


4. 点击 **Save and deploy**。

### 4️⃣ 第四步：开始使用

访问你的 Worker 域名（如 `https://manager.你的前缀.workers.dev`），输入密码即可进入控制台。

---

## 🔑 核心数据获取指南

### 🅰️ Cloudflare 账号信息

在添加账号时需要填写：

* **Account ID**: 登录 CF 后，URL 地址栏 `dash.cloudflare.com/` 后面的那串字符。
* **Global API Key**:
1. 点击右上角头像 -> **My Profile** -> **API Tokens**。
2. 找到 **Global API Key** -> View -> 输入密码复制。


* *注意：必须使用 Global Key，普通 Token 权限不足以创建 KV 和绑定域名。*



### 🅱️ GitHub Token (用于解除限流)

如果不配置此项，GitHub 每小时限制请求 60 次，可能导致无法检查更新。

1. 登录 GitHub -> 头像 -> **Settings** -> **Developer settings**。
2. **Personal access tokens** -> **Tokens (classic)** -> **Generate new token (classic)**。
3. **Scopes**: 如果只用公共仓库（如 cmliu），**不需要勾选任何权限**。
4. 生成并复制 `ghp_` 开头的 Token。

---

## 📖 常用操作指南

### ✨ 批量部署新项目

1. 点击顶部「**✨ 批量部署**」。
2. **模板选择**：
* `CMliu`: 经典 EdgeTunnel，建议开启 KV。
* `Joey`: 推荐关闭 KV (取消勾选 "绑定 KV 存储")，使用纯变量模式。


3. **KV 设置**：如果开启 KV，请填写 KV 名称（中控会自动创建）。
4. **域名设置**：
* 勾选 `禁用默认域名` 可提高隐蔽性。
* 填写 `自定义域名` 前缀（前提：账号已读取到预设域名）。


5. 勾选目标账号 -> **🚀 开始部署**。

### 🔄 变量同步 (反向更新)

如果你在 Cloudflare 后台手动修改了某个 Worker 的变量：

1. 在中控面板找到该项目。
2. 点击「**🔄 同步**」。
3. 中控会将云端的最新配置拉取回本地数据库，确保数据一致。

### 🗑️ 安全删除

1. 点击账号右侧的「**📂 管理**」。
2. 点击「**🗑️ 删除**」。
3. **勾选 "同时删除绑定的 KV"** (推荐)，系统将自动清理残留资源。

---

## 📝 内置模板说明

| 模板代码 | 项目名称 | 特性说明 | 建议配置 |
| --- | --- | --- | --- |
| **cmliu** | EdgeTunnel (Beta 2.0) | 功能最全，支持订阅 | 开启 KV |
| **joey** | 少年你相信光吗 | 自动修复，极简 | **关闭 KV** (变量模式) |
| **ech** | ECH Proxy | 无需维护，WebSocket | 关闭 KV |

---

## ⚠️ 免责声明

本项目仅供技术研究和学习使用，请勿用于任何非法用途。开发者不对使用本工具产生的任何后果负责。您的 API Key 仅保存在您自己的 Cloudflare KV 中，请妥善保管。
