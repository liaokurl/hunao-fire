# 胡鬧消防隊（車干製）

手機可玩的 3D 消防勤務模擬網頁遊戲。玩家扮演消防分隊長：排班、派遣、親自開車滅火與救護，從小型分隊升到大型分隊。使用者是現職消防幹部，用繁體中文溝通，只有平板、沒有電腦。

- 遊玩網址：https://liaokurl.github.io/hunao-fire/ （GitHub Pages，main 分支根目錄）
- 推到 main 就是上線。每次改版都要改版本號並直接推上去，使用者重新整理即可。

## 檔案

- `index.html`：整個遊戲，唯一的原始碼（HTML + CSS + 一段 JS，約一千八百行，很多行很長）。直接改這個檔。
- `three.min.js`：Three.js r128，本機檔，不走 CDN。
- `sw.js`：離線快取。**每次改版要把 `const C="hunao-vX.Y"` 改成新版本號**，不然玩家拿到舊檔。新增圖片檔也要加進 `F` 清單。
- `manifest.webmanifest`、`icon-*.png`：加到主畫面用。
- `cover.jpg` 封面、`avatars.jpg` 3×3 頭像表（第 9 格白頭盔是隊長）、`promo1.jpg`／`promo2.jpg` 升遷圖、`vehicles.jpg` 車庫卡片用的 4×2 車輛圖（順序同 `VORDER`）。圖片由使用者用別的工具產生後提供；需要新圖時寫提示詞給他。3D 模型（車、人、消防衣）全部用程式拼，不需要圖。

## 版本

- JS 開頭 `const VERSION='車干製V2.4'`，畫面左下角會顯示。格式固定為「車干製V主.次」。
- 目前 V2.4。

## 程式結構（index.html 內，依序）

資料表 `STAGES`、`FIRE_T`／`EMS_T`／`SVC_T`（案件）、`VEH`（車輛）、`NOZ`（瞄子）→ 存檔 `Save`、`persist()`、`bump()`、`fixSave()`（舊存檔補欄位都放這裡）→ 雲端 `Cloud` → 音效 `Snd`（全部 Web Audio 合成）→ Three 基礎與 `merge()` → 地圖 `genMap()` → 模型：`SKINS`／`PAINT`／`RAR`、`makePerson()`、`makeVehicle()` → `buildWorld()` → 輸入 → 對話 → 排班 → 分隊倉庫（`renderShop()`／`shopClick()`、皮膚箱 `rollBox()`、今日任務 `quest()`）→ 值勤開始／結束 → 案件（`spawnIncident()`、受困者 `showTrap()`／`rescueTrap()`、現場效果 `fxOf()`／`hitNode()`、支援車 `sendSup()`／`updateSup()`、雲梯 `ladPose()`）→ 派遣卡 → 隊員自動出勤 `updateAI()` → 玩家互動 `getUse()`／`getAct()` → 入室搶救（`enterIndoor()`／`updateIndoor()`／`exitIndoor()`）→ 車流 → 突發狀況 → 日夜與天氣 → 每幀更新 → HUD → 設定 → 主迴圈。

