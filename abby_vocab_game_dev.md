# Abby 單字練習遊戲｜開發文件

> 最後更新：2026-07-15  
> 目前版本：v6（HTML 離線版，含聽力測驗、口說練習、批次新增）  
> 程式碼檔案：`abby_vocab_game_v6.html`（單一檔案，可直接分享）

---

## 一、專案概述

為英文學習者 Abby（CEFR A1-A2，目標 B1+）設計的單字練習遊戲。  
單一 HTML 檔案，無需安裝，瀏覽器直接開啟即可使用。

### 核心設計原則

- 所有單字都有 **4 組不同例句**，每次隨機抽取，避免背句子
- **自適應出題**：依答題紀錄自動調整每個單字的出現頻率
- **即時解析**：答錯立刻顯示正確答案、中文意思、正確例句
- **Web Speech API**：瀏覽器內建語音，點按鈕即可聽發音（Chrome / Safari）
- **localStorage** 儲存所有學習紀錄，重開不遺失
- **匯出 / 匯入 JSON**：跨裝置搬移學習紀錄

---

## 二、功能清單

| 功能 | 狀態 | 說明 |
|------|------|------|
| 填空遊戲 | ✅ | 看句子選正確單字（4 選 1） |
| 聽力測驗・單字 | ✅ | 聽英文單字發音，選出正確單字 |
| 聽力測驗・句子填空 | ✅ | 聽整句英文，選出空格單字（可按「看句子」提示） |
| 對錯判斷 | ✅ | 判斷句子描述是否正確 |
| 拼字挑戰 | ✅ | 看中文拼出英文，可用提示（-0.5分） |
| 配對遊戲 | ✅ | 英文配中文，每組 6 對，共 4 組 |
| 聽發音 | ✅ | 佇列式 TTS，可連續念多段 |
| 換題複誦 | ✅ | 填空／聽力答完念「單字 ×N ＋ 完整句子 ×N」，N 可設 0/1/2/3 |
| 錯題手動換頁 | ✅ | 答錯停在解析畫面等按「下一題」；答對念完自動換 |
| 錯題解析 | ✅ | 答錯顯示：正確答案＋中文＋例句＋注意事項 |
| 自適應出題 | ✅ | 依 weight 機制控制出題頻率 |
| 出題範圍篩選 | ✅ | 全部 / 需加強 / 新單字 / 最近新增 |
| 手動新增單字 | ✅ | 一次一個，含確認預覽 |
| 批次新增單字 | ✅ | 一次多個，逐行檢查並列出問題 |
| 新增防呆 | ✅ | 重複、中英填反、含數字、拼錯相近字、例句沒挖空→自動修正 |
| 新增後復原 | ✅ | 加錯可一鍵移除 |
| 刪除自訂單字（單筆） | ✅ | 列表右側「刪除」鈕，僅自訂單字有 |
| 刪除自訂單字（全部） | ✅ | 「清空我自己新增的單字」，內建單字與練習紀錄都保留 |
| 單字搜尋 | ✅ | 支援英文、中文搜尋 |
| 單字篩選 | ✅ | 全部 / 需加強 / 已熟練 / 自己新增 |
| 學習統計 | ✅ | 總字數、熟練數、需加強數、練習次數、答對數 |
| 需加強列表 | ✅ | 自動列出錯多於對的單字 |
| 最近錯誤 | ✅ | 最近 30 筆答錯紀錄（含日期） |
| 匯出 JSON | ✅ | 含統計、自訂單字、錯誤紀錄 |
| 匯入 JSON・只匯入單字 | ✅ | 只加單字，**練習紀錄完全不動**（別人分享單字給你時用）|
| 匯入 JSON・連紀錄一起 | ✅ | 會覆蓋練習紀錄，有二次確認（自己換裝置搬進度時用）|
| 重置統計 | ✅ | 只重置答題資料，單字不刪 |
| 口說練習 | ✅ | Web Speech Recognition，念單字／念句子，逐字比對給分 |
| 即時 AI 生成例句 | 🔜 | 下一版用 Claude API 實作 |
| 跨裝置即時同步 | 🔜 | 下一版改 Artifact + window.storage |

---

## 三、自適應出題機制（Weight System）

