# Catcher Academy 產品功能盤點與付費問題清單

- 盤點日期：2026-10-02
- 方式：用家長測試帳號登入 catcheracademy.com 實際操作，並閱讀前端程式碼，整理出所有頁面、按鈕與後端 API。
- 範圍說明：測試帳號底下沒有孩子（0 / 1 seats）。為了不改動帳號資料，我**沒有新增孩子、沒有按「Add seats」**。所以學生端的畫面是從前端程式碼整理出來的，沒有實際點過。如果需要，我可以在你同意後建一個測試孩子，把學生端實際走一遍。

---

## 一、頁面地圖

| 路徑 | 頁面 | 給誰用 |
|---|---|---|
| `/home`、`/welcome`、`/landing` | 官網首頁、登入註冊彈窗、Beta 名單頁 | 訪客 |
| `/about`、`/about/team`、`/about/product` | 公司介紹、團隊、產品介紹 | 訪客 |
| `/founding-family` | Founding Families 招募頁（連到 Tally 表單） | 訪客 |
| `/terms-of-service`、`/privacy-policy` | 服務條款、隱私權政策 | 所有人 |
| `/confirm-email` | Email 驗證跳轉 | 家長 |
| `/parent` | 家長首頁：管理孩子 | 家長 |
| `/report`、`/report/:childId`、`/report/:childId/detail` | Focus Report 家長報告 | 家長 |
| `/settings` | 外觀設定（Appearance） | 家長、學生 |
| `/student-home` | 學生首頁：跟 Catchie 聊天 | 學生 |
| `/dashboard` | 學習花園地圖 | 學生 |
| `/lesson/:lessonId` | 上課頁（題目、手寫、語音家教） | 學生 |
| `/canvas` | 手寫畫布 | 學生 |
| `/history`、`/history/practice/:id`、`/history/quiz/:id` | 練習與小考紀錄 | 學生 |

---

## 二、詳細功能列表

### 1. 帳號與登入
1. **兩種身分登入**：登入視窗有「Learner / Parent」切換。
   - 家長：Email（或 username）＋密碼，或 **Google 登入**。
   - 學生：只用家長給的 **Login code** 登入，不需要 Email。
2. **只有家長能註冊**：註冊時身分固定為 Parent，孩子無法自己註冊。
3. **註冊審核狀態**：註冊後有三種狀態：直接啟用、需要到信箱點確認連結、**等待審核（pending）**。
4. **學生帳號啟用**：孩子第一次用預設密碼登入時，要自己設定新密碼（至少 6 個字元，且不能跟預設密碼相同）。
5. **同時只能一處登入**：同一帳號在別處登入時，原本的裝置會被登出，並提示「this account was used to log in somewhere else」。
6. **同意條款**：註冊和 Google 登入前要勾選同意條款。目前同意紀錄只存在瀏覽器的 localStorage。
7. 後端使用 Firebase Authentication（Email 密碼、Google、Email link 等）。

### 2. 家長端：家庭管理（`/parent`）
1. 顯示「Hi, {名字}」與「My children」列表。
2. **座位（Seats）**：顯示「已用 / 總數 seats」，測試帳號預設 **1 個座位**。座位用完時顯示「You've used all your seats. Add seats to create more children.」
3. **Add seats 按鈕**：座位滿時出現，按下後直接呼叫 `POST parent/seats` 增加座位，**目前沒有任何付款步驟**（詳見第三節）。
4. **新增孩子**：填姓名、年齡（選填，0–120）、年級（選填，1–12 年級、大學、研究所、其他）。建立後顯示該孩子的 **Login code**，可一鍵複製。
5. **孩子卡片**：姓名、Login code（可複製）、年齡、年級、外觀設定（Custom 或預設，含更新日期）、狀態（Active / Invited）。
6. **每個孩子的操作**：
   - **Go to study**：從家長端直接進入學習。
   - **View report**：看這個孩子的報告。
   - **Edit**：改姓名、年齡、年級。
   - **Reset password**：重設孩子密碼（有確認視窗）。
7. **沒有刪除孩子的功能**。
8. 帳號選單：Focus Report、My Children、Appearance、Log out。

### 3. 家長端：Focus Report 報告（`/report`）
1. 多個孩子時可以切換（Previous / Next child、Choose child）。
2. **本週重點（Weekly highlight）**。
3. **每日專注度（Daily focus percentage）**。
4. **專注一致性（Focus index / consistency）**，數據不足時標示「Low confidence」。
5. **學習進度（Mastery）**：已嘗試主題數 / 總主題數、答對題數 / 作答題數，進度圓環。
6. **語音課程列表（Voice sessions）**。
7. **單堂課分析（Session analysis）**：
   - 專注時間軸（Focus timeline）
   - 情緒分布（Emotion distribution）
   - 專注統計（Focus stats）
   - 課程狀態提示（Health banner），例如課程太短、取樣不足
   - 文字摘要，例如「Stayed engaged through most of the session, with a brief dip into frustration when fractions were introduced…」
