# 🧠 思维导图喵：Cloudflare 实时协作版

这个版本保留原来的单页思维导图 UI，改成：

- 前端：Yjs + y-websocket + y-indexeddb
- 实时后端：Cloudflare Workers + Durable Objects
- 持久化：Durable Object SQLite 存储
- 静态网页：Workers Static Assets
- 房间：URL 的 `?room=...`

## 目录

```text
mindmap-collab/
├─ public/
│  └─ index.html
├─ src/
│  └─ index.ts
├─ package.json
├─ tsconfig.json
├─ wrangler.jsonc
├─ .gitignore
└─ README.md
```

## 本地运行

需要 Node.js 18+。首次安装依赖需要能访问 npm registry。

```bash
npm install
npm run dev
```

然后打开 Wrangler 输出的本地地址。

用两个浏览器窗口打开同一个 `?room=xxxx`，即可测试实时同步。

## 部署到 Cloudflare

先登录：

```bash
npx wrangler login
```

然后部署：

```bash
npm install
npm run deploy
```

首次部署时 Cloudflare 会根据 `wrangler.jsonc` 创建 SQLite-backed Durable Object 命名空间，并把 `public/` 里的网页一起发布。

部署成功后，Cloudflare 会给出类似：

```text
https://mindmap-collab.<你的子域>.workers.dev
```

打开这个网址即可使用。

注意：首次部署的 Worker 需要创建 Durable Object 命名空间；当前配置采用 Cloudflare 推荐的 SQLite-backed Durable Object。

## 邀请朋友

1. 你先打开网站。
2. 地址栏会自动带一个随机 `?room=xxxx`。
3. 点击网页里的「🔗 分享」。
4. 把复制出来的完整链接发给朋友。
5. 对方打开同一个链接，就是同一个 Yjs 房间。

## GitHub

把整个 `mindmap-collab` 文件夹作为一个仓库上传即可。不要上传 `node_modules/` 和 `.wrangler/`。

示例：

```bash
git init
git add .
git commit -m "feat: realtime mindmap with Cloudflare Durable Objects"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<你的仓库>.git
git push -u origin main
```

## Cloudflare GitHub 自动部署（可选）

也可以把 GitHub 仓库连接到 Cloudflare 的 Workers Build，然后每次 push 自动重新部署。第一次建议先用 `npm run deploy` 验证 Worker、Durable Object 和网页都正常，再接自动部署。

## 注意

当前图片仍然以压缩后的 JPEG Data URL 作为 Yjs 节点字段同步，所以适合小图片。不要把大量原图长期放进思维导图，否则房间数据会迅速变大。后续如果需要大量图片，可以改成 R2 + 图片 URL。


## GitHub Actions 自动部署

仓库里已经附带 `.github/workflows/deploy.yml`。push 到 `main` 后会自动执行 Cloudflare 的官方 Wrangler Action。Cloudflare 的 GitHub Actions 文档要求在 GitHub Secrets 中保存 `CLOUDFLARE_ACCOUNT_ID` 和 `CLOUDFLARE_API_TOKEN`；不要把 token 直接写进仓库。

推荐的最小流程是：先本地 `npm run deploy` 成功一次，再配置 GitHub Secrets。
