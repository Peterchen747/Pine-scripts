# SNR 3.0 HPA Reaction Model — 使用手冊

> 版本 v4　　適用指標：`SNR_3.0_HPA_Reaction_Model.pine`

---

## 目錄

1. [策略概念](#1-策略概念)
2. [安裝方式](#2-安裝方式)
3. [輸入參數說明](#3-輸入參數說明)
4. [圖表元素說明](#4-圖表元素說明)
5. [資訊表格判讀](#5-資訊表格判讀)
6. [訊號類型與進場邏輯](#6-訊號類型與進場邏輯)
7. [風險管理（SL / TP）](#7-風險管理-sl--tp)
8. [設定 Alert](#8-設定-alert)
9. [常見問題](#9-常見問題)

---

## 1. 策略概念

SNR 3.0 是一個**非預測性**交易系統，由三個條件疊加確認才進場：

```
 Position (HPA)  +  Pressure (Expansion Leg)  +  Confirmation (Reaction & DXY)
     ↓                       ↓                              ↓
 在哪裡反應           有方向性擴張段               AOI 觸及 + DXY 驗證
```

### 核心概念

| 概念 | 說明 |
|---|---|
| **HPA** (High Probability Area) | 趨勢中的 50% pullback 位，或盤整的頂底 25% 區域 |
| **Expansion Leg** | 進入 HPA 之前的方向性推進段，品質需達標（大小 / 連續性 / 動能）|
| **AOI** (Area of Interest) | HPA 內的特定反應位：SNR pivot、擴張段 50%、1-1-1 趨勢線、流動性掃蕩 |
| **DXY Filter** | 美元指數輔助確認（黃金與美元負相關）|

### 兩種進場型態

- **Type 5**：主力型態。HPA + Expansion Leg + AOI 觸及 + DXY + 前一根 bar 突破確認
- **Type 1**：BOS 回撤型態。突破結構後回撤至 BOS 中點，HPA + AOI + DXY 確認

---

## 2. 安裝方式

1. 開啟 TradingView → Pine Editor（下方分頁）
2. 複製 `SNR_3.0_HPA_Reaction_Model.pine` 全文貼入 Editor
3. 點 **Save** → 確認無紅字錯誤訊息
4. 點 **Add to chart**
5. 建議圖表設定：
   - 商品：**XAUUSD**（黃金 / 美元）
   - 時間框架：**M1**（1 分鐘）為主，也可 M5
   - 背景：深色主題（深色底更容易識別 HPA 色塊）

---

## 3. 輸入參數說明

### Structure & Lookback（結構與回溯）

| 參數 | 預設值 | 說明 |
|---|---|---|
| Lookback Hours | 8 | 用於計算視窗高低點的回溯小時數（建議 3-15）|
| Swing Detection Length | 10 | Pivot High / Low 的左右確認 K 線數，越大越少但越可靠 |
| Min Trend Swing % of Range | 15% | 判定趨勢的最小振幅門檻（相對於整個視窗高低差）|

### Expansion Leg（擴張段品質）

| 參數 | 預設值 | 說明 |
|---|---|---|
| Min Bars | 15 | 擴張段最少 K 線數 |
| Max Bars | 30 | 擴張段最多 K 線數 |
| Min Directional % | 80% | 方向 K 線（多 = 陽線）佔比門檻 |
| Min Leg Size (ticks) | 150 | 最小移動幅度，XAUUSD = $1.50 |
| Min Continuity % | 40% | 非重疊 K 線比例（open ≥ 前根 close 為多頭非重疊）|
| Min Momentum % | 60% | 方向 K 線中收盤在有利半段的比例 |

> **調整建議**：初次使用可先保留預設值，若訊號過少可降低 `Min Leg Size` 或 `Min Continuity`。

### Liquidity Grab（流動性掃蕩）

| 參數 | 預設值 | 說明 |
|---|---|---|
| Min Points | 4 | 掃蕩最小點數（太小會有雜訊）|
| Max Points | 15 | 掃蕩最大點數（太大可能是行情突破而非掃蕩）|
| Bars to Confirm Return | 5 | 價格回補確認的最大 K 線數 |

### AOI Detection（AOI 偵測）

| 參數 | 預設值 | 說明 |
|---|---|---|
| AOI Proximity (ticks) | 10 | 價格需在 AOI 位 ±N ticks 範圍內才算觸及，XAUUSD = $0.10 |

> 訊號過少時可調高到 20-30 ticks；訊號雜訊多時調回 5-10 ticks。

### DXY Confirmation（DXY 確認）

| 參數 | 預設值 | 說明 |
|---|---|---|
| Enable DXY Filter | ✓ | 關閉後僅用 HPA + Exp + AOI 判斷，不看 DXY |
| DXY Reaction Lookback | 10 | DXY 尋找極值的回溯 K 線數 |
| DXY Sensitivity % | 0.15% | DXY 需從極值回撤或反彈的最低幅度 |

**DXY 邏輯**：
- 多金訊號：DXY 需從 lookback 高點下跌 ≥ 0.15%，**且** DXY 身處自身視窗頂部 25%（空頭 HPA）
- 空金訊號：DXY 需從 lookback 低點上漲 ≥ 0.15%，**且** DXY 身處自身視窗底部 25%（多頭 HPA）

### Risk Management（風險管理）

| 參數 | 預設值 | 說明 |
|---|---|---|
| Minimum TP (R) | 1.5 | TP 目標最小 R 倍數 |
| Maximum TP (R) | 2.0 | TP 目標最大 R 倍數（實際取兩者平均 1.75R）|
| SL Buffer (ticks) | 50 | SL 在結構位之外的緩衝，XAUUSD = $0.50 |

### Session Filter（時段過濾）

| 參數 | 預設值 | 說明 |
|---|---|---|
| Enable Session Filter | ✓ | 只在倫敦（08:00-17:00 UTC）或紐約（13:00-22:00 UTC）時段發出訊號 |

### Statistics / Multi-TF Reference / Display

以開關為主，控制各視覺元素的顯示與否，依個人偏好設定。

---

## 4. 圖表元素說明

### HPA 色塊

| 顏色 | 含義 |
|---|---|
| 綠色半透明方塊 | Bull HPA（多頭高機率區）|
| 紅色半透明方塊 | Bear HPA（空頭高機率區）|
| 橙色虛線框 | M15 HPA 參考（需開啟 Multi-TF Reference）|
| 黃色虛線框 | H1 HPA 參考 |

> 趨勢模式：HPA 為最後 PL-PH 區間的 50% ±5%
> 盤整模式：HPA 為視窗頂部 25%（空頭）和底部 25%（多頭）

### 擴張段藍色方塊

進入 HPA 的推進段（品質達標後顯示）。方塊高低點對應段內最高/最低價。

### AOI 線

| 線型 | 顏色 | 含義 |
|---|---|---|
| 水平虛線（綠）| 最後 PL 位 | 結構支撐 AOI |
| 水平虛線（紅）| 最後 PH 位 | 結構阻力 AOI |
| 水平點線（藍）| 擴張段 50% | Exp50 回撤 AOI |
| 延伸線（紫，實線）| 1-1-1 趨勢線 | 當前在 HPA 內且價格接近 |
| 延伸線（紫，虛線）| 1-1-1 趨勢線 | 不在 HPA 內或價格偏遠 |

### 訊號圖形

| 圖形 | 位置 | 意義 |
|---|---|---|
| 大綠三角（向上）| K 線下方 | **Type 5 多頭訊號** |
| 大紅三角（向下）| K 線上方 | **Type 5 空頭訊號** |
| 小綠圓點 | K 線下方 | Type 1 多頭訊號 |
| 小紅圓點 | K 線上方 | Type 1 空頭訊號 |
| 紫色菱形 | K 線下/上方 | Type 5 + 趨勢線 AOI 同時確認 |
| `LG` 標籤（水藍）| K 線下方 | 多頭流動性掃蕩 in HPA |
| `LG` 標籤（橙色）| K 線上方 | 空頭流動性掃蕩 in HPA |

### 訊號標籤內容

```
T5 L [SNR+TL]       ← 型態 + 觸及的 AOI 來源
SL: 3245.50         ← 止損價格
TP: 3258.30         ← 止盈目標
R: 1.8              ← 潛在 R 倍數
⚠ SNR↑             ← TP 路徑障礙警告（若有）
```

---

## 5. 資訊表格判讀

右上角表格（16 行）即時顯示市場狀態：

| 欄位 | 說明 | 讀法 |
|---|---|---|
| Market | 市場狀態 | `Bull Trend` / `Bear Trend` / `Range` / `Unclear` |
| Structure | Pivot 結構 | `HH-HL`（多趨勢）/ `LL-LH`（空趨勢）/ `Unclear` |
| Mid × | 盤整中軸穿越次數 | ≥ 2 才算有效盤整（橙色）|
| Session | 當前時段 | `ACTIVE`（倫敦或紐約）/ `CLOSED` |
| Bull Exp | 多頭擴張段品質 | `Q 3/3`（全達標）/ `Q 2/3 !sz`（尺寸不足）/ `NO` |
| Bear Exp | 空頭擴張段品質 | 同上 |
| **Bull AOI** | 多頭 AOI 觸及來源 | `SNR` / `EXP50` / `TL` / `LG` / `—` |
| **Bear AOI** | 空頭 AOI 觸及來源 | 同上 |
| In B.HPA | 價格是否在多頭 HPA | `YES` / `NO` |
| In S.HPA | 價格是否在空頭 HPA | `YES` / `NO` |
| DXY Long | DXY 是否確認多金 | `YES`（DXY 回落 + 在頂部 HPA）/ `NO` |
| DXY Short | DXY 是否確認空金 | `YES`（DXY 反彈 + 在底部 HPA）/ `NO` |
| DXY Zone | DXY 目前所在區域 | `Bear HPA` / `Bull HPA` / `—` |
| TP Block | TP 路徑是否有障礙 | `⚠ SNR↑` / `⚠ MID` / `⚠ TL` / `—` |
| AOI Prox | 目前 AOI 接近門檻設定 | 顯示 ticks 數（提示用）|

**品質標記說明（Exp 欄）**：
- `!sz` = 擴張段尺寸不足（調低 `Min Leg Size` 可放寬）
- `!ct` = 連續性不足（K 線重疊太多）
- `!mo` = 動能不足（方向 K 線收盤位置偏弱）

左上角小表格（統計面板）：

| 欄位 | 說明 |
|---|---|
| Signals | 本次圖表載入後的訊號總數 |
| Wins | 達到 TP 的次數 |
| Win % | 勝率（≥50% 顯示綠色）|
| Avg R | 平均 R 倍數（>0 顯示綠色）|

> 統計僅限當前圖表視窗內的歷史資料，K 線數量越多結果越有參考性。

---

## 6. 訊號類型與進場邏輯

### Type 5 訊號（主力型態）

**所有條件必須同時成立：**

1. 價格在 HPA 區域內（`In B/S.HPA = YES`）
2. 擴張段品質達標（`Q 3/3`）
3. **AOI 觸及**（`Bull/Bear AOI` 顯示來源，不為 `—`）
4. DXY 確認（或關閉 DXY 過濾）
5. 前一根 K 線突破（多頭：`high[1] > high[2]`；空頭：`low[1] < low[2]`）
6. 在倫敦或紐約時段（或關閉時段過濾）

**進場時機**：訊號 K 線收盤後的下一根 K 線開盤進場。

### Type 1 訊號（BOS 回撤型態）

1. 發生 BOS（Break of Structure）：收盤穿越最後 PH（多）或 PL（空）
2. 等待回撤至 BOS 幅度的 **50%** 位置
3. 同時確認：在 HPA + AOI 觸及 + DXY + 時段

**進場時機**：回撤觸及 50% 目標的當根 K 線收盤進場。

### Type 5 + TL 訊號（紫色菱形）

 Type 5 條件全部達標 + 1-1-1 趨勢線同時觸及 HPA，代表**雙重 AOI 確認**，優先級最高。

### AOI 來源解讀

| 標記 | 意涵 | 可信度 |
|---|---|---|
| `SNR` | 舊支撐/阻力翻轉，水平 S/R 位 | 高 |
| `EXP50` | 擴張段 50% 回撤，動能回測 | 中高 |
| `TL` | 趨勢線支撐/壓力，動態 AOI | 高（需結合 `in_hpa`）|
| `LG` | 流動性掃蕩完成，止損掃除後反轉 | 中高 |
| `SNR+TL` | 多重 AOI 疊加 | 最高 |

---

## 7. 風險管理（SL / TP）

### SL 計算

| 方向 | SL 位置 |
|---|---|
| 多單 | 最後 PL（結構低點）之下 `sl_buffer_ticks` ticks |
| 空單 | 最後 PH（結構高點）之上 `sl_buffer_ticks` ticks |

> XAUUSD 預設 50 ticks = $0.50 緩衝，可依波動率調整。

### TP 計算

以 **1.75R**（min 1.5R 和 max 2.0R 的平均）為目標，若**擴張段 50% 中點**恰好落在目標附近（比 R 目標更近）則優先取 Exp50。

### TP 路徑障礙（`TP Block`）

下列情況會觸發 ⚠ 警告：
- `SNR↑` / `SNR↓`：最後 pivot 位落在 Entry 和 TP 之間
- `MID`：盤整中軸落在 Entry 和 TP 之間
- `TL`：趨勢線值落在 Entry 和 TP 之間

**遇到 TP 路徑障礙時的處理方式**：
1. 縮小 TP 至障礙位之前（手動調整）
2. 若 R 值縮水到 1R 以下，考慮略過此次訊號
3. 障礙為 MID 時，可等待 MID 被突破再評估

---

## 8. 設定 Alert

在 TradingView 右上角點「Alert」→「Create alert」：

| Alert 名稱 | 觸發條件 |
|---|---|
| `SNR3 Type5 Long` | Type 5 多頭（全條件達標）|
| `SNR3 Type5 Short` | Type 5 空頭（全條件達標）|
| `SNR3 Type5 Long + TL AOI` | Type 5 + 趨勢線 AOI 雙確認多頭 |
| `SNR3 Type5 Short + TL AOI` | Type 5 + 趨勢線 AOI 雙確認空頭 |
| `SNR3 Type1 Long` | Type 1 BOS 回撤多頭 |
| `SNR3 Type1 Short` | Type 1 BOS 回撤空頭 |
| `SNR3 Bull Liq Grab in HPA` | 多頭流動性掃蕩（in HPA + Exp + DXY）|
| `SNR3 Bear Liq Grab in HPA` | 空頭流動性掃蕩 |
| `SNR3 TP Obstacle (Long)` | 多頭訊號但 TP 路徑有障礙 |
| `SNR3 TP Obstacle (Short)` | 空頭訊號但 TP 路徑有障礙 |

**建議 Alert 頻率**：設為「Once per bar close」，避免 bar 內重複觸發。

---

## 9. 常見問題

**Q：訊號很少，甚至沒有訊號？**
- 先看右上角表格：`Market` 是否顯示 `Unclear`（需等更多 Pivot 形成）
- `Bull/Bear Exp` 是否顯示 `NO`（調低擴張段門檻）
- `Bull/Bear AOI` 是否顯示 `—`（調高 `AOI Proximity` 到 20-30 ticks）
- `Session` 是否顯示 `CLOSED`（非倫敦/紐約時段）

**Q：訊號太多，想過濾？**
- 調低 `AOI Proximity`（例如 5 ticks，要求更精確觸及）
- 調高 `Min Leg Size` 或 `Min Continuity`
- 確認 DXY Filter 已開啟

**Q：表格 `Structure = Unclear` 但看起來是趨勢？**
- Pivot 可能在 lookback 窗口外，增加 `Lookback Hours`
- 或 Pivot 振幅未達 `Min Trend Swing %`，調低此值

**Q：DXY 資料顯示 N/A 或 DXY Long/Short 永遠是 NO？**
- TradingView 免費帳號可能無法取得 DXY 資料，可暫時關閉 `Enable DXY Filter`
- 確認圖表商品是 XAUUSD（DXY 對黃金以外的商品意義有限）

**Q：TP 標示價格看起來不合理？**
- 確認 `SL Buffer` 設定符合市場波動性
- 若商品不是 XAUUSD，`Min Leg Size (ticks)` 和 `SL Buffer` 需按比例調整

**Q：1-1-1 趨勢線是紫色實線還是虛線？**
- 實線 = 趨勢線當前在 HPA 內且價格接近（≤ `AOI Proximity` ticks）
- 虛線 = 趨勢線在 HPA 外，或價格距離過遠（僅作視覺參考）

---

## 附錄：訊號確認清單

進場前快速核對以下 6 個條件：

```
□ 1. Market = Bull/Bear Trend 或 Range（非 Unclear）
□ 2. 對應方向 Exp = Q 3/3
□ 3. 對應方向 AOI ≠ —（有觸及 AOI）
□ 4. In B/S.HPA = YES
□ 5. DXY Long/Short = YES（或 DXY Filter 關閉）
□ 6. Session = ACTIVE（或 Session Filter 關閉）
□ 7. TP Block = — （無障礙，或確認可接受障礙）
```

全部 ✓ → 訊號有效，依標籤上的 SL/TP 進場。

---

*最後更新：v4 | 2026-06*