8. **課程筆記（Session notes）**。
9. **詳細週報（Detailed weekly report）**頁。
10. **安全過濾**：報告不顯示 ADHD 嚴重度、情緒疾患、心理異常、醫療診斷、用藥建議、原始情緒分數、原始對話內容；被過濾的內容會顯示「This note was removed because parent reports must stay educational and observable.」

### 4. 學生端：首頁（`/student-home`）
1. **Catchie 吉祥物**問候，會依狀況說不同的話（歡迎回來、找到新題目等），閒置 7 秒後主動打招呼。
2. **文字聊天**：「Type to Catchie…」。
3. **語音聊天**：「Tap the mic and talk to me!」，可以停止語音模式。

### 5. 學生端：學習花園地圖（`/dashboard`）
1. 以「花園」呈現學習地圖，每個主題是一塊地，會依精熟度長出不同的植物（成長階段圖例）。
2. 年級分頁可以切換，往下鑽到單元和主題，也能返回上一層。
3. **鎖定機制**：前一個年級完成才解鎖下一個年級（「This area opens after the earlier grade is complete.」）。
4. Catchie 依主題狀態給不同鼓勵（剛開始、進行中、已精熟、鎖定）。
5. 選主題後「Start practice」。

### 6. 學生端：上課頁（`/lesson/:id`）
**題目與作答**
1. 數學題目（用 MathJax 顯示數學式），每次練習 5 題。
2. 選擇題選項預覽與作答視窗。
3. 作答回饋：「Correct! Loading the next one…」、「Answer saved…」。
4. 有 Practice（練習）和 Quiz（小考）兩種模式。

**手寫**
5. **手寫畫布**：可調筆刷粗細、全部清除。
6. 自動擷取手寫畫面上傳給 AI 辨識（handwriting recognition）。
7. 白板同步（board sync），讓 AI 看到孩子寫的內容。

**AI 家教**
8. **提示（Hint）**：浮動提示卡，提示以串流方式逐字出現；寫錯時會自動給提示。
9. **語音家教**：使用 **OpenAI Realtime** 即時語音，可以選麥克風、靜音 Catchie；AI 只用英文、每次回應不超過兩句。
10. AI 回覆面板可展開或收合。

**ADHD 專屬設計**
11. **ADHD 計時器**與時間提醒（timer alert）。
12. **注意力檢查（Attention gate）**：每 5 題一次，例如「Quick eye check: Tap the numbers in order to come back to me.」。
13. **心情檢查（Mood check-in / mood screen）**。
14. **呼吸練習（Breathing）**。
15. **小遊戲**：顏色配對（Color match）、找不同（Odd one out）、路徑遊戲（Trail game）。
16. 步驟進度環（Step ring）、粒子特效、成長動畫（Growth）。
17. **背景音樂**，可調音量或靜音。
18. **減少動態（Reduce motion）**開關。
19. 外觀主題：Forest Warm、Ocean Calm、Focus Bright。

**結束**
20. 課程摘要（Lesson summary）與課程數據（Session metrics），可結束練習（End practice）。

### 7. 學生端：歷史紀錄（`/history`）
1. 練習紀錄和小考紀錄分開，可以切換。
2. 每筆顯示日期、正確率、狀態（Needs review、In progress）。
3. 點進去看每題詳細：孩子的手寫作答圖片、選了哪個答案、正確答案、解題步驟。

### 8. 外觀設定（`/settings`）
1. 主題色：Warm forest、Calm ocean、High focus。
2. 自訂模式可調：視覺風格、版面密度、圖示風格、裝飾程度、動態程度。
3. 即時預覽。

### 9. AI 安全
1. 有「AI learning safety」說明。
2. AI 規則：不診斷、不評估心理健康風險、不建議用藥、不詢問隱私身分資料、不鼓勵孩子對家長或老師隱瞞。
3. 後端有安全政策與安全事件紀錄 API（`safety/policy`、`safety/events`）。

### 10. 官網與行銷
1. 首頁、About、Team、Product（含 3 分 39 秒示範影片）。
2. Founding Families 招募（Tally 表單），條款中提到入選家庭可能獲得**終身免費**。
3. Beta 名單收集（Email）。
4. 家教頁在另一個子網域（blog.catcheracademy.com/math-tutoring），Tony $150 一堂。

