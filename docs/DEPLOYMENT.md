# 部署到 Cloudflare

[![Deploy to Cloudflare](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/avsba001/personal-blog-cloudflare-template)

## 首次部署

1. 点击按钮，登录 Cloudflare 和 GitHub，授权创建新仓库。首次使用 R2 的账号，需要先在 Cloudflare 控制台开通 R2。
2. 选择 Worker、D1、KV 和 R2 的名称。Cloudflare 会创建资源、把新 ID 写入生成的仓库，并绑定到 Worker。不要选正在使用的数据库或存储桶来测试模板。
3. 在 Secrets 中填写 `JWT_SECRET`：至少 32 个字符的独立随机值，可用 `openssl rand -hex 32`，或 `node -p "require('node:crypto').randomBytes(32).toString('hex')"` 生成。不要填到 Git。若部署界面未显示该输入框，部署后在 Worker 的 **Settings → Variables and Secrets** 添加同名 Secret。
4. 保持根目录为仓库根目录，Build command 为 `npm run build`，Deploy command 为 `npm run deploy`。Workers Builds 会安装锁文件中的 npm 依赖；部署命令会先检查后端类型，再按 `DB` 绑定应用尚未执行的 D1 迁移，最后部署 Worker。
5. 打开 Cloudflare 显示的 `workers.dev` 地址。首页和 API 共用一个 Worker，无需另外创建 Pages，也不用开启 R2 的公开地址。可访问 `/api/health` 检查 Worker，再访问 `/api/posts` 检查数据库。

## 设置管理员

模板没有默认管理员，也不会把第一个注册的人自动提为管理员。先在新站点注册自己的账号，再在 **Cloudflare → D1 → 此博客的数据库 → Console** 查出并核对账号 ID：

```sql
SELECT id, username, email, role FROM users WHERE username = '替换为你的用户名' AND id > 0 AND deleted_at IS NULL;
```

核对后将 `123` 换成查出的 ID：

```sql
UPDATE users SET role = 'admin', updated_at = CURRENT_TIMESTAMP WHERE id = 123 AND id > 0 AND deleted_at IS NULL;
```

退出后重新登录，再访问 `/admin`。

## 可选配置

| 配置 | 用途 |
| --- | --- |
| 自定义域名 | Worker 的 **Settings → Domains & Routes** 中添加。不配置时也可使用 `workers.dev`。 |
| `FRONTEND_URL` | 站点规范 URL，例如 `https://blog.example.com`。启用邮件发送前设置，邮件中的链接使用它。基础站点不要求配置。 |
| `RESEND_API_KEY`、`RESEND_FROM_EMAIL` | 邮箱验证码和邮件通知。正式发送需要在 Resend 验证发件域名。 |
| `GITHUB_CLIENT_ID`、`GITHUB_CLIENT_SECRET` | GitHub OAuth 登录。按实际站点 URL 配置 OAuth App 的回调地址。 |
| `POST_PASSWORD_SECRET` | 可选，专用于文章访问令牌；不设置时沿用 `JWT_SECRET`。 |
| `STORAGE_PUBLIC_URL` | 可选，改用独立的 R2 公开域名。默认由同一个 Worker 读取私有桶，图片内联展示，普通附件以下载方式交付。 |

`FRONTEND_URL` 和 `STORAGE_PUBLIC_URL` 可以在生成仓库的 `wrangler.toml` 中维护，不带尾部 `/`。实际密钥只在 Cloudflare Secrets 中管理。

## 更新与手动发布

部署按钮创建的仓库已连接 Workers Builds，后续推送会自动构建；不需要 GitHub Deploy Hook，也不用给 GitHub 配 Cloudflare API Token。迁移失败会阻止发布。新增数据库变更请添加新的 SQL 文件，不要更改已经应用的迁移文件。

若从已初始化的部署仓库手动发布，请先确认 `wrangler.toml` 中的 D1/KV ID 和 R2 桶名属于该环境，再从仓库根目录执行：

```bash
npm ci
npx wrangler login
npm run build
npm run deploy
```

不要用部署按钮原地更新上游博客。它用于创建新站点，不会复制已有站点的数据或 Secrets。已有生产库接入前应先备份并检查迁移记录。