```javascript
function getWeight(word) {
  var s = getStats(word);
  if (s.correct >= 3 && s.streak >= 3) return 0.15;  // 已熟練 → 低頻出現
  if (s.wrong > s.correct && s.wrong > 0) return 2.5; // 需加強 → 高頻出現
  if (s.correct === 0 && s.wrong === 0)  return 1.5;  // 新單字 → 稍微優先
  return 1;                                            // 一般
}
```

**判斷邏輯：**
- 熟練（mastered）= `correct >= 3` 且 `streak >= 3`
- 需加強（weak）= `wrong > correct` 且 `wrong > 0`

**weighted 抽樣：** 每個 weight 乘以 10 決定複製幾份進抽籤池，再隨機抽取不重複樣本。

---

## 四、資料結構

### DB（localStorage key: `abby_vocab_v5`）

```javascript
{
  stats: {
    "reach": { correct: 5, wrong: 1, streak: 3, lastSeen: 1720000000000 },
    // ...每個單字的答題紀錄
  },
  custom: [
    {
      e: "beautiful",
      zh: "美麗的",
      pos: "adj",
      hint: "be__ 開頭，9個字母",
      sentences: ["She is a ___ girl.", "What a ___ day!"],
      note: "",
      opts: ["beautiful", "cross", "carry", "strong"]
    }
  ],
  sessions: 12,
  totalCorrect: 150,
  totalWrong: 30,
  streakDays: 3,
  recentErrors: [
    { word: "uncertain", date: "2026/7/15" }
  ]
}
```

### 單字物件（BUILTIN / custom 共用格式）

```javascript
{
  e: "reach",                   // 英文單字（唯一鍵）
  zh: "到達；搆到",              // 中文意思
  pos: "v",                     // 詞性
  hint: "re__ 5字母",           // 拼字遊戲提示
  sentences: [                  // 多組例句（___ 代表空格）
    "Can you ___ the salt?",
    "She managed to ___ the top.",
    "He stood on his toes to ___ the shelf.",
    "The hiker could not ___ camp before dark."
  ],
  opts: ["reach","carry","cross","land"],  // 填空遊戲選項（含正確答案）
  note: ""                      // 注意事項（如易混淆拼法）
}
```

---

## 五、匯出 / 匯入規格

### 匯出 JSON 格式

```json
{
  "version": 2,
  "exportedAt": "2026-07-15T10:30:00.000Z",
  "stats": { "reach": { "correct": 5, "wrong": 1, "streak": 3 } },
  "custom": [],
  "sessions": 12,
  "totalCorrect": 150,
  "totalWrong": 30,
  "streakDays": 3,
  "recentErrors": []
}
```

### 匯入邏輯（v6 起分兩種模式）

`confirmImport(mode)`：

| | `'words'` 只匯入單字 | `'all'` 連紀錄一起 |
|---|---|---|
| custom 單字 | 只新增不覆蓋，同名以自己的為準 | 同左 |
| stats | **完全不動** | `Object.assign(現有, 匯入)`，匯入的覆蓋 |
| 數值統計 | **完全不動** | 取 `Math.max(現有, 匯入)` |
| recentErrors | **完全不動** | 合併後取前 30 筆 |
| 二次確認 | 不需要 | 需要（會列出將被覆蓋的單字數）|

> **為什麼要分兩種**：v5 只有一種匯入，會把統計一起蓋掉。
> 朋友分享單字給你時，你的練習紀錄會被他的洗掉 —— 這是 bug，v6 已修。
> 單字一律「只新增不覆蓋」，所以不管哪種模式都不會弄丟自己的單字。

## 六、頁面架構

```
.app
├── header（標題 + TTS 測試按鈕）
├── .nav（練習 / 口說 / 單字管理 / 學習統計）
├── #page-home
│   ├── #home-select（模式選擇 + 範圍篩選 + 開始按鈕）
│   └── #home-game（動態渲染題目）
├── #page-speak（口說：念單字 / 念句子 → 麥克風 → 評分）
├── #page-manage
│   ├── 新增單字（分頁：一次一個 / 一次多個）
│   └── 單字列表（搜尋 + 篩選）
└── #page-stats
    ├── 統計數字 grid
    ├── 需加強單字列表
    ├── 最近錯誤列表
    ├── 匯出 / 匯入區塊
    └── 重置按鈕
```

---

## 七、遊戲流程