### 11. 後端 API 一覽（給工程師對照）
- 帳號：`auth/login`、`auth/firebase`、`auth/register`、`auth/confirm-email`、`auth/activate`、`auth/logout`、`users/me`、`users/grades`
- 家長：`parent/students`（列表、新增）、`parent/students/{id}`（修改）、`parent/students/{id}/reset-password`、`parent/seats`、`parent/students/{id}/mastery`
- 報告：`parent/children/{id}/reports/`（weekly-highlight、daily-focus、focus-index、mastery、voice-sessions、sessions/{sid}/analysis、sessions/{sid}/notes）
- 學習：`learning-map`（root、nested、list、pinned、node、children）、`dashboard/garden`、`mastery`、`lessons`、`lessons/{id}/answer`、`lessons/{id}/problems/{pid}/hint`、`lessons/sessions/{sid}/report`、`practice/start`、`practice/{id}/submit-answer`、`practice/{id}/result`、`practice/records`、`quiz/by-topic/{id}`、`quiz/answer-records`、`assessments`（start、answer、end、history）、`shared/question-bank`
- 手寫與語音：`api/questions/handwriting`、`api/questions/hints/stream`、`api/realtime/token`、realtime `session`、`catchie/chat`、`catchie/greeting`、`session/{id}/image`、`context`、`answer`、`next-problem`、`audio/*`
- 其他：`appearance-profile`、`safety/policy`、`safety/events`

---

## 三、跟付費有關的重要發現

1. **🔴 「Add seats」不用付錢就能加座位。**前端按下去直接呼叫 `POST parent/seats` 並重新載入，沒有付款步驟。如果後端也沒擋，任何家長都能無限新增孩子。**上線收費前一定要先處理。**
2. **目前完全沒有付費相關頁面**：沒有方案頁、帳單頁、付款方式、訂閱狀態、發票，帳號選單裡也沒有 Billing。
3. **現有的「座位」概念**跟我們 PRD 裡「每個孩子一份訂閱」可以直接對應，但要決定怎麼接（見問題 1）。
4. **服務條款寫「fees are non-refundable」**，跟頁面上承諾的「30-day money-back」衝突。
5. **Founding Families 承諾終身免費**，系統需要能標記這些家庭不收費。
6. **語音家教用 OpenAI Realtime**，是最主要的變動成本。Lite 的時間限制、Catcher 的「無限」，成本都跟它直接相關。
7. **已經有「同時只能一處登入」**，可以防止多個孩子共用一個帳號，對按孩子收費有利。
8. **只有家長能註冊**，付款人一定是家長，這點很好。
9. **註冊有「等待審核」狀態**，代表目前可能有人工審核註冊。收費後還要不要保留？
10. **條款同意紀錄只存在瀏覽器**。開始收費後建議存到後端，作為付款與同意的證據。

---

## 四、PM 想跟老闆確認的付費問題

標 🔴 的是會擋住開發、必須先決定的問題。每題都附上我的建議。

### A. 收費單位：座位 vs 訂閱
1. 🔴 **「座位（Seat）」要怎麼對應訂閱？**
   建議：一個座位就是一個孩子的訂閱。家長新增孩子時一起選方案（試用、Lite、Catcher），「Add seats」改成導到 Stripe 付款。
2. 🔴 **註冊時預設送的 1 個座位，算什麼？**
   建議：就是第一個孩子的 7 天試用座位。試用結束未付費時，座位保留，但孩子不能開新課。
3. 🔴 **「Add seats」目前免費，上線前要先關掉或改成付費嗎？**現在已經有多少帳號多加過座位？
   建議：本週先在後端擋住，等付費上線再打開。
4. **家長要移除孩子時怎麼辦？**目前沒有刪除功能。
   建議：新增「停用孩子」，同時取消該孩子的訂閱（用到本期結束），手足折扣自動重算。
5. **多加的孩子也有 7 天試用嗎？**
   建議：每個家庭只有第一個孩子有試用，之後加的孩子直接選付費方案（手足折扣會讓價格好接受）。

### B. 哪些功能要被付費擋
6. 🔴 **Lite 的「時間」算哪些功能？**上課（Lesson）、練習（Practice）、小考（Quiz）、首頁跟 Catchie 聊天、逛花園地圖、看歷史紀錄，哪些要計時？
   建議：只算上課、練習、小考的時間。聊天、地圖、歷史紀錄不計時，讓孩子隨時可以回來。
