# CLAUDE.md

此檔案提供 Claude Code (claude.ai/code) 在此儲存庫中工作時的指引。

## 指令

```bash
pnpm dev          # 啟動開發伺服器（host: true，可在區域網路存取）
pnpm build        # 將正式環境網站建置至 ./dist/
pnpm postbuild    # 建置後執行 pagefind 索引器（搜尋功能必須執行此步驟）
pnpm preview      # 在本地預覽正式環境建置結果
pnpm lint         # Biome 程式碼檢查
pnpm format       # 完整格式化（biome format + prettier + import 排序）
pnpm check        # Astro 型別檢查
```

Pagefind 搜尋功能僅在完整執行 `build && postbuild` 後才能運作，開發模式下無法使用。

## 架構

這是一個 **Astro v5** 部落格（基於 Astro Citrus 模板），部署於 Vercel。網站使用繁體中文（zh-TW），網址為 `https://www.billyji.com/`。

### 內容集合

三個集合定義於 `src/content.config.ts`：

- **post** — `src/content/post/` — 完整的部落格文章。每篇文章為一個資料夾，包含 `index.md`（或 `.mdx`）及相關圖片。主要 frontmatter：`title`、`description`、`publishDate`、`tags`、`draft`、`coverImage`、`ogImage`、`seriesId`、`orderInSeries`。
- **note** — `src/content/note/` — 較短的隨筆。可以是單一 `.md` 檔案或資料夾。
- **series** — `src/content/series/` — 透過 `seriesId` 將文章分組成系列。系列檔案包含 `id`、`title`、`description`、`featured`。

`draft: true` 的文章會從正式環境建置、RSS、OG 圖片及所有 `getAllPosts()` 呼叫中排除。

### 重要原始碼路徑

- `src/site.config.ts` — 網站後設資料（`author`、`title`、`description`、`lang`）與 `menuLinks`。
- `src/styles/global.css` — 亮色／暗色主題的 CSS 變數。
- `src/plugins/` — 兩個自訂 remark 插件：`remark-reading-time.ts` 與 `remark-admonitions.ts`（處理 `:::note`、`:::tip` 等指令語法）。
- `src/pages/og-image/[slug].png.ts` — 基於 Satori 的每篇文章 OG 圖片產生器。
- `src/pages/rss.xml.ts` — RSS 訂閱。

### Markdown 處理流程

語法高亮使用 **rehype-pretty-code**（Shiki），主題為 `rose-pine`（暗色）和 `rose-pine-dawn`（亮色）。更換主題後需要重新啟動開發伺服器。提示區塊（Admonitions）透過 `remark-directive` 搭配自訂 `remarkAdmonitions` 插件，使用 `:::type` 指令語法。

### 工具鏈

- **Biome** — 非 Astro 檔案的程式碼檢查與格式化（tabs、縮排寬度 2、行寬 100）。
- **Prettier** — `.astro` 檔案及其他 Biome 未涵蓋檔案的格式化。
- **Pagefind** — 靜態搜尋，範圍限定於 `BlogPost.astro` 和 `Note.astro` 中的 `data-pagefind-body` 元素。
- **pnpm** — 套件管理器（參見 `pnpm-lock.yaml`）。
