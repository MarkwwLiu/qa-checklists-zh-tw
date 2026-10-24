# Bug Report SOP — 8 個欄位

被開發退回「無法重現」、「不是 bug」、「資訊不足」三次以上的 bug，
通常是 report 寫得不夠具體，不是開發不配合。

一份好的 bug report，讓開發 30 秒讀懂、5 分鐘重現。

可直接複製的版本：[template.md](template.md)
GitHub Issue Form 版本：[`.github/ISSUE_TEMPLATE/bug_report.yml`](../.github/ISSUE_TEMPLATE/bug_report.yml)

---

## 8 個欄位

| 欄位 | 必填 | 用途 |
|------|------|------|
| Title | 是 | 一句話總結 |
| Steps to Reproduce | 是 | 編號的重現步驟 |
| Expected Result | 是 | 應該怎樣 |
| Actual Result | 是 | 實際怎樣 |
| Environment | 是 | OS / Browser / Version / Build |
| Severity / Priority | 是 | 影響範圍與修復急迫度 |
| Attachments | 是 | Screenshot / Video / Log / HAR |
| Related | 否 | 相關 PR、之前的 bug、spec 連結 |

少一個就會被退回問。

---

## Title

公式：`[模組] <行為>，<前提下> 應該 <預期> 但 <實際>`

不合格：

- 「登入有問題」
- 「按鈕沒反應」
- 「畫面跑掉」

合格：

- 「[Login] 使用者連續輸錯密碼 5 次後，第 6 次仍能正常嘗試（應被鎖定 15 分鐘）」
- 「[Checkout] iOS Safari 17 上，按『確認結帳』按鈕第二次點擊才有反應」
- 「[Profile] 上傳超過 5MB 的頭像時，前端無錯誤提示但後端 500」

標準：能在 stand-up 上唸出來，別人聽得懂。

## Steps to Reproduce

寫死，不要寫泛指。

不合格：

```
1. 進入購物車
2. 加入一些商品
3. 結帳
```

合格：

```
1. 用瀏覽器開 https://staging.example.com
2. 用以下帳號登入：
   - email: qa-test-007@example.com
   - password: Test@123456
3. 進入「商品列表」/products/
4. 將「商品 A (id: sku-001)」加入購物車 x 2
5. 將「商品 B (id: sku-099)」加入購物車 x 1
6. 點右上「購物車（3）」
7. 點「前往結帳」
8. 選「信用卡」付款
9. 輸入測試卡號 4111 1111 1111 1111 / 12/30 / 123
10. 點「確認付款」
```

標準：把這份 steps 交給一個沒接觸過產品的人，他能跑出同樣結果。

### Steps 常見錯誤

- [ ] **跳步驟** — 「然後就 fail 了」，中間少了 3 步
- [ ] **沒寫 test data** — 「輸入 email」沒寫是哪個 email
- [ ] **預設知識** — 「用 admin 進入後台」沒寫怎麼進
- [ ] **多個 bug 寫在同一個 report** — 「另外順便提一下…」請開新的
- [ ] **寫概念而非操作** — 「驗證搜尋功能」要寫具體點什麼、輸入什麼

## Expected vs Actual

一定要分開寫，而且數量盡量一對一。

不合格：

```
結帳後沒有顯示成功頁面，反而跳出錯誤。
```

合格：

```
Expected:
- 點「確認付款」後 → 跳轉到 /order/<id>/success
- 頁面顯示「訂單建立成功」+ 訂單編號
- 寄送確認 email 到 qa-test-007@example.com

Actual:
- 點「確認付款」後 → 停留在原頁面
- 跳出紅色錯誤訊息：「系統忙碌中，請稍後再試」
- 後端 log 顯示 500 InternalServerError
- 未收到確認 email
```

## Environment

開發在自己機器上跑不出來，多半是環境不同。至少要寫：

```
- OS: macOS 14.3 / Windows 11 22H2 / iOS 17.4
- Browser: Chrome 124.0.6367.78 / Safari 17.4
- App version: v2.4.1 (build 2026-06-09)
- Backend: staging-api-v3 (commit a1b2c3d)
- Network: WiFi / 4G / VPN
- Account: qa-test-007@example.com (paid plan / 台灣 / 已驗證)
- 時間: 2026-06-09 14:32 UTC+8
```

Mobile bug 額外要附：

- 裝置型號（iPhone 13 Pro / Pixel 8）
- 螢幕方向（直 / 橫）
- 是否開 dark mode
- 是否啟用 VPN

## Severity vs Priority

兩個常被混用，但問的是不同的事。

| 欄位 | 定義 | 例子 |
|------|------|------|
| **Severity** | 對使用者的傷害 | 資料遺失（critical）、UI 跑版（low） |
| **Priority** | 修復的急迫度 | 影響營收（P0）、極少觸發（P3） |

兩者不一定一致：

- Severity critical 不一定 P0：「會掉資料但只有 1% 流量觸發，而且有 workaround」可能是 P1
- Severity low 也可能 P0：「首頁 logo 變形」對 brand 是大事

建議分類：

```
Severity:
  S1 Critical   - 系統當機、資料遺失、安全漏洞
  S2 High       - 主要功能無法使用
  S3 Medium     - 次要功能異常、有 workaround
  S4 Low        - UI 小瑕疵、文案錯字

Priority:
  P0 Now        - 阻擋發版、需馬上修
  P1 This sprint
  P2 Next sprint
  P3 Backlog    - 修不修都行
```

## Attachments

不要只貼一張 screenshot。完整的證據是：

- [ ] **Screenshot** — 整個畫面，含 URL、含時間
- [ ] **Video** — 重現過程，30 秒以內
- [ ] **Console log** — DevTools Console 全選複製
- [ ] **Network log** — DevTools Network export HAR
- [ ] **Backend log** — 失敗時間區間的 log（如果拿得到）

Video 是金標準，能解掉大約一半的「無法重現」退回。

| 工具 | 用途 |
|------|------|
| Loom / CleanShot | 螢幕錄影 + 上傳分享連結 |
| Chrome DevTools → More tools → Recorder | 錄成可重播的 recording |
| BrowserStack | 跨平台跨瀏覽器重現 |
| Sentry / LogRocket | 自動錄影 + session replay（需前端先接） |

---

## 被退回的常見理由，以及怎麼解

| 退回理由 | 真正原因 | 解法 |
|---------|---------|------|
| 「無法重現」 | Steps 太抽象 | 加 test data + 環境細節 + video |
| 「這是 spec」 | 沒對到 spec | 附 spec 連結，標明差異在哪一行 |
| 「不是 bug」 | Expected 跟 dev 的認知不同 | 拉 PM 三方確認，結論寫回 spec |
| 「資訊不足」 | 缺 console / network log | 補上 |
| 「我這邊 OK」 | 環境不同 | 詳列 OS / browser / build / account |

## 換工具不影響結構

Jira / Linear / GitHub Issues / Notion 決定的是樣式，8 個欄位永遠都要寫。

範本可以做成：

- Jira → Issue template
- GitHub → `.github/ISSUE_TEMPLATE/bug_report.yml`（本 repo 附了一份）
- Notion → Database template

設一次，團隊一直受用。

---

→ 深入說明：[Bug Report 撰寫 SOP — 8 個欄位讓開發 30 秒看懂、不被退回](https://9niche.com/guide/bug-report-sop/)
