# Waline 留言系統整合計畫

## Context

在文章頁面加入留言區，使用 [Waline](https://waline.js.org/)：輕量、開源，後端部署在 Vercel，資料庫使用 Neon（PostgreSQL）。留言僅加在 `post` 頁面（`/posts/*`），不加在隨筆（essays）。

---

## 步驟一：手動操作（需要你自行完成）

### 1. 資料庫設定（Neon，免費）

LeanCloud 已停止新用戶註冊，改用 **Neon**（PostgreSQL，512MB 免費，閒置時自動暫停並在收到請求時自動喚醒，適合個人部落格）。

建立 Neon 資料庫有兩種方式，擇一即可：

**方式 A：透過 Vercel Dashboard（推薦，環境變數自動注入）**
1. 進入 Vercel 主站專案 → 「Storage」→「Create Database」→ 選擇「Neon Postgres」
2. 建立後，Vercel 會自動將連線資訊注入到專案環境變數（`POSTGRES_URL` 等）
3. 記下 `POSTGRES_HOST`、`POSTGRES_DATABASE`、`POSTGRES_USER`、`POSTGRES_PASSWORD`（之後填入 Waline 後端）

**方式 B：直接在 Neon 官網建立**
1. 前往 https://neon.tech 註冊帳號
2. 建立新專案與資料庫
3. 進入「Connection Details」複製連線資訊

### 2. Vercel 部署 Waline 後端

1. 點擊一鍵部署：https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fwalinejs%2Fwaline%2Ftree%2Fmain%2Fexample
2. 連結 GitHub，讓 Vercel 建立一個新 repo（例如 `waline-server`）
3. 在 Vercel **Waline 後端**部署設定的「Environment Variables」加入：

   | 環境變數 | 填入 Neon 的哪個值 |
   |---|---|
   | `PG_HOST` | `PGHOST_UNPOOLED` 的值（直連，避免 pgbouncer 相容問題） |
   | `PG_PORT` | `5432` |
   | `PG_DB` | `PGDATABASE` 的值 |
   | `PG_USER` | `PGUSER` 的值 |
   | `PG_PASSWORD` | `PGPASSWORD` 的值 |
   | `PG_SSL` | `true` |
4. 部署完成後，複製 Waline server URL（例如 `https://your-waline.vercel.app`）

5. 訪問 `https://your-waline.vercel.app/ui/register` 註冊管理員帳號

### 3. Vercel 主站環境變數設定

在 Vercel 專案的「Settings」→「Environment Variables」加入：
- `WALINE_SERVER_URL` = 你的 Waline server URL

---

## 步驟二：專案代碼調整（Claude 執行）

### 修改的檔案

| 檔案 | 動作 |
|------|------|
| `astro.config.ts` | 加入 `WALINE_SERVER_URL` 到 env schema |
| `src/components/blog/WalineComment.astro` | 新建 Waline 元件 |
| `src/layouts/BlogPost.astro` | 引入並插入 WalineComment 元件 |

### 1. `astro.config.ts` — 加入 env 宣告

在現有 `env.schema` 區塊（約第 134 行）加入：

```ts
WALINE_SERVER_URL: envField.string({
  context: "client",
  access: "public",
  optional: true,
}),
```

### 2. 新建 `src/components/blog/WalineComment.astro`

```astro
---
import { WALINE_SERVER_URL } from "astro:env/client";
---

{WALINE_SERVER_URL && (
  <section class="mt-10 pt-6 border-t border-color-200">
    <div id="waline"></div>
  </section>
)}

<script define:vars={{ serverURL: WALINE_SERVER_URL }}>
  if (serverURL) {
    import("@waline/client").then(({ init }) => {
      import("@waline/client/style");
      init({ el: "#waline", serverURL });
    });
  }
</script>
```

### 3. `src/layouts/BlogPost.astro` — 插入元件

在 `<WebMentions />` 之後（約第 66 行）加入：

```astro
import WalineComment from "@/components/blog/WalineComment.astro";
// ...
<WebMentions />
<WalineComment />
```

---

## 本地開發測試方式

1. 在專案根目錄建立 `.env` 檔案（已被 .gitignore 忽略）：
   ```
   WALINE_SERVER_URL=https://your-waline.vercel.app
   ```
2. 執行 `pnpm dev`，開啟任一文章頁面，確認留言框出現
3. 測試發送留言是否成功送達 Waline 後台（`/ui`）

## 驗證清單

- [ ] 文章頁面底部出現 Waline 留言框
- [ ] 可匿名或登入留言
- [ ] 留言顯示在 Waline 管理後台 `/ui`
- [ ] 未設定 `WALINE_SERVER_URL` 時，留言框不顯示（不報錯）
- [ ] 隨筆（/essays/）頁面不受影響
