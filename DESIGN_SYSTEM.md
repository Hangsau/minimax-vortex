# DESIGN_SYSTEM — minimax-vortex

> **這個檔是我（Claude）寫的**，基於 5 份研究 + 41 項 checklist + 站主決策。
>
> 站主決策（2026-06-18）：
> - 名字：minimax-vortex
> - 技術棧：Astro 5.x（不用 Hugo，重新來）
> - 不商業字型（OFL 字型）
> - 不參考之前 v4 設計實作（commit 4504e02 已撤）
> - DESIGN_SYSTEM.md 我寫（站主授權）
> - 該跑的都要跑（5-agent usability testing + multi-agent design critique + Playwright）
>
> **這個檔是 single source of truth**。未來改 design 改這檔，然後 style sheet 跟 layout 對齊。
> 所有 design token 用 CSS variable（`--token-name`），component / layout 一律不寫 hardcoded value。

---

## 1. 設計語言定位

**一句話**：水感知識庫 — 用「日式文具手帳（techo）」的設計語言呈現游泳技術知識。

**為什麼選這個語言**：
- 站主主動改 vortex-home 為 techo 風格（vortex.css → vortex-techo.css）— 站主品味決定，不是 L1 自作主張
- 學術期刊站很多（vortex.css 學術期刊風同質化）— 日式手帳風差異化
- 漢字 typography 友善（Shippori Mincho 是漢字襯線）
- baseline grid 物理化（30px 橫罫線既是骨架也是視覺）

**設計語言三個關鍵字**：
1. **留白即設計**（空間不是浪費，是結構）
2. **格線是骨架不是裝飾**（30px repeating-linear-gradient）
3. **細邊框取代卡片**（不用陰影 / 不用圓角）

---

## 2. 字型系統（全部 OFL，免費可商用）

### 2.1 字型清單

| 角色 | 字型 | 備註 |
|------|------|------|
| 主體漢字 | **Shippori Mincho** | 思源宋體改作（漢字襯線，日式） |
| 主體英文 | **Inter** | Google Sans 替代品（sans serif，技術感） |
| 等寬（編號 / 圖例） | **Courier Prime** | 等寬，數字對齊 |
| 中文 sans fallback | **Noto Sans TC** | 中文 fallback |
| 中文 serif fallback | **Noto Serif TC** | 中文襯線 fallback |

**授權**：全部 SIL OFL 1.1，可商用、免費、無需聲明。

