# 馬會亞指推薦板

靜態頁：`index.html`（內嵌最新 JSON）＋ `tickets.json`。

公開站：https://pages/ah-05cf743473/

## 內容（觀眾視圖）

兩個分頁，**不以 T-45／T-30／T-15 分窗呈現**（出票窗只供自動化抓莊家走勢；網站觀眾不需要三窗結構）：

1. **即將／未來 12h**  
   - `automation/upcoming.json` 中未來 12 小時（HKT）、`has_hkjc_ah=true` 的馬會讓球  
   - 每場**一張**分析卡：永遠用**最新一次**已出分析覆蓋（按寫入時間）；T-45 一出即顯示，其後 T-30／T-15 再覆蓋——**不必等 T-15**（歷史 T-20／T-10 仍可覆蓋）  
   - 尚未出票 →「待出票」／只顯示賽程與馬會線  
   - 顯示：【主方向·鳴人／鹿丸】（標明來源；歷史 fade 票可仍見 fade 字樣）、【鳴人】、旁註 chips（含顯示向【支線·clean】／【支線·聯賽】；不驅動主方向）、上／下盤、馬會 line  

2. **歷史／覆盤**  
   - 已出票且開賽時間已過的場次（按場分組）  
   - 主視圖：最新結論 + **覆盤**  
   - 「分析歷程」可展開：同場各窗快照按時間列出（僅淡標記「賽前約45／30／15分」；歷史可含 20／10，不作 UI 主結構）  
   - 覆盤欄位（有賽果時）：FT `score`（hg-ag）、主方向（標來源）全贏／半贏／走水／半輸／全輸；鳴人可解析時一併顯示  
   - 無賽果：**覆盤：待賽果**（從不捏造比分）

## 公開顯示原則

- 顯示：**主方向**、**平行參考**、馬會**盤口／水位變化**說明（初→現）
- **不展示**模型內部指標與操作細節（例如 conf%、樣本 n、TEAR／K-FIX、水箱規則等）
- 旁註若顯示，只保留極短方向／狀態，不含方法說明

## 主方向鎖

- **LIVE 2026-09-27 02:41 HKT：鳴人-first@40 · NO fade**（`tickets/LINE_ORDER.md`）
- **預設主方向＝鳴人**
- 鳴人 conf **<40%** → **一律**跟鹿丸（**即使同向**）
- **停用反投**：同向時不再 fade
- **已廢止：** 預設鹿丸 + 鳴人反向 conf>55% 覆蓋；conf<40 同向 fade
- 盤口只顯示來源實數（upcoming／票據「馬會：」／done.json）；**從不捏造**

## 賽果來源

`automation/scores.json`（由 `scripts/fetch_scores_for_board.py` 寫入）：

- 主來源：`https://livestatic.titan007.com/phone/txt/analysisheader/cn/{id[0]}/{id[1:3]}/{id}.txt`  
  - 僅當 `state=-1`（完場）才寫入 FT 比分  
- 結算數學：`backtest/settle.py`（`ah_result`／`settle_ticket`），角色／線來自 `done.json`（`role`、`line`／`hkjc_close.line`）與票面讓／受  
- 備援：WebFetch `https://live.titan007.com/detail/{matchId}sb.htm`（box 直連 vip／部分頁可能 TLS／WAF 失敗時）

## 刷新節奏（2026-09-27）

**主路徑（近 KO／覆盤）：出票即推**

```bash
# T-45／T-30／T-15／更正票寫入後必跑（抓 FT → 重建 → push）
./scripts/after_window_issue.sh --reason "T15 issued …"
```

**兜底：** `@every 2h`（或 cron `0 */2 * * *`）仍跑 `./scripts/rebuild_public_board.sh --push`。  
每次重建**預設**先跑 `fetch_scores_for_board.py`，完場場次會盡快出現覆盤（全贏／半贏／…），唔好再等下一個 2h。

```bash
./scripts/rebuild_public_board.sh          # scores + build（本地）
./scripts/rebuild_public_board.sh --push   # scores + build + Pages/Surge
```

倒數徽章改由瀏覽器按 `kickoff_iso` 即時計算（避免靜態板把 T−4m 凍住）。  
本腳本**不**負責抓盤／出票；只把已有 upcoming／tickets／scores 編成公開板。

## GitHub Pages

- Repo：隨機名私隱路徑（見 Pages URL）
- Pages source：`main` 分支根目錄 `/`
- URL：https://pages/ah-05cf743473/
- 本 pack 產出在 `public/`（含 `.nojekyll`）；`--push` 會把 `public/` 同步到該 repo 根目錄

## 內容原則

- 只顯示結論（主方向·鹿丸／鳴人／旁註線）與馬會 path／線、覆盤結果
- **不含**完整票據原文
- 資訊參考、非投注建議
