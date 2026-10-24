# Bug Report 範本

下面這段直接複製進 Jira / Linear / GitHub Issue / Notion。
欄位說明見 [README.md](README.md)。

## 空白範本

````markdown
# [模組] <行為>，<前提下> 應該 <預期> 但 <實際>

## Severity / Priority

S? / P? — <一句話說明影響範圍>

## Steps to Reproduce

1. 開啟 <完整 URL>
2. 登入 <帳號> / <密碼>
3. ...
4. ...

## Expected

- <預期行為 1>
- <預期行為 2>

## Actual

- <實際行為 1>
- <實際行為 2>

## Environment

- OS:
- Device:
- Browser:
- App version:
- Backend:
- Account:
- Network:
- 時間:

## Attachments

- 影片重現：<連結>（<秒數>）
- Network HAR：<連結>
- Console log：<連結>
- Screenshot：<檔名>

## Related

- 類似 bug：#
- 相關 spec：<連結>
- 暫時 workaround：
````

## 填好長這樣

````markdown
# [Checkout] iOS Safari 17 上連點「確認付款」會重複建立訂單

## Severity / Priority

S1 Critical / P0 — 影響使用者收費，每天可能發生

## Steps to Reproduce

1. iOS 17.4 Safari 開 https://staging.example.com
2. 登入 qa-test-007@example.com / Test@123456
3. 加入「商品 A (sku-001)」到購物車
4. 進入結帳頁
5. 選「信用卡」、輸入測試卡 4111 1111 1111 1111 / 12/30 / 123
6. 在「確認付款」按鈕快速連點兩次（間距 < 1 秒）
7. 等待 5 秒

## Expected

- 第二次點擊應被忽略（按鈕應 disable 或顯示 loading）
- 只建立 1 筆訂單

## Actual

- 兩次點擊都被接受
- 建立 2 筆相同訂單（order #12345 + #12346）
- 信用卡被扣款兩次

## Environment

- OS: iOS 17.4
- Device: iPhone 13 Pro
- Browser: Safari 17.4
- App version: v2.4.1 (build #2026-06-09-001)
- Backend: staging-api-v3 (commit a1b2c3d)
- Account: qa-test-007@example.com
- Network: WiFi
- 時間: 2026-06-09 14:32 UTC+8

## Attachments

- 影片重現：https://loom.com/share/xxx（15 秒）
- Network HAR：attachments/network.har
- Console log：attachments/console.txt
- Screenshot：screenshot-2026-06-09.png

## Related

- 類似 bug 於 2025 Q4：#3421（Android Chrome）
- 相關 spec：https://confluence.example.com/checkout-double-click
- 暫時 workaround：使用者刷新頁面、客服手動退款一筆
````

---

## 送出前自己檢查

- [ ] Title 唸出來別人聽得懂？
- [ ] Steps 有寫死的帳號、商品 ID、測試卡號？
- [ ] Expected 跟 Actual 條數對得起來？
- [ ] Environment 有 build number 或 commit hash？
- [ ] 有附 video 或至少 console + network log？
- [ ] 這份 report 只講一個 bug？

六項全勾，退回率會明顯下降。

---

→ 深入說明：[Bug Report 撰寫 SOP — 8 個欄位讓開發 30 秒看懂、不被退回](https://9niche.com/guide/bug-report-sop/)