```
startGame()
  → 依 SCOPE 篩選 wordPool
  → weightedSample(pool, 20)
  → G = { mode, words, idx, score, total, wrong }
  → renderQuestion()
      ├── fill  → renderFill(w)  → checkFill()   → afterAnswer() ─┐
      ├── tf    → renderTF()     → checkTF()     → afterAnswer() ─┤
      ├── spell → renderSpell(w) → checkSpell()  → afterAnswer() ─┼→ 對:自動 nextQ()
      ├── lword → renderListen(w,false) → checkListen() → afterAnswer() ─┤   錯:等使用者按
      ├── lsent → renderListen(w,true)  → checkListen() → afterAnswer() ─┘
      └── match → renderMatch()  → pickMatch()   → 組完 → 下一組 or renderEnd()
  → renderEnd()（顯示分數 + 答錯單字列表）
```

---

## 八、TTS（文字轉語音）

v6 改成**佇列式**：一次排入多段語音，靠 `onend` 串接，才能做到「複誦 3 次」。
`speechSynthesis.cancel()` 後要延遲約 120ms 再 `speak()`，否則 Chrome 第一段會沒聲音。

```javascript
speakSeq([{t:'train', r:0.78, gap:380}, ...], onDone);   // 念完才呼叫 onDone
stopSpeak();                                             // 中斷並清空佇列
```

### ⚠️ 底線問題（v5 bug，v6 已修）

例句裡的 `___` 若直接丟給 TTS，會被念成底線。**送進 TTS 前一定要先還原成單字**：

```javascript
function fillBlank(sent, word){
  return String(sent||'').replace(/[_＿]{2,}/g, word).replace(/\s+/g,' ').trim();
}
```

注意用 `/g`：v5 用 `replace('___', w)` 只換第一個，遇到「一句兩個空格」的例句
（如舊版 `simple` 的第 4 句）就會殘留底線被念出來。v6 已把該例句改成單一空格。

### 換題複誦

```javascript
function repeatItems(word, sent){   // 單字 ×N + 完整句子 ×N
  var n = DB.settings.repeat || 0;  // 0 / 1 / 2 / 3，存在 DB.settings
  ...
}
```

- 單字：`rate 0.78`；完整句子：`rate 0.72`
- **對錯判斷模式換題不念**（依需求），只留手動「🔊 念整句」按鈕
- 播放時右下角出現「⏭️ 跳過語音」

### 答完題的換頁規則

```
答對 → 念完複誦 → 自動下一題
答錯 → 停住，等使用者按「下一題 →」（解析看得完為止）
```
用 `G.tok` 當令牌：使用者手動按下一題後，先前排隊的語音回呼不會重複跳題。

**瀏覽器支援：** Chrome ✅ Safari ✅ Firefox ✅（部分）

## 九、下一版規劃

### 目前刻意不做的事

| 項目 | 為什麼不做 |
|------|-----------|
| 前端直接呼叫 Claude API 生成例句 | ⛔ **本站是公開的 GitHub Pages，金鑰放前端等於公開送人盜刷。**（v5 文件曾規劃此做法，已否決）詳見第十一章。 |
| 跨裝置 / 跨使用者同步 | 需要後端資料庫；使用者已決定「保持完全獨立」，各自 localStorage，靠匯出匯入搬移。 |
| 自動翻譯 | 品質不足且劍橋無法接，見第十一章實測。 |

### 若未來真的要做同步

必須有後端（Supabase / Cloudflare Workers 等），且要處理：
1. 金鑰不可進前端 → 需 serverless proxy
2. 公開網站 → 需權限控制，否則陌生人可改題庫
3. 會失去「純離線可用」這個優點

### 可以做的小改進

- 配對遊戲計分邏輯與 `G.total` 統一（見第十章）
- 更多易混淆單字的 `note` 提醒
- 內建單字的中文若發現有誤，直接改 `BUILTIN` 重新部署

---

## 十、已知問題 / 注意事項

| 問題 | 說明 | 處理方式 |
|------|------|---------|
| localStorage 上限 | 約 5MB，目前使用量極小 | 目前不影響，v7 改 window.storage |
| TTS 在部分 Android 瀏覽器無聲 | 需安裝 Google TTS 引擎 | 提示使用者安裝 |
| 配對遊戲 match 不計入 G.total | mState 獨立管理分數 | v7 統一計分邏輯 |
| ~~自訂單字 opts 固定 4 個~~ | v6 已修：出題時自動從題庫挑同詞性的字當干擾選項 | ✅ 已解決 |
| quiet / behind 易拼錯 | 已加 `note` 欄位 + warn-tag 提示 | 持續新增其他易錯字 |

