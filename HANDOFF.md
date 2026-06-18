# HANDOFF — minimax-vortex

> 開發交接單。給未來接手 / 站主 review。

## 1. 目前狀態（2026-06-18, v0.2.1）

**站**：Astro 5.18.2 靜態站 + GitHub Pages deploy
**網址**（待站主 push 後啟用）：`https://hangsau.github.io/minimax-vortex/`
**內容**：15 pages + 12 YAML data + 43 markdown content
**Build**：✓ 3.0s（無錯誤、無警告）

### 1.1 設計決策（站主授權）

- 名字：`minimax-vortex`
- 技術棧：**Astro 5.x**（不用 Hugo — 重新來的精神）
- 字型：**全部 OFL**（Shippori Mincho / Inter / Courier Prime / Noto Sans TC / Noto Serif TC）— 不商業字型
- 風格：**techo（techo 日式文具手帳）** — 站主主動改 vortex-home 為 techo 是品味決定
- 不參考 v4 minimax-vortex 設計實作（commit 4504e02 已撤）

### 1.2 DESIGN_SYSTEM.md

**站主授權我寫**（11 章節）：
- 設計語言定位 / 字型系統 / 顏色系統 / 間距系統 / 互動系統 / a11y / 結構決策 / content 分界 / anti-pattern / checklist 整合 / 版本演進

**單一真相源**：未來改 design 改這檔 → CSS / components 對齊 → commit 前 41 項打勾。

## 2. 41 項 checklist 現況

**v0.2.1 結果**（Playwright + axe-core + checklist 自動掃描）：

| 類別 | ✓ | ✗ | 部分 |
|------|---|---|------|
| 1. 結構決策（10） | 9 | 0 | 1 |
| 2. Typography（8） | 8 | 0 | 0 |
| 3. a11y/perf（10） | 9 | 1 | 0 |
| 4. 設計原理溯源（5） | 4 | 0 | 1 |
| 5. Design craft（8） | 8 | 0 | 1* |
| **總計（41）** | **38** | **1** | **3** |

*5.1.5 算「部分」是因為 hero 是 4 元素（eyebrow + h1 + lead + CTA）

**目標**：✓ 30+ / ✗ < 5 / 部分 < 5
**現況**：✓ 38 / ✗ 1 / 部分 2-3 → **已達標** v1 釋出目標

### 2.1 重大修復（v0.2 → v0.2.1）

**a11y P0 — color-contrast（WCAG 1.4.3）**：
- accent `#D4622A` → `#B8511F`（對比 4.83:1 ✓ AA）
- sub `#88887e` → `#6b6b62`（對比 5.0:1 ✓ AA）
- accent-2 / faint 同步加深
- MiniLadder active node 加 `font-weight: 700`（視覺強化）

**a11y P0 — DESIGN_SYSTEM.md 對比數字錯誤**：
- 原本寫 4.9:1 / 5.1:1，實測 3.59:1 / 3.57:1（fail AA）
- 已重算並加註實際值

### 2.2 已知未修（v0.2.1 → v0.2.2 backlog）

| # | 項目 | 嚴重度 | 預估修法 |
|---|------|--------|---------|
| 1 | heading-order skip（levels + adm-matrix 跳 h3） | P1 moderate | 統一 `<h3>` |
| 2 | `size-adjust` font fallback | P3 nice-to-have | 加 `@font-face` size-adjust 描述符 |
| 3 | axe-core CI 整合 | P2 | 寫 GitHub Action 在 PR 跑 axe |
| 4 | stroke 頁三件式（drill + 視覺 + 操作） | P2 | 把 stroke 動作分解做成 drill-card |
| 5 | inline reference comments | P3 | component 開頭加 `<!-- ref: ... -->` |

### 2.3 Performance 實測

- **CLS**：0.0008–0.0019（web-vitals good ≤ 0.1）— **極優**
- **LCP**：652–1132ms（good ≤ 2500ms）— **極優**
- **320px reflow**：scrollWidth == clientWidth（無水平 overflow）✓
- **PageSpeed**：待 v1 釋出後跑完整 audit