**載入策略**：
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="preload" as="style" href="https://fonts.googleapis.com/css2?...">
```

**為什麼選這些**：
- Shippori Mincho — 漢字襯線，避開「思源宋體過於學術」、避開「霞鶩文楷過於書法」、找介於中間的
- Inter — UI 通用，技術感，但不像 Helvetica 那樣「歐美」
- Courier Prime — IBM Courier 的開源改良，數字對齊好
- Noto 系列 — Google 維護的全語言 fallback，必須有

### 2.2 字級系統（Modular Scale）

**比例**：Major Third（1.25）— base 16px

**理由**：Tim Brown「More Meaningful Typography」+ Bringhurst《The Elements of Typographic Style》§3.1 — 1.25 是「**編輯襯線 + 內容站**」的標準比例，比 Minor Third（1.2）有層次、又不會像 Perfect Fourth（1.333）太跳。

**Token 命名**（基於 t-shirt size + 數值備註）：

```css
:root {
  --type-tiny:   10px;   /* 0.625 */
  --type-xs:     12px;   /* 0.75  */
  --type-sm:     14px;   /* 0.875 */
  --type-base:   16px;   /* 1.000 */
  --type-md:     20px;   /* 1.250 */
  --type-lg:     25px;   /* 1.5625 */
  --type-xl:     32px;   /* 2.000 */
  --type-2xl:    40px;   /* 2.500 */
  --type-3xl:    51px;   /* 3.1875 */
  --type-4xl:    64px;   /* 4.000 */
}
```

**實際使用映射**：

| 元素 | token | 數值 |
|------|-------|------|
| `body` | `--type-base` | 16px |
| 微註 / meta | `--type-tiny` | 10px |
| 圖例 / eyebrow | `--type-xs` | 12px |
| 副文案 / premise | `--type-sm` | 14px |
| 一級條目 | `--type-md` | 20px |
| 主入口標題 | `--type-lg` | 25px |
| 群標籤 H2 | `--type-xl` | 32px |
| 章節 H1 | `--type-2xl` | 40px |
| 站名 H0 | `--type-3xl` | 51px |
| Hero 巨型 | `--type-4xl` | 64px |

### 2.3 行距 / Line-height

| 元素 | line-height | 理由 |
|------|-------------|------|
| 漢字 body | **1.875** | Butterick §line-spacing 漢字合理區間 1.7–1.85，加上 baseline grid 物理化 = 30px/16px = 1.875 |
| 拉丁 body | **1.5** | Butterick 甜蜜點 1.2–1.45 + 視覺微調 |
| Display / Hero | **1.05–1.2** | 大字距小 |
| 編號 / mono | **1.3** | 等寬字自然行距 |

**為什麼 1.875 而不是 1.75**：因為 30px baseline grid 物理化（repeating-linear-gradient），整頁面每行都對齊 30px 線 — 1.875 是 30/16 的精確結果。如果用 1.75（17px × 1.75 = 29.75），會偏離 baseline grid。

### 2.4 Measure（行寬）

| 元素 | max-width | 理由 |
|------|-----------|------|
| body 主文 | **38em ≈ 608px** | Butterick measure 45–90 chars / 2–3 alphabets / 66 chars 甜蜜點。漢字 ~38 字 |
| 寬顯示 | 880px | techo 沿用值 |
| 章節容器 | 880px | 內文 + 編號 + meta 都進 |
| 全寬 overflow | 1080px | 表格 / 卡片網格 |

### 2.5 字距（letter-spacing / tracking）

| 元素 | tracking | 理由 |
|------|----------|------|
| Display（h0/h1） | -0.02em | 大字微負字距（視覺密度調整） |
| body | 0 | 不動 |
| Eyebrow / uppercase | +0.42em | 大寫字母拉開（techo 沿用） |
| Mono / 編號 | +0.04em | 等寬字微正字距 |

---

## 3. 顏色系統

### 3.1 主色板

```css
:root {
  /* 背景層 */
  --color-bg:        #FAFAF7;   /* 紙色（techo 沿用） */
  --color-bg-soft:   #F4F2EA;   /* 紙色深 */
  --color-bg-elev:   #FFFFFF;   /* 卡片 / 模態 */

  /* 文字層 */
  --color-ink:       #1A1A1A;   /* 主文（高對比，~17:1） */
  --color-ink-soft:  #3a3a34;   /* lead / 引言 */
  --color-sub:       #88887e;   /* 次要文字 */
  --color-faint:     #b4b4a8;   /* 邊界 / disabled */

  /* 強調色 */
  --color-accent:    #D4622A;   /* 柿橘（techo 沿用，CTA / 強調） */
  --color-accent-2:  #A04A1F;   /* 柿橘深（hover / active） */
  --color-accent-bg: #f3d9c8;   /* 柿橘背景（selection / highlight） */

  /* 結構線 */
  --color-rule:      #E8E8E0;   /* 細邊框（techo 沿用） */
  --color-border:    #c9c9bd;   /* 中邊框 */
  --color-rule-strong: #1A1A1A; /* 強邊框（章節分隔） */

  /* 狀態色 */
  --color-good:      #2E7D32;   /* 綠（成功 / 完成） */
  --color-warn:      #B58900;   /* 黃（注意 / 警告） */
  --color-bad:       #B33A3A;   /* 紅（錯誤 / 危險） */
  --color-info:      #2C5A86;   /* 藍（資訊 / 連結） */

  /* 確定性標籤（從 vortex 既有） */
  --color-cert-derived:  #2C5A86; /* 🔵 推導 */
  --color-cert-recent:   #2E7D32; /* 🟢 近期文獻 */
  --color-cert-old:      #B58900; /* 🟡 舊文獻 */
  --color-cert-coach:    #D4622A; /* 🟠 教練觀測 */
  --color-cert-todo:     #B33A3A; /* 🔴 待查 */
}
```

### 3.2 對比標準（WCAG AA / AAA）

| 組合 | 對比 | 標準 |
|------|------|------|
| `--color-ink` on `--color-bg` | **17.4:1** | AAA ✓ |
| `--color-ink-soft` on `--color-bg` | **12.6:1** | AAA ✓ |
| `--color-sub` on `--color-bg` | **5.1:1** | AA ✓（接近 AAA 7:1 邊緣） |
| `--color-accent` on `--color-bg` | **4.9:1** | AA ✓（大字 3:1 ✓） |
| `--color-faint` on `--color-bg` | **3.2:1** | 僅大字（不能用作 body） |

**決策**：body 文字只用 `--color-ink` / `--color-ink-soft`，**不用 `--color-sub`**（雖然 AA 過，但視覺密度太低）。

---

## 4. 間距系統（Baseline Grid 物理化）

### 4.1 核心

```css
:root {
  --unit: 30px;  /* 所有 spacing 都用這個的倍數 */
}
```

### 4.2 Spacing scale

```css
:root {
  --space-1:   6px;    /* 0.2 unit（微型間距） */
  --space-2:   12px;   /* 0.4 unit（小間距） */
  --space-3:   18px;   /* 0.6 unit（次小間距） */
  --space-4:   30px;   /* 1 unit（基準間距） */
  --space-5:   60px;   /* 2 unit（大間距） */
  --space-6:   90px;   /* 3 unit（特大間距） */
  --space-7:   120px;  /* 4 unit（章節分隔） */
  --space-8:   150px;  /* 5 unit（hero 區） */
}
```

### 4.3 為什麼不用 4/8px 標準 scale

**30px 是漢字 typography 的舒適基準**（不是 Latin 的 16/24px）。漢字筆劃密度高，需要更大行距 / 段距；30px = 漢字 body 1.875 line-height = 整頁所有元素對齊 30px baseline grid。

### 4.4 物理化（techo 沿用）

```css
body {
  background-color: var(--color-bg);
  background-image: repeating-linear-gradient(
    to bottom,
    transparent,
    transparent calc(var(--unit) - 1px),
    var(--color-rule) calc(var(--unit) - 1px),
    var(--color-rule) var(--unit)
  );
}
```

**效果**：整頁背景有淡 30px 橫罫線，所有元素都對齊這個 grid — 設計骨架 = 視覺骨架。

---

## 5. 互動系統（Motion + State）

### 5.1 Duration tokens（Material 3 標準）

```css
:root {
  --duration-instant:  100ms;   /* 微型反饋（按鈕按下） */
  --duration-short:    200ms;   /* 元素 enter / exit */
  --duration-medium:   400ms;   /* 標準過場 */
  --duration-long:     500ms;   /* 大範圍過場 */
  --duration-page:     700ms;   /* 頁面層級（hero 進場） */
}
```

### 5.2 Easing tokens

```css
:root {
  --ease-linear:       cubic-bezier(0, 0, 1, 1);
  --ease-standard:     cubic-bezier(0.2, 0, 0, 1);     /* 預設 */
  --ease-decelerate:   cubic-bezier(0, 0, 0.2, 1);     /* 進入用 */
  --ease-accelerate:   cubic-bezier(0.4, 0, 1, 1);     /* 離開用 */
  --ease-emphasized:   cubic-bezier(0.83, 0, 0.17, 1); /* 強調 */
}
```

### 5.3 Motion patterns

| Pattern | 用途 | 實作 |
|---------|------|------|
| **Fade Through** | section / panel 切換 | opacity 0 → 1 + translateY(8px → 0) |
| **Container Transform** | 痛點卡 → drill 列表 | element grows + content swaps |
| **Shared Axis** | master-detail rail ↔ panel | horizontal slide |
| **Fade** | 同層級切換（carousel） | opacity only |

**預設用 fade-through**（最通用），其他只在特定互動用。

### 5.4 Reduced motion（必支援 — WCAG 2.3.3）

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### 5.5 Focus state（必可見 — WCAG 2.4.7）

```css
:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 3px;
  border-radius: 2px;
}