---

## 十一、單字從哪裡來（重要：不要重複研究）

### 工作流程

```
朋友／使用者把英文單字清單丟給 Claude
        ↓
Claude 寫：中文意思 + 4 組例句 + 提示 + 干擾選項 + 易混淆提醒
        ↓
寫進 BUILTIN 陣列 → 重新部署 → 所有人下次開網頁就有新字
```

**網站本身沒有任何翻譯功能。** 所有中文都是人工寫死在 `BUILTIN` 裡的靜態資料。
使用者在網站上「新增單字」時，中文必須自己填（`validateWord` 會擋沒填中文的）。

### 中文的來源與可靠度

| 來源 | 數量 | 說明 |
|------|------|------|
| v5 檔案裡既有 | 100 | 先前的 Claude 對話寫的 |
| 2026-07-15 新增 | 35 | 本次對話寫的 |

⚠️ **這些中文沒有經過辭典查證**，來源是模型知識。A1–A2 基礎字通常沒問題，
但可能漏字義或語感有偏差。使用者若要查證，請自行對照劍橋英漢辭典。

### ❌ 已評估並否決的自動翻譯方案（2026-07-15 實測）

| 方案 | 為什麼不行 |
|------|-----------|
| **劍橋辭典網頁** | 回傳 520 + 無 `Access-Control-Allow-Origin` → 瀏覽器 CORS 直接擋死。且其條款禁止爬取。 |
| **劍橋官方 API** | 302 轉址，需申請授權金鑰（付費）。 |
| **Claude API 即時翻譯** | 金鑰必須放進前端；本站是**公開的 GitHub Pages**，任何人都能看原始碼偷金鑰盜刷。**絕對不可以做。** |
| **MyMemory 免費翻譯** | 技術可行（CORS `*`、免金鑰），但單字無上下文品質太差：實測 `save`→「保存」（漏拯救／節省）、`train`→「列車」（漏訓練）。且不會產生例句，而本站的填空／聽力**高度依賴例句**。 |

> 結論：維持「Claude 人工撰寫 → 更新 BUILTIN」的模式。品質最好、零外部相依、可離線。
> **本檔案不得出現任何 API 金鑰。** 目前全站零對外連線（無 fetch / XHR / 外部資源）。

---

## 十二、單字庫現況（v6）

| 來源 | 數量 |
|------|------|
| 內建單字（BUILTIN） | 135 個 |
| 自訂單字（custom） | 依使用者新增 |

### 批次新增紀錄

| 日期 | 批次內容 |
|------|---------|
| 初始版 | reach、carry、appear、cross 等核心動詞 |
| 批次 2 | perhaps、church、different、difficult 等 23 字 |
| 批次 3 | along、lie、allow、rise、raise 等 55 字 |
| 2026-07-06 | list、cent、dollar、north、south、east、west、join、price、rule、ruler、ready、fine、kill、top、bottom、size、spring、pot、plant、dark、answer、foreign 等 23 字 |
| 2026-07-15/16 | foreigner、save、heat、animal、zoo、monkey、train、training、pull、push、married、marry、farm、farmer、wide、sale、easy、hair、finish、deal、poor、practice、trip、date、police、home、talk、same、cookie、truck、hospital、plan、idea、radio、pool 共 35 字（TF 題庫同步 +22 題，共 52 題）|

---

## 十三、版本紀錄

| 版本 | 日期 | 主要變更 |
|------|------|---------|
| v1 | 2026-07 | 填空 + 拼字，固定例句 |
| v2 | 2026-07 | 新增對錯判斷、配對遊戲 |
| v3 | 2026-07 | 加入不熟單字加強模式 |
| v4 | 2026-07 | 批次追加單字（along、lie 等） |
| v5 | 2026-07-15 | 多例句輪替、自適應出題、TTS、錯題解析、手動新增、匯出匯入 |
| v6 | 2026-07-15 | 聽力測驗（單字／句子填空）、口說練習、換題複誦、錯題手動換頁、批次新增＋防呆、單筆／全部刪除、匯入分兩種模式、TTS 不再念底線、+35 單字、部署到 GitHub Pages |
| v7 | 🔜 規劃中 | 跨裝置同步（需後端，目前刻意不做）|