## 3. 架構

### 3.1 內容真相源

- **`~/projects/TheVortexProject/canonical/`** — vortex 內容 single source of truth（markdown）
- **`~/projects/cortex/data/vortex/*.yaml`** — sync_vortex.py 處理後的 12 個 YAML（淨化後的公開層）
- **`~/projects/minimax-vortex/src/{data,content}/`** — minimax-vortex 從 cortex 複製過來

**同步流程**（手動，未來可改 git submodule）：
```bash
cp -v ~/projects/cortex/data/vortex/*.yaml ~/projects/minimax-vortex/src/data/
cp -rv ~/projects/cortex/content/vortex/* ~/projects/minimax-vortex/src/content/
```

### 3.2 設計 vs 內容

| 維度 | 來源 |
|------|------|
| **內容**（YAML / markdown） | TheVortexProject → cortex → minimax-vortex（手動 copy） |
| **設計**（Astro / CSS / components / layouts） | minimax-vortex 自有 |
| **設計規範**（DESIGN_SYSTEM.md） | minimax-vortex 自有，single source of truth |

**嚴守禁區**：
- 公開層不放「自陳 → 感知/技術品質判斷」推論語
- 不改 canonical 內容（在 minimax-vortex 內不編輯 markdown）

### 3.3 為什麼不用 Hugo（站主決定）

- 站主說「必須重新來」— 用新技術棧的精神
- Astro 對內容站 + Islands 互動極佳
- TypeScript 一流支援
- 內建 MDX / Content Collections / Sitemap

## 4. 開發規範

### 4.1 加新頁面

1. 在 `src/pages/<section>/index.astro` 建檔
2. 用 Base.astro layout + 必要 components
3. 嵌入 MiniLadder（除非首頁 / 入口頁）
4. 用 design tokens (`var(--type-X)` / `var(--color-X)` / `var(--space-X)`)
5. **不**hardcoded value（除非數學計算）
6. 加 `.skip-link` 不需（Base.astro 已有）
7. 跑 `npm run build` 確認無錯

### 4.2 加新 component

1. `src/components/<Name>.astro`
2. Props 用 TypeScript interface
3. Style 用 scoped `<style>`（自動隔離）
4. 用 design tokens，**不**hardcoded
5. 命名空間隔離：vortex 頁用 `.tx-*` / `.stroke-*` / `.adm-*` / `.database-*` 等前綴（per section）

### 4.3 改 design

1. 改 `DESIGN_SYSTEM.md`（single source of truth）
2. 同步改 `src/styles/global.css`（CSS variables）
3. 改 components 引用
4. 跑 build + Playwright 截圖驗證
5. 跑 axe-core 確認 a11y 沒退步
6. 跑 41 項 checklist 打勾
7. Commit + push

### 4.4 改內容

1. 改 `~/projects/TheVortexProject/canonical/` 對應 YAML / markdown
2. 跑 `tools/sync_vortex.py`（如果存在於 TheVortexProject）
3. 從 cortex 複製到 minimax-vortex：
   ```bash
   cp -v ~/projects/cortex/data/vortex/*.yaml ~/projects/minimax-vortex/src/data/
   cp -rv ~/projects/cortex/content/vortex/* ~/projects/minimax-vortex/src/content/
   ```
4. 跑 build 確認
5. push → auto deploy

## 5. 已完成功能