7. 🔴 **ADHD 休息活動要算時間嗎？**呼吸練習、心情檢查、注意力檢查、三個小遊戲。
   建議：不算。這些是幫孩子回到專注的設計，不該讓家長覺得在浪費額度。
8. 🔴 **語音家教（成本最高）Lite 也有嗎？**
   建議：Lite 也有，因為這是產品核心體驗。但要先算一下：一個 Lite 孩子用滿 20 小時語音，OpenAI 成本是多少？會不會超過 $20？
9. 🔴 **Catcher 的「無限」要不要有隱藏上限，防止濫用？**例如一天 4 小時以上就提醒家長。
   建議：設一個公平使用上限（例如每月 100 小時），超過時通知團隊，但不擋孩子。
10. **家長報告 Lite 有沒有？**目前報告是家長端最有價值的功能（週報、專注度、單堂分析、筆記）。之前 PRD 寫「Lite 沒有報告」。
    建議：Lite 只給「本週重點」，Catcher 解鎖全部報告。家長看得到價值，才會想升級。
11. **歷史紀錄、外觀設定、背景音樂**，是不是所有方案都有？
    建議：都有，這些幾乎沒有成本。
12. **App 裡要不要出現 Tony 真人課的入口？**目前官網有「Meet Tony & explore 1-on-1 tutoring」。
    建議：在家長端放「Talk to our team」聯絡按鈕，不放價格。

### C. 計時規則
13. **「一天」用哪個時區？**
    建議：家長帳號的時區，第一次付費時自動抓瀏覽器時區，家長可以修改。
14. **閒置怎麼算？**孩子開著畫面離開了怎麼辦？
    建議：連續 5 分鐘沒有寫字、說話或作答就暫停計時。
15. **月額度從哪天開始算？**每月 1 號，還是付款日？
    建議：付款日（跟扣款週期同一天），學期方案則從購買日每滿一個月重置。

### D. 帳號與家庭
16. **一個孩子可以屬於兩個家長嗎？**例如父母分開、共同監護。
    建議：第一版不支援。付款人就是建立孩子的那個家長。
17. **付費後還需要註冊審核（pending）嗎？**
    建議：付費用戶直接開通，不需要人工審核。
18. **付款要不要先完成 Email 驗證？**
    建議：要。避免假帳號和收不到收據的問題。

### E. 現有用戶
19. 🔴 **Founding Families 終身免費要怎麼處理？**
    - 免費的是 Lite 還是 Catcher 完整版？
    - 只限當時的那一個孩子，還是之後加的孩子也免費？（條款寫「personal, may not be transferred」）
    - 目前有幾個家庭？名單在哪裡？
    建議：Catcher 完整版終身免費，限申請時登記的孩子；之後加的孩子適用手足折扣。
20. 🔴 **現有的 Beta 用戶開始收費時怎麼處理？**
    建議：提前 14 天通知，給 30 天免費過渡期，前三個月用早鳥價（例如 Catcher $150）。

### F. 法務與合規
21. 🔴 **服務條款要改**：現在寫「fees are non-refundable」，跟 30 天退款保證衝突。誰負責改？
22. **兒童隱私（COPPA）**：用家長信用卡付款，可以同時當作「可驗證的家長同意」的一部分。要不要請律師確認流程？
23. **條款同意紀錄要存到後端嗎？**
    建議：要。存下同意時間和條款版本，付款爭議時用得到。
24. **銷售稅**：要不要啟用 Stripe Tax？（上一份 PRD 也問過）

### G. 介面與內容
25. **付費頁要放在哪裡？**
    建議：帳號選單新增「Plan & billing」，家長首頁每個孩子卡片上顯示方案和到期日。
26. **孩子端的訊息由誰寫？**例如試用到期、額度用完時，Catchie 要說什麼。
    建議：沿用 Catchie 的語氣，由 Jennifer 確認，絕對不出現價格。
27. **價格頁要放到官網哪裡？**目前官網沒有 `/pricing`。
    建議：新增 `/pricing`，導覽列加上「Pricing」。
28. **多語言與多幣別**：資料有英文、繁中、日文。第一版只收美元嗎？
    建議：第一版只收美元、只有英文付費頁。

---

## 五、建議的下一步
1. 先決定標 🔴 的 10 題（1、2、3、6、7、8、9、19、20、21），以及第一份 PRD 的待確認事項。
2. 本週先在後端擋住免費的「Add seats」。
3. 算出語音家教每小時的實際成本，確認 Lite $20 和 Catcher $200 的毛利。
4. 決定後我把答案合併進 `docs/paywall-prd.md`，給前後端開工。
