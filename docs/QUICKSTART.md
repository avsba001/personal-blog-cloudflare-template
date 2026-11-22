# 本地开发

部署到 Cloudflare 请看[部署指南](./DEPLOYMENT.md)。本地开发需要 Node.js 22、npm 和 Git，不需要 Cloudflare 账号，D1、KV、R2 和 Durable Objects 均由 Wrangler 在本地模拟。

从仓库根目录执行：

```bash
npm ci
cp .dev.vars.example .dev.vars
```

在 `.dev.vars` 中填写随机的 `JWT_SECRET`（至少 32 个字符），只用于本地开发。可用 `node -p "require('node:crypto').randomBytes(32).toString('hex')"` 生成。不要提交 `.dev.vars`。

```bash
npx wrangler d1 migrations apply DB --local
npm run build
npm run dev:backend
```

访问 `http://localhost:8787`。需要前端热更新时，保留 Wrangler 进程，在另一个终端执行 `npm run dev` 并访问 `http://localhost:5173`。Vite 会将 `/api` 转发给 Wrangler。

开发时如需管理员，在本地注册自己的账号，再参照部署指南中的 SQL，用 `npx wrangler d1 execute DB --local --command="SQL"` 查询并更新该账号。不要对本地开发命令使用 `--remote`。