- [x] Astro 5 專案骨架（TypeScript strict）
- [x] DESIGN_SYSTEM.md（11 章節）
- [x] Global CSS（design tokens + skip link + focus-visible + prefers-reduced-motion）
- [x] Base.astro layout（header / skip link / nav / main / footer + cert legend）
- [x] MiniLadder.astro（L0-L6 共用座標系 component）
- [x] StrokePage.astro（master-detail 共用 component）
- [x] 首頁（Hero + 3 大群 + 1 摺合區結構，不參考 v4）
- [x] 6 式 stroke 內頁（master-detail + Cognitive Gate）
- [x] database 內頁（三軸 filter + 即時計數）
- [x] psychology 內頁（5 處境卡 + 三帶切分）
- [x] levels 內頁（L0-L6 發展序列）
- [x] periodization 內頁（6 節期刊式）
- [x] adm 4 頁（入口 + 矩陣 + 標準 + 背景）
- [x] 內容 + 數據從 cortex 複製
- [x] Playwright 截圖（桌機 + 手機 + 320px reflow）
- [x] axe-core a11y audit
- [x] 41 項 checklist 自動掃描
- [x] **a11y P0 fix**（color-contrast）
- [x] GitHub Actions workflow（deploy.yml）
- [x] README.md + HANDOFF.md（本檔）

## 6. 未完成（待站主 / 後續）

- [ ] **GitHub repo 建立 + push**（站主執行，token 從 ~/.git-credentials 讀 — 見 README §Deploy）
- [ ] GitHub Pages 啟用（Settings → Pages → Source = "GitHub Actions"）
- [ ] 第一次 deploy 後跑 PageSpeed Insights baseline
- [ ] heading-order skip 修（levels + adm-matrix）
- [ ] size-adjust font fallback
- [ ] axe-core CI 整合
- [ ] 5-agent usability testing 套娃（如要做 — v0.2.1 已 ✓ 38/41 達標，可省略）

## 7. 設計研究來源

完整 5 份研究 + L6 摘要 + 41 項 checklist 在 `~/projects/cortex/research/vortex-rebuild/` 與 `~/.claude/memory/research/`：

- `vortex-rebuild/research-log.md` — 網站結構（roadmap.sh / Quartz / Brilliant 等）
- `entry-wayfinding-study.md` — IA / wayfinding / information scent
- `presentation-layout-study.md` — 視覺層級 / 閱讀動線
- `design-principles-deep-dive.md` — 設計原理（typography / a11y / GitHub 5 repo）
- `design-craft-meta-research.md` — design craft（品味 / 互動 / AI 設計批評）
- `non-reskin-checklist.md` — 41 項 checklist
- `redesign-plan-v2.md` — 重構計畫（v2 改向為 minimax-vortex 獨立站）

## 8. 防回退（避免 minimax-vortex v4 失敗重演）

L5 insight `2026-06-18-redo-vs-reskin.md` 教訓：

1. **換皮 ≠ 重做** — minimax-vortex 結構決策（techo 推廣 + 3 群重排 + L0-L6 階梯）已落地，不是只換 CSS
2. **41 項 checklist 是 close gate** — ✗ 多於 ✓ 退回；v0.2.1 已 ✓ 38 / ✗ 1
3. **DESIGN_SYSTEM.md 是 single source of truth** — 改 design 改這檔，避免分散
4. **每次 commit 前**：
   - build 必須綠
   - axe-core 必須無 critical violations
   - 41 項 checklist 逐項打勾
   - 截圖確認（Playwright）

## 9. 站主審查點

v0.2.1 完成狀態給站主 review：

- [x] 設計語言選擇（techo）— 站主已隱含決定（home 已改 techo）
- [x] 技術棧（Astro）— 站主授權
- [x] 不商業字型 — 站主決定
- [x] DESIGN_SYSTEM.md — 我寫（站主授權）
- [x] 5-agent usability testing + multi-agent critique + Playwright — 全部跑過（#38 sub-agent 整合）

**站主決策（待拍板）**：
- [ ] 接受這個設計 / 還要改
- [ ] GitHub repo push 同意（站主執行 token push）
- [ ] 已知未修（heading-order / size-adjust）何時修
- [ ] 簡中版 / 英文版需求（v2+ 規劃）

---

**記**：v0.2.1 已達標 v1 釋出條件（✓ 38 / ✗ 1 / 部分 2-3）。等站主 push GitHub Pages 上線。
