# minimax-vortex

> 水感知識庫 — minimax-vortex 獨立站（基於 5 份設計研究 + 41 項 checklist）

## 🚀 怎麼看這個網站

**網址**：[https://hangsau.github.io/minimax-vortex/](https://hangsau.github.io/minimax-vortex/)

**首次設定**（站主一次性操作）：
1. 進 GitHub repo：[`Hangsau/minimax-vortex`](https://github.com/Hangsau/minimax-vortex) → Settings → Pages
2. Source 設為 **「GitHub Actions」**（不要用「Deploy from a branch」— 那是舊 v4 的 gh-pages 部署方式）
3. 之後每次 `git push` 會自動 build + deploy（~2-5 分鐘）

**驗證 deploy 狀態**：
- GitHub Actions：[`/actions`](https://github.com/Hangsau/minimax-vortex/actions)
- 找 workflow「Deploy minimax-vortex to GitHub Pages」— 看到 ✓ success = 上線
- 如果顯示「Queued」或「In progress」— 等一下再看

**為什麼剛 push 還看不到 v0.2.1**：
- v4 時代 Pages 用「gh-pages branch」部署（GitHub 預設）
- v0.2.1 改成 GitHub Actions — 站主要在 Settings 手動切
- 切換後，新 push 才會 deploy 到 Pages

**Troubleshooting**：

| 症狀 | 原因 | 解法 |
|------|------|------|
| 網站顯示 v4 內容 | Pages 還在 deploy gh-pages branch | Settings → Pages → Source 改「GitHub Actions」 |
| 404 Not Found | repo 還沒啟用 Pages | Settings → Pages → 啟用，選 source |
| workflow 顯示失敗 | build error（npm install / astro build） | 看 run log，修正後重 push |
| workflow queued 太久 | GitHub 排隊 | 等（高峰期可能 10+ 分鐘） |

---

## 技術棧

- **Astro 5.18.2** — 靜態內容站（SSG）
- **TypeScript strict**
- 無 React / Vue — 純 Astro components + 少量 vanilla JS
- **GitHub Pages** deploy via GitHub Actions

## 設計語言

**techo（techo 日式文具手帳）** — 站主決定方向

- 字型：Shippori Mincho + Inter + Courier Prime + Noto Sans/Serif TC（全部 OFL）
- 強調色：#B8511F（柿橘深 — WCAG AA 對比 4.83:1）
- Baseline grid：30px 物理化（repeating-linear-gradient 橫罫線）
- Modular scale：1.25（base 16px）

完整設計規範見 [`DESIGN_SYSTEM.md`](./DESIGN_SYSTEM.md)。

## 結構

```
minimax-vortex/
├── DESIGN_SYSTEM.md        # 完整設計規範（11 章節）
├── README.md                # 本檔
├── HANDOFF.md               # 開發交接單 + 已知問題
├── package.json             # Astro 5 + sitemap + rss + mdx
├── astro.config.mjs         # base: /minimax-vortex
├── tsconfig.json            # TypeScript strict
├── src/
│   ├── components/          # MiniLadder, StrokePage
│   ├── content/             # 43 個 markdown（從 cortex 同步）
│   ├── data/                # 12 個 YAML（drills 125 + indicators 43 + errors 76 + tech 188 + ...）
│   ├── layouts/             # Base.astro
│   ├── pages/               # 15 個路由
│   └── styles/              # global.css (design tokens)
├── .github/workflows/       # deploy.yml (GitHub Pages)
└── dist/                    # build 輸出（GitHub Actions 用）
```

## 頁面結構

| 路由 | layout | 內容 |
|------|--------|------|
| `/` | 首頁 | Hero + 3 大群 + 1 摺合區 |
| `/strokes/{freestyle,backstroke,breaststroke,butterfly,underwater-dolphin-kick,starts-turns}/` | master-detail | 各式動作分解 |
| `/database/` | 雙區 | 依需求找練習 + 跨式查資料 |
| `/psychology/` | 三帶 | 5 處境卡 + L0-L6 |
| `/levels/` | 章節 | L0-L6 發展序列 |
| `/periodization/` | 期刊 | 6 節 Bompa Periodization |
| `/adm/` | 入口 | 4 支柱 × 4 階段 |
| `/adm/matrix/` | 矩陣 | 16 格 master-detail |
| `/adm/standards/` | 卡片 | 22 項技術標準 |
| `/adm/background/` | 長文 | LTD / 獎牌台 / 八大考量 |

## 開發指令

```bash
# 安裝依賴
npm install

# 開發 server（http://localhost:4321）
npm run dev

# Build
npm run build

# Preview build
npm run preview
```

## Deploy 到 GitHub Pages

### 一次性設定

1. **建立 GitHub repo**：`Hangsau/minimax-vortex`（站主執行）
2. **啟用 GitHub Pages**：Settings → Pages → Source = "GitHub Actions"
3. **push source code**：

```bash
# 設定 remote（站主執行，token 從 ~/.git-credentials 讀）
cd /home/hangsau/projects/minimax-vortex
TOKEN=$(awk -F'[:@]' '{print $3}' ~/.git-credentials)
git remote add origin "https://${TOKEN}@github.com/Hangsau/minimax-vortex.git"
git push -u origin main
```

4. **第一次 push 觸發 GitHub Actions**：`.github/workflows/deploy.yml` 自動 build + deploy
5. **網站上線**：`https://hangsau.github.io/minimax-vortex/`

### 後續更新

`git push` 觸發自動 deploy。

## 內容同步

內容真相源：`~/projects/TheVortexProject/canonical/`（cortex 站 vortex section 也是這個）

同步流程：
1. 改 `TheVortexProject/canonical/` 的 YAML / markdown
2. 跑 `tools/sync_vortex.py`（cortex 站已有的 sync script）→ 產出 `~/projects/cortex/data/vortex/*.yaml`
3. **手動複製** 到 minimax-vortex：
   ```bash
   cp -v ~/projects/cortex/data/vortex/*.yaml ~/projects/minimax-vortex/src/data/
   cp -rv ~/projects/cortex/content/vortex/* ~/projects/minimax-vortex/src/content/
   ```
4. push → 自動 deploy

（未來可改成 git submodule，但目前不必要）

## 41 項 checklist 現況（v0.2.1）

- ✓ 38 / ✗ 1（size-adjust font fallback）/ 部分 2-3
- 詳見 [`HANDOFF.md` §2](./HANDOFF.md#2-41-項-checklist-現況)

## 設計研究來源

5 份研究 + 41 項 checklist + 2 份 L6 摘要：

- `~/projects/cortex/research/vortex-rebuild/research-log.md`（網站結構）
- `~/projects/cortex/research/entry-wayfinding-study.md`（IA / wayfinding）
- `~/projects/cortex/research/presentation-layout-study.md`（視覺層級 / 閱讀動線）
- `~/projects/cortex/research/vortex-rebuild/design-principles-deep-dive.md`（設計原理）
- `~/projects/cortex/research/vortex-rebuild/design-craft-meta-research.md`（design craft）
- `~/projects/cortex/research/vortex-rebuild/non-reskin-checklist.md`（41 項 checklist）

## License

內容以「教練觀測 + 近期文獻」為主，公開層不放感知判讀診斷語。授權細節見各內容 frontmatter。