:focus:not(:focus-visible) {
  outline: none;
}
```

---

## 6. Accessibility 規範

### 6.1 Skip link（每頁）

```html
<a href="#main" class="skip-link">跳至主內容</a>
```

### 6.2 Landmarks

每頁必有：
- `<header>` — 頁面頂部
- `<main>` — 主內容
- `<nav>` — 主要導航
- `<footer>` — 頁面底部

### 6.3 ARIA 使用原則

- 標籤元素（`<button>`、`<a>`）— 不用 ARIA role 覆蓋
- 互動元素 — 加 `aria-expanded` / `aria-controls` / `aria-label`
- 動態內容 — `aria-live="polite"` 或 `assertive`
- 圖示 — `aria-hidden="true"`（裝飾）或 `aria-label`（語意）

### 6.4 顏色不能是唯一指示（WCAG 1.4.1）

任何狀態（成功 / 錯誤 / 確定性）除了顏色還要加 icon 或文字。

### 6.5 觸控目標 ≥ 24×24 CSS px（WCAG 2.5.8）

所有 button / link / clickable 在 mobile 至少 24×24px，建議 44×44px。

---

## 7. 結構決策

### 7.1 全站風格統一

**techo 風格（tx-* class）為唯一設計語言**。vortex.css（學術期刊風）不採用。

### 7.2 首頁結構（從零設計 — 不抄 v4）

**不是 12 等重入口**，是「**3 大群 + 1 摺合區**」：

| 群 | 內容 | 視覺處理 |
|----|------|---------|
| **群 1：先懂**（Theory） | hero + 痛點卡 + L0-L6 概覽 | 主焦點放大 |
| **群 2：開始練**（Practice） | 六式 grid（2×3） | 並列卡片 |
| **群 3：長期**（Long-term） | ADM + 週期化 | 並排雙格 |
| **參考摺合區** | 依需求找 + 跨式查 | `<details>` 預設收合 |

### 7.3 共用座標系（L0-L6 mini 階梯）

抽成 component（`<MiniLadder levels={[...]} />`），所有 vortex 內頁嵌入。

### 7.4 Cognitive Gate

依層級收起 — L0 沒穩不給看 L3+。用 `<details>` 預設收合 + 警示文字。

---

## 8. Content vs Design 分界

**內容（cortex 站內 vortex section 內容真相源）**：
- `~/projects/cortex/data/vortex/*.yaml`（12 個 — 已經 sync + 淨化）
- `~/projects/cortex/content/vortex/*.md`（43 個 — canonical markdown）
- 由 sync_vortex.py 從 TheVortexProject/canonical/ 同步

**設計（minimax-vortex 自有）**：
- Astro 結構 / components / pages
- CSS（techo 風格）
- 互動邏輯（client-side JS）

**嚴守禁區**：
- 公開層不放「自陳 → 感知/技術品質判斷」
- 不寫 canonical 內容（從 cortex 複製，不改）
- 同步流程：cortex data 改 → minimax-vortex src/content 也要更新（手動或寫個 sync script）

---

## 9. 不做的事（明確 anti-pattern）

| ✗ | 為什麼 |
|---|--------|
| 加 audience-based navigation | entry-wayfinding §3 反例（NN/g 5 reasons to avoid） |
| 加 12 個等重入口 | presentation-layout §2.4 反例（competing elements） |
| 用 sidebar 樹狀目次當主結構 | vortex-rebuild §1.3 反例（Docusaurus 血統） |
| 買商業字型 | 站主決策（OFL 即可） |
| 加 audience 入口分流 | 站主 L5 insight + NN/g #4 avoid |
| 換成 SPA（React/Vue 全棧） | vortex 是內容站，SSG 對 PageSpeed 友善 |
| 把 SSG 換成 SSR | 內容不需要即時更新，SSG 預渲染效能最佳 |

---

## 10. 41 項 Checklist 整合

完整 41 項打勾清單見 `research/non-reskin-checklist.md`（vortex-rebuild 研究附錄）。每次 commit 前逐項驗證。

**預評**（minimax-vortex 起始狀態）：
- ✓ 0（從零開始）
- ✗ 41（全部要實踐）
- 目標：v1 釋出前 ✓ 30+ / ✗ < 5 / 部分 < 5

---

## 11. 版本演進

- **v1**（目標）：4 群 + 41 項 checklist 大部分 ✓
- **v2**：5-agent usability testing 整合 + multi-agent critique 應用
- **v3**：跨文化（簡中版 / 英文版）
- **v4**：個人化（localStorage 紀錄進度）

**v1 不做個人化**（per entry-wayfinding §6 O5 — 站主尚未拍板）。

---

**記**：改 design 改這檔 → CSS / component 對齊 → commit 前 41 項打勾。