要點：
- 座標：一格 10 單位；前進方向 `(sin h, cos h)`；鏡頭固定從 +z 往 -z 斜俯視。
- 地圖規則：東西向道路固定每 3 格一條，確保每棟房子至少一面臨路。改生成邏輯後要驗證這點。
- 存檔：localStorage `ffcaptain_v1`，並同步到 Firebase（專案 `hunao-fire`，匿名登入 + Firestore `saves/{接續碼}`，用 REST 直接呼叫，沒有載 SDK）。`S.rev` 只在有實際進度時加一，雲端用它判斷新舊。
- 經濟：`S.coin` 獎金（`coin()`）。`S.own` 記擁有的車、瞄子、消防衣、塗裝；`S.bays` 是這個階段的車位編成（`fixBays()` 會修正），`buildWorld()` 依它擺車。車輛實體有 `v.type`（`amb` 或 `VEH` 的鍵）和 `v.M`（規格）。
- 案件旗標：`inc.high`（高處火點，要雲梯）、`inc.chem`（化學火，要泡沫）、`inc.cut`（車禍受困）、`inc.trap`（受困者，`hidden` 表示到場才揭露）、`inc.want`（建議車種）。傷害一律走 `hitNode()`，它會套用這些規則。`inc.fx` 每幀由 `fxOf()` 算出現場有哪些車在幫忙。
- 支援車是 `G.ai` 裡 `sup:true` 的項目，階段 `go → stay → back`，不占隊員。
- 入室搶救的屋內場景搭在地圖外（z = 地圖半徑 + 140），`G.indoor` 存在時 `resolve()` 改用屋內碰撞，`hoseSrc()` 改從屋內門口算。
- 消防衣只是外觀；性能差異只來自車和瞄子（之後做連線時要維持這個原則）。
- 值勤中也能開排班表和分隊倉庫（設定選單，或點 HUD 的獎金）：`inDuty` 為真時這兩個面板關閉後走 `resumeDuty()`，不重建世界；車位與塗裝的更動隔天才生效，消防衣用 `reskin()` 立刻換。
- 全螢幕預設不自動進入（`S.vol.full`），設定選單可開；`goHome()` 回主畫面。
- 連線合作（`RT`、`CO`、`net*`、`coop*`）：Firebase Realtime Database `https://hunao-fire-default-rtdb.asia-southeast1.firebasedatabase.app`，規則只開放 `rooms/$room` 給已登入者。用 REST 寫入、EventSource 串流讀取，沒有載 SDK，沿用存檔的匿名登入（`Cloud.tok()`）。資料：`rooms/{4位數房號}/meta`（房主寫：state lobby/play/end、seed、order、rd 回合數、res 結果）、`p/{pid}`（各自寫：名字、消防衣 sk、塗裝 pt、瞄子 nz、車 veh、心跳 hb、即時狀態 s）、`w`（房主寫：各火點血量、剩餘時間、受困狀態）。房主是主機：火點血量、延燒、勝負都它算；其他人回報自己累計打了多少血（`CO.dmg`，由 `hitNode()` 累加）。開打時 `G.coopCfg` 讓 `buildWorld()` 用共同種子建同一張圖，`G.coop` 為真時值勤迴圈不出案件、不走時鐘，`inc.net` 的案件不在本機結案。玩家帶自己的消防衣、塗裝、瞄子、車進場。分頁切到背景時房主會停擺，是已知限制。
- 連線測試：`localStorage.ff_rtdb` 設成本機位址就會改連那裡且不帶登入，測試用的假伺服器要實作 GET/PUT/PATCH/DELETE 和 `text/event-stream`（put／patch 事件）。每個玩家要開獨立的瀏覽器 context。用 `__ff.update(.05)` 加 `__ff.netTick(.05)` 快轉。
- 地圖風格（`THEMES`：tw 台灣、jp 日本、eg 埃及、db 杜拜）：每種風格有小、中、大三個分隊，`S.theme`＋`S.stage` 決定現在在哪，**一律用 `ST()` 取得目前分隊設定**（它會把 `STAGES[S.stage]` 加上風格的分隊名和 `th`），不要直接讀 `STAGES[S.stage]`。進度在 `S.prog[風格]`：`top` 已開放到哪一級（-1 是整個風格還鎖著）、`xp[]` 各級經驗、`clr[]` 是否通過、`seed[]` 各級地圖種子；`S.xp` 是目前這一級的經驗。`STAGES[k].xp` 是該級的通過門檻（不是累計）。台灣圳頂分隊通過後才開放其他風格。`gotoMap()` 換地圖，多出來的隊員放 `S.bench`。風格影響配色、樹、公園裝飾、招牌、路名、獎金倍率 `cm`。
- 圳頂分隊（台灣小型，`st.zd`）用 `ROADS_ZD` 的真實路名，地址由 `addrOf()` 依路段固定產生。分隊的貓狗有名字（`PETS`、茱蒂）。
- 大量傷病患（`inc.mci`）：`inc.pts` 是所有傷患，`inc.patient` 指向目前處理的那位，`mciNext()` 換下一位；車輛的 `load` 記載了幾人，一台最多 2 人。零星勤務（`JOB`、`inc.job`）是按住完成的通用流程。
- 測試用入口：`window.__ff` 暴露了 G、S、spawnIncident(kind, sub)、update(dt)、openCard、enterIndoor 等。

## 測試方式

用 Playwright 開無頭 Chromium（`/opt/pw-browsers`，加 `--use-gl=swiftshader --enable-unsafe-swiftshader`），手機橫向 844×390，起一個本機靜態伺服器載入 index.html，攔掉 `googleapis` 請求，用 `localStorage` 塞存檔，透過 `window.__ff` 觸發案件，並用 `__ff.update(0.05)` 迴圈快轉（比等真實時間快很多；快轉前把 `G.spawnT`、`G.evT` 設很大，免得隨機案件打斷）。檢查 console 錯誤與截圖。建議每次改完跑一輪「自動派遣連玩三天」的煙霧測試。雲端環境連不到 Google，Firebase 只能用攔截請求的假伺服器測，真連線要請使用者在平板上確認設定頁的「雲端：已同步」。無頭瀏覽器聽不到聲音、量不到實機流暢度，回報時要講明。

## 還沒做／已知問題

- 平衡仍是估的：獎金收入（小型分隊一天約 200、大型約 1000）、車價、火勢血量都沒有實機驗證過。
- 中途關掉遊戲會從當天早上重來（經驗聲望保留）。
- 隊員頭像只有 8 個，大型分隊 14 人會重複。
- 鏡頭固定斜俯視，路南側的高建築會擋住現場。
- 屋內格局只有一種（左右鏡射），商店也用住宅格局。
- 規劃中：連線版（建議用 Firebase Realtime Database、關卡制、房主當主機，先做「大型火災」單一關卡）、宣導模式、音效精進、隊員專長、節慶事件、排行榜、大隊長階段。
