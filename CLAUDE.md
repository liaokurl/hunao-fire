# 胡鬧消防隊（車干製）

手機可玩的 3D 消防勤務模擬網頁遊戲。玩家扮演消防分隊長：排班、派遣、親自開車滅火與救護，從小型分隊升到大型分隊。使用者是現職消防幹部，用繁體中文溝通，只有平板、沒有電腦。

- 遊玩網址：https://liaokurl.github.io/hunao-fire/ （GitHub Pages，main 分支根目錄）
- 推到 main 就是上線。每次改版都要改版本號並直接推上去，使用者重新整理即可。

## 檔案

- `index.html`：整個遊戲，唯一的原始碼（HTML + CSS + 一段 JS，約兩千行）。直接改這個檔。
- `three.min.js`：Three.js r128，本機檔，不走 CDN。
- `sw.js`：離線快取。**每次改版要把 `const C="hunao-vX.Y"` 改成新版本號**，不然玩家拿到舊檔。
- `manifest.webmanifest`、`icon-*.png`：加到主畫面用。
- `cover.jpg` 封面、`avatars.jpg` 3×3 頭像表（第 9 格白頭盔是隊長）、`promo1.jpg`／`promo2.jpg` 升遷圖。圖片由使用者用別的工具產生後提供；需要新圖時寫提示詞給他。

## 版本

- JS 開頭 `const VERSION='車干製V1.7'`，畫面左下角會顯示。格式固定為「車干製V主.次」。
- 目前 V1.7。

## 程式結構（index.html 內，依序）

資料表 `STAGES`（三個分隊階段，大隊長／局長尚未製作）→ 存檔 `Save`、`persist()`、`bump()` → 雲端 `Cloud` → 音效 `Snd`（全部 Web Audio 合成）→ Three 基礎與 `merge()`（把小零件合成一個模型）→ 地圖 `genMap()` → 模型 → `buildWorld()` → 輸入 → 對話 → 排班 → 值勤開始／結束 → 案件 → 派遣卡 → 隊員自動出勤 → 玩家互動 `getUse()`／`getAct()` → 車流 → 突發狀況 → 日夜與天氣 → 每幀更新 → HUD → 設定 → 主迴圈。

要點：
- 座標：一格 10 單位；前進方向 `(sin h, cos h)`；鏡頭固定從 +z 往 -z 斜俯視。
- 地圖規則：東西向道路固定每 3 格一條，確保每棟房子至少一面臨路。改生成邏輯後要驗證這點。
- 存檔：localStorage `ffcaptain_v1`，並同步到 Firebase（專案 `hunao-fire`，匿名登入 + Firestore `saves/{接續碼}`，用 REST 直接呼叫，沒有載 SDK）。`S.rev` 只在有實際進度時加一，雲端用它判斷新舊。
- 測試用入口：`window.__ff` 暴露了 G、S、spawnIncident、spawnEvent 等。

## 測試方式

用 Playwright 開無頭 Chromium（`/opt/pw-browsers`，加 `--use-gl=swiftshader --enable-unsafe-swiftshader`），手機橫向 844×390，起一個本機靜態伺服器載入 index.html，透過 `window.__ff` 觸發案件並檢查 console 錯誤與截圖。雲端環境連不到 Google，Firebase 只能用攔截請求的假伺服器測，真連線要請使用者在平板上確認設定頁的「雲端：已同步」。無頭瀏覽器聽不到聲音、量不到實機流暢度，回報時要講明。

## 還沒做／已知問題

- 平衡未調：任務太容易拿五星；蛇常常一下車就抓到。
- 中途關掉遊戲會從當天早上重來（經驗聲望保留）。
- 隊員頭像只有 8 個，大型分隊 14 人會重複。
- 使用者想要的方向：隊員養成做深（專長）、節慶事件、更多車種（雲梯車）、排行榜、馬路再熱鬧一點。
