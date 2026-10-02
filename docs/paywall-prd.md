# Catcher Academy 付費牆需求文件（PRD）

- 版本：v1.0（2026-10-02）
- 金流：Stripe
- 參考產品：[Aristotle](https://www.heyaristotle.com/pricing)（免費試用 → 輕量月費 → 完整月費）
- 狀態：需求已與創辦人逐項確認；「待確認」的項目在文末列出

---

## 1. 目標

1. 家長能自己完成試用、付費、升降級、取消、退款，不需要人工介入。
2. **孩子永遠看不到價格或付費畫面**，所有付費相關操作只出現在家長端。
3. **孩子不會因為費用在寫功課時被中斷。**這是最高原則，與其他規則衝突時以這條為準。
4. 主推 Catcher（$200）；Lite 的作用是對照組。

## 2. 方案總覽

| 方案 | 價格 | 使用額度 | 語音家教 | 計費方式 | 購買方式 |
|---|---|---|---|---|---|
| 免費試用 | $0 | Catcher 完整功能，無限制 | 有 | 7 天，不綁卡，再加 3 天寬限期 | 註冊即開始 |
| Lite | $20/月 | 每月 20 小時，每天最多 1 小時 | 沒有（文字＋手寫） | 月費自動續訂 | Stripe Checkout |
| Lite 學期 | $80 一次付清（5 個月，8 折） | 同 Lite | 沒有 | 一次性付款，不自動續約 | Stripe Checkout |
| Catcher | $200/月 | 無限制 | 有 | 月費自動續訂 | Stripe Checkout |
| Catcher 學期 | $800 一次付清（5 個月，8 折） | 同 Catcher | 有 | 一次性付款，不自動續約 | Stripe Checkout |
| Academy | $5,000/月 | Catcher + Tony 到府 | 有 | 月費 | 申請制，審核後人工寄 Stripe 付款連結 |

- **語音家教只給 Catcher（已確認）**：試用、Catcher、Catcher 學期、Academy 有語音；Lite 和 Lite 學期只有文字聊天和手寫。語音（OpenAI Realtime）是成本最高的功能，也是 Lite 升級到 Catcher 最主要的理由。
- 幣別：USD。
- 付款方式：Stripe Checkout 支援的信用卡、Apple Pay、Google Pay、Link。
- Tony 的 1:1 真人課（$150/小時）**不在本次範圍**：沒有預約或付款系統，頁面上只放「聯繫官方」，由專人處理。

## 3. 帳號結構

- **一個家長帳號 = 一個 Stripe Customer**，可以加入多個孩子。
- **每個孩子各自一份訂閱**，各自選方案、各自計時。
- 家長在同一個帳號內為所有孩子付款，卡片共用。

## 4. 手足優惠

| 孩子順位 | 折扣 |
|---|---|
| 第 1 個 | 原價 |
| 第 2 個 | 7 折（30% off） |
| 第 3 個起 | 5 折（50% off） |

- 月費和學期方案都適用，學期方案的 8 折之上再疊加。例如第二個孩子的 Catcher 學期方案：$800 × 0.7 = **$560**。
- **順位計算**：同一個家長帳號底下所有「付費中」的孩子訂閱（不含試用和 Academy），按方案價格由高到低排序；價格相同時，先開始付費的排前面。
- **重算時機**：家長新增或取消一個孩子的付費訂閱時重新排序，新折扣從**下一個扣款週期**生效，當期不追補也不退。
- **Stripe 實作**：建兩張 Coupon，`SIBLING_30`（30% off，duration=forever）與 `SIBLING_50`（50% off，duration=forever），套在對應孩子的 Subscription 上。學期方案則在建立 Checkout Session 時帶入 `discounts`。

## 5. 免費試用

- 家長註冊、加入孩子後立即開始 **7 天 Catcher 完整版試用**，**不需要綁卡**。
- 試用期間在家長端顯示剩餘天數和「選擇方案」按鈕。
- **第 7 天結束，家長還沒付費**：自動給 **3 天寬限期**，孩子照常使用，功能不變。
  - 寬限期第 1 天和第 3 天寄 Email 給家長，附付費連結。
- **第 10 天結束，仍未付費**：
  - 孩子端：顯示溫和訊息，例如「Great work this week! Ask a grown-up to keep going.」，**不出現任何價格**。進行中的課照第 7 節的原則讓他上完。
  - 家長端：顯示選擇方案的畫面。
- 試用中任何時候付費：試用立刻結束，付費方案即時生效，從付款當天起算計費週期。

## 6. 付費流程（Checkout）

1. 家長在家長端點「選擇方案」，選孩子、選方案（月費或學期）。
2. 後端建立 Stripe Checkout Session：
   - 月費：`mode=subscription`，帶入對應 Price 與手足 Coupon。
   - 學期：`mode=payment`，帶入一次性 Price 與手足 Coupon。
   - `metadata` 帶 `family_id`、`child_id`、`plan`、`billing`（monthly 或 semester）。
3. 家長在 Stripe 頁面完成付款，導回 `/parent/billing?status=success`。
4. **權限以 webhook 為準**：前端不能因為導回成功頁就開通，要等後端收到 webhook 並更新狀態。成功頁顯示「Setting things up…」並輪詢狀態。

## 6.5 語音家教的開關（Lite 沒有語音）

- 後端在建立語音連線（`POST /api/realtime/session`、`api/realtime/token`）前檢查 `can_use_voice(child_id)`；Lite 一律拒絕，不能只靠前端隱藏。
- Lite 孩子的上課頁**不自動連語音**，也不顯示麥克風、喇叭和「Voice connection unavailable」訊息；Catchie 直接以文字陪伴。孩子端**不出現「升級才有語音」之類的字眼**（見第 14 節）。
- 家長端：方案比較、Lite 孩子的卡片和報告中，標示「Voice tutoring — Catcher」，附升級按鈕。
- 試用結束轉 Lite：語音在轉換當下關閉。Catcher 降級 Lite：本期結束前保留語音，下期起關閉。Lite 升級 Catcher：語音立即開通。
- 進行中的語音課**不會因為方案變更被中斷**（與第 7 節的不打斷原則一致），只在開新課時套用新方案。

## 7. 使用額度與「不打斷」原則（Lite）

**額度**
- 每天最多 **60 分鐘**，每月最多 **1,200 分鐘（20 小時）**。
- 「每天」以家長帳號設定的時區午夜重置。
- 「每月」以該孩子的**計費週期**重置，學期方案則以購買日起每滿一個月重置。
- **一堂課的定義（已確認）**：孩子開始一堂課（按「Start practice」進入上課頁）就開始計時，到離開上課頁為止。後端要建立一筆 `usage_sessions`，記錄開始與結束時間；不能沿用現在「每交一題就一個新 session」的做法。
- **閒置判定**：沿用團隊既有的掛機判定機制，判定為掛機的時間不計入額度。計時以後端為準。

**不打斷原則**
- **進行中的課永遠不會因為額度用完被中斷**，讓孩子寫到他自己停下來。
- 額度用完後的超時分鐘**不扣額度、不收費**，由公司吸收。帳上記錄 `overage_minutes` 供內部分析。
- 額度只在**開始新的一堂課時**檢查：當天或當月已用完，就不能開新的一堂。
- 快用完時**不顯示倒數或警告**，改由 Catchie 鼓勵孩子，例如「You're on fire today. Let's keep going!」。
- 額度用完後，孩子想開新課時看到：「Amazing work today! See you tomorrow.」（月額度用完時是「See you next month」或請家長協助），**不出現價格**。
- 家長端：當天或當月額度用完時，寄 Email 給家長，附升級到 Catcher 的連結。

Catcher、Academy 和試用期間沒有任何額度限制。

## 8. 升級與降級

**升級（Lite → Catcher）**
- **立即生效**，額度限制當下解除。
- **本期不補差額**：Stripe 更新 Subscription 時設定 `proration_behavior: 'none'`，下一個扣款日起才收 $200。
- 升級成功後，家長端顯示：**「This month on us! Appreciate your trust!」**
- 學期方案的升級：見第 13 節待確認。

**降級（Catcher → Lite）**
- 在**本期結束時**生效（用 Stripe Subscription Schedule），本期內維持 Catcher。
- 家長端顯示「Your plan changes to Lite on {date}」，並可在生效前取消降級。

## 9. 取消訂閱

- 家長在家長端點「取消訂閱」，設定 `cancel_at_period_end: true`。
- **服務用到本期最後一天**，之後停止。孩子端看到的訊息同第 5 節，不出現價格。
- 本期結束前，家長可以隨時「恢復訂閱」。
- 學期方案沒有取消動作，到期自動結束（見第 11 節）。

## 10. 退款（30 天退款保證）

- 只適用**每個孩子的第一筆付款**（第一個月的月費或第一筆學期付款），付款後 **30 天內**可申請。
- 家長端有「申請退款」按鈕，**自助完成，不需聯繫客服**：
  1. 後端透過 Stripe Refund API 全額退款。
  2. 同時立即取消該訂閱，服務立即停止。
- 超過 30 天或非首筆付款：不顯示退款按鈕，只能取消。特殊情況由客服在 Stripe 後台手動處理。
- 同一個孩子只能退款一次，退款後再訂閱不再享有退款保證。

## 11. 學期方案

- 一次付清 5 個月：Lite $80，Catcher $800（手足優惠可疊加）。
- 有效期：從付款日起算 **5 個月**（例如 2026-10-02 付款，到 2027-03-02 為止）。
- **不自動續約**。到期前 **14 天**和 **3 天**寄 Email 提醒家長續購，可選再買一個學期或改成月費。
- 到期後未續購：孩子端與家長端的行為同第 5 節「第 10 天結束」。
- 退款依第 10 節：付款後 30 天內可全額退。

## 12. 扣款失敗

- 由 Stripe Smart Retries 自動重試扣款，並由 Stripe 寄信請家長更新卡片。
- 扣款失敗後給 **7 天寬限期**，期間**孩子照常使用，服務不中斷**。
- 家長端顯示橫幅「Payment failed. Update your card to keep things running.」，附更新卡片的連結（Stripe Customer Portal）。
- 7 天後仍未成功：暫停服務，孩子端訊息同第 5 節。家長付款成功後立即恢復。

## 13. Academy（$5,000/月，申請制）

- 網站只做**申請表單**，送出後通知團隊，不在網站上直接付款。
- 團隊審核通過後，在 Stripe 後台建立月費訂閱並寄付款連結給家長。
- 家長付款後，webhook 依 `metadata.plan=academy` 自動開通權限；或由管理員在後台手動開通。
- 名額上限 6 個家庭，由團隊人工控管。

## 14. 「孩子永遠看不到價格」

- 價格、方案、付款、帳單**只出現在家長端**（家長登入後的 `/parent/*`）。
- 孩子端所有提示只用「Ask a grown-up」這類語氣，不出現金額、方案名稱或「升級」字眼。
- AI 對話中不能提到付費、方案或升級。

## 15. Stripe 設定

**Products / Prices**

| 方案 | Product | Price | 類型 |
|---|---|---|---|
| Lite 月費 | Catcher Lite | $20 | recurring, monthly |
| Lite 學期 | Catcher Lite – Semester | $80 | one-time |
| Catcher 月費 | Catcher | $200 | recurring, monthly |
| Catcher 學期 | Catcher – Semester | $800 | one-time |
| Academy | Catcher Academy | $5,000 | recurring, monthly |

**Coupons**：`SIBLING_30`、`SIBLING_50`（見第 4 節）。

**需要處理的 webhooks**

| Event | 用途 |
|---|---|
| `checkout.session.completed` | 開通月費或學期方案、結束試用 |
| `customer.subscription.created` / `updated` | 同步方案、狀態、週期、降級排程 |
| `customer.subscription.deleted` | 本期結束、停止服務 |
| `invoice.paid` | 續訂成功，更新週期、清除扣款失敗狀態 |
| `invoice.payment_failed` | 進入 7 天寬限期 |
| `charge.refunded` | 標記已退款、停止服務 |

- 所有 webhook 要驗證簽章，並以 event id 做冪等處理。
- 權限判斷一律讀後端資料庫，不直接打 Stripe API。

## 16. 資料模型（建議）

```
families        id, parent_user_id, stripe_customer_id, timezone
children        id, family_id, name, grade
subscriptions   id, child_id, plan(lite|catcher|academy), billing(trial|monthly|semester),
                status(trialing|grace|active|past_due|canceled|expired|refunded),
                stripe_subscription_id, stripe_payment_intent_id,
                current_period_start, current_period_end, trial_ends_at, grace_ends_at,
                cancel_at_period_end, scheduled_plan, scheduled_change_at,
                first_payment_at, refund_eligible_until, sibling_rank, coupon_id
usage_sessions  id, child_id, started_at, ended_at, counted_minutes, overage_minutes
```

**權限判斷（後端統一函式）**：`can_start_session(child_id)`
1. 狀態是 `trialing`、`grace`、`active`、`past_due`（寬限 7 天內）→ 繼續往下判斷；其他狀態 → 不行。
2. 方案不是 Lite → 可以。
3. 方案是 Lite → 今天已用分鐘 < 60 **且**本月已用分鐘 < 1,200 → 可以；否則 → 不行。

進行中的課**不呼叫**這個函式，因為永遠不中斷。

**語音判斷**：`can_use_voice(child_id)`：`can_start_session` 為「可以」**且**方案是試用、Catcher（月費或學期）或 Academy → 可以；Lite → 不行。在每堂課開始時判斷一次。

## 17. 通知一覽（寄給家長）

| 時機 | 內容 |
|---|---|
| 試用第 7 天 | 試用結束，再給你 3 天，附選方案連結 |
| 寬限期第 1、3 天 | 提醒選方案 |
| Lite 當天或當月額度用完 | 今天的努力 + 升級 Catcher 連結 |
| 升級成功 | This month on us! Appreciate your trust! |
| 降級排程 | 方案將於 {date} 改為 Lite |
| 扣款失敗 | 請更新卡片，服務 7 天內不中斷 |
| 學期到期前 14、3 天 | 續購提醒 |
| 退款完成 | 退款確認 |

## 18. 驗收標準（QA 測試案例）

1. 新家長註冊、加入孩子 → 立即可用 Catcher 完整功能，不需輸入卡號。
2. 試用第 8 天 → 孩子照常使用；家長收到提醒信。
3. 試用第 11 天、未付費 → 孩子無法開新課，訊息中**沒有任何價格**；家長端顯示選方案畫面。
4. Lite 孩子當天已用 55 分鐘、正在上課 → 課程持續到孩子自己結束，不會中斷；超過 60 分鐘的部分記為 overage，不扣額度。
5. 同上，下課後再開新課 → 被擋，看到「See you tomorrow」，家長收到升級信。
6. Lite 升級 Catcher → 限制立即解除；本期不另收費；下期扣 $200；家長端顯示「This month on us!」。
7. Catcher 降級 Lite → 本期維持 Catcher，下期起為 Lite。
8. 第二個孩子訂閱 Catcher 月費 → 帳單 $140；第三個孩子訂閱 Lite 月費 → 帳單 $10。
9. 第二個孩子的 Catcher 學期方案 → 一次收 $560。
10. 首次付款後第 20 天申請退款 → 全額退回、服務立即停止；第 31 天 → 沒有退款按鈕。
11. 扣款失敗 → 7 天內孩子照常使用；第 8 天暫停；更新卡片成功後立即恢復。
12. 取消訂閱 → 用到本期最後一天；期間可以恢復。
13. 學期方案到期 → 不自動扣款；到期前 14、3 天收到提醒信。
14. 孩子端任何畫面、任何 AI 回覆都不出現價格、方案名稱或「升級」。
15. Lite 孩子開始上課 → 沒有麥克風按鈕、不嘗試語音連線；直接呼叫語音 API 時後端回拒絕。
16. Lite 升級 Catcher → 下一堂課立即有語音；Catcher 降級 Lite → 本期結束前仍有語音，下期起沒有。

## 19. 待確認事項


**已確認**
- 語音家教只給 Catcher（見第 2、6.5 節）。

**已確認、不列入付費牆範圍**
- 家長報告的內容（包含沒開語音的孩子）由團隊另外處理。
- 題目 API 會把正確答案傳到瀏覽器的問題，收費前不修。

1. **「寫到完」的解讀**：目前寫成「進行中的課永遠不中斷、超時不扣額度、額度用完後不能開新課」。如果意思是 Lite 根本不擋新課，請告訴我。
2. **試用的次數**：每個孩子都有一次 7 天試用，還是每個家庭只有一次？（建議：每個家庭一次。之後加入的孩子不再試用，避免重複使用。）
3. **學期方案中途升級**（Lite 學期 → Catcher）：要補差額，還是同樣「This month on us」直接升級？
4. **銷售稅**：是否啟用 Stripe Tax 代收美國各州銷售稅？（建議請會計確認，數位教育服務在部分州需要課稅。）
5. **頁面文案要同步更新**：v23 已改成「20 hours a month, up to 1 hour a day」，並標示語音只在 Catcher；手足優惠的說明還沒加。
