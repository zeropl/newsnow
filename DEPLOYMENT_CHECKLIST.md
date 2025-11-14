# Cloudflare Pages 部署清单

## 📋 部署前准备

### 1. 创建 Cloudflare D1 数据库

**步骤：**
1. 访问 [Cloudflare Dashboard](https://dash.cloudflare.com)
2. 进入 **Workers & Pages** → **D1**
3. 点击 **Create database** 按钮
4. 数据库名称输入：`newsnow-db`
5. 点击创建后，记录下显示的 **Database ID**

**需要记录的信息：**
- ✅ Database ID（格式：`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`）

---

## 🔧 配置文件修改

### 修改 `wrangler.toml` 文件

找到文件中的这一行：
```toml
database_id = ""
```

将你的 Database ID 填入：
```toml
database_id = "你的-database-id-在这里"
```

---

## 🌐 环境变量配置

### 在 Cloudflare Pages 中设置环境变量

**方式 A：通过 wrangler.toml 配置（已包含）**

以下变量已在 `wrangler.toml` 的 `[vars]` 部分配置：
- ✅ `INIT_TABLE = "true"` - 初始化数据库表
- ✅ `ENABLE_CACHE = "true"` - 启用缓存功能

**方式 B：通过 Cloudflare Dashboard 配置（可选补充）**

如果需要添加额外的环境变量：
1. 部署后进入项目的 **Settings** → **Environment variables**
2. 添加变量（区分生产环境和预览环境）

### 可选的环境变量（不需要登录功能时可忽略）

这些变量仅在需要登录功能时配置：
- ❌ `G_CLIENT_ID` - GitHub OAuth Client ID（跳过）
- ❌ `G_CLIENT_SECRET` - GitHub OAuth Secret（跳过）
- ❌ `JWT_SECRET` - JWT 签名密钥（跳过）

其他可选变量：
- 🔹 `PRODUCTHUNT_API_TOKEN` - Product Hunt API 访问令牌（仅需要 PH 数据源时）

---

## 🚀 部署步骤

### 方式 1：命令行部署（推荐）

```bash
# 1. 确保已修改 wrangler.toml 中的 database_id
# 2. 登录 Cloudflare（如果未登录）
pnpm wrangler login

# 3. 构建并部署
pnpm deploy
```

### 方式 2：通过 GitHub 集成自动部署

1. 将代码推送到 GitHub 仓库
2. 在 Cloudflare Pages 中连接 GitHub 仓库
3. 配置构建设置：
   - **Framework preset**: None
   - **Build command**: `pnpm run build`
   - **Build output directory**: `dist/output/public`
   - **Root directory**: `/`
4. 添加环境变量（D1 数据库绑定）
5. 触发部署

---

## ✅ 部署后验证

部署完成后，访问你的 Cloudflare Pages URL：
- `https://newsnow.pages.dev` 或你的自定义域名

**测试功能：**
1. ✅ 页面正常加载
2. ✅ 新闻源数据正常显示
3. ✅ 切换栏目功能正常
4. ✅ 暗黑模式切换正常
5. ✅ 拖拽排序功能正常

**注意：** 由于跳过了 OAuth 配置，以下功能将不可用：
- ❌ 用户登录
- ❌ 云端数据同步
- ❌ 手动强制刷新缓存

这些功能不影响核心的新闻浏览体验。

---

## 📝 最终检查清单

部署前确认：
- [ ] 已创建 D1 数据库
- [ ] 已将 Database ID 填入 `wrangler.toml`
- [ ] 已验证 `wrangler.toml` 中的环境变量配置正确
- [ ] 已提交所有代码更改到 Git

部署后确认：
- [ ] 网站可以正常访问
- [ ] 新闻数据正常加载
- [ ] 无 JavaScript 错误（检查浏览器控制台）
- [ ] 数据库连接正常（检查 Cloudflare Pages 日志）

---

## 🆘 常见问题

**Q: 部署后提示数据库连接错误？**
A: 检查 `wrangler.toml` 中的 `database_id` 是否正确，并确保 D1 数据库已创建。

**Q: 新闻数据不显示？**
A: 查看 Cloudflare Pages 的 Functions 日志，检查是否有 API 错误。可能是数据源网站的访问限制。

**Q: 如何更新部署？**
A: 修改代码后重新运行 `pnpm deploy` 即可。如果使用 GitHub 集成，推送代码会自动触发部署。

**Q: 如何查看部署日志？**
A: Cloudflare Dashboard → Workers & Pages → 选择项目 → View details → Functions 标签页

---

**部署文档更新日期**: 2025-11-14
**项目版本**: 0.0.36
