# Test Case 撰寫原則 — 7 個欄位

寫 test case 不是把 spec 抄一遍。
一份好的 case 要能讓完全沒接觸過產品的人照著跑，跑出一致的結果。

可直接複製的版本：[template.md](template.md)

---

## 7 個欄位

| 欄位 | 必填 | 用途 |
|------|------|------|
| ID | 是 | 唯一識別、追蹤、被其他文件引用 |
| Title | 是 | 一句話描述「測什麼」 |
| Preconditions | 是 | 執行前必須成立的狀態 |
| Test Data | 是 | 具體輸入值 |
| Steps | 是 | 編號的操作步驟 |
| Expected Result | 是 | 每一步的預期結果 |
| Priority | 否 | P0 / P1 / P2，跑回歸時用來篩 |

少任何一個，半年後維護都會痛。

---

## Title

公式：`<前置條件>，<操作>，<預期>`

不合格：「測試登入」、「驗證密碼」

合格：「以正確 email + 錯誤密碼登入時，顯示『帳號或密碼錯誤』錯誤訊息」

標準跟 bug report 一樣：能在 sprint review 唸出來，別人聽得懂。

## Preconditions

別讓執行的人去猜環境。

> 例：使用者已註冊但尚未驗證 email、購物車有 1 件商品、
> 後台 feature flag `new_checkout=true` 已開啟

**最常見的錯誤**：把 precondition 寫成第一個 step。
「1. 註冊一個帳號」如果不是這個 case 要測的重點，它是 precondition。

原則：測試重點以外的所有狀態，都是 precondition。

## Test Data

寫死，不要寫類別。

不合格：

- 帳號：一個合法帳號
- 密碼：一個合法密碼

合格：

- 帳號：`qa-test-001@example.com`
- 密碼：`Test@123456`

用「合法帳號」這種描述，不同人會準備不同資料，結果不會一致。
Test data 要能 copy-paste。

## Steps 與 Expected Result 一對一

```
Step                              Expected Result
1. 開啟首頁                        1. 看到 Login 按鈕在右上
2. 點 Login                        2. 跳出 Login Modal
3. 輸入 qa-test-001@example.com    3. Email 欄位顯示輸入值
4. 輸入錯誤密碼 wrong123            4. Password 欄位顯示為遮罩
5. 點「登入」                       5. 顯示紅色錯誤「帳號或密碼錯誤」，
                                      且停留在 Login Modal
```

不要把 5 個 expected 全擠在最後一行，那樣 fail 的時候看不出來是哪一步壞了。

---

## 常見錯誤

- [ ] **把多個情境塞進同一個 case** — 「測 login 各種情況」會變一坨。每個情境一個 case
- [ ] **Steps 寫得像功能說明** — 「系統應該驗證 email 格式」是 spec，不是 step。
      Step 是「使用者輸入 `abc` 並點下一步」
- [ ] **Expected 寫「正確」** — 不可量測。要寫「畫面跳轉到 /dashboard」、「右上角顯示使用者名稱」
- [ ] **太依賴前一個 case** — 一個 fail 就連環倒。除非真有 dependency，每個 case 都該能獨立跑
- [ ] **不留 spec 版本** — 回歸時發現 case 對不上，是因為 spec 改了沒同步。
      在 case 末尾加 `Spec ref: confluence/login-v2`

---

## 工具差異不大

| 工具 | 適合 |
|------|------|
| Excel / Sheets | 起步最快，但難追 history、難跨 case 引用 |
| Notion | 小團隊，搜尋好用 |
| TestRail / Xray / Zephyr | 中大型團隊，能跑 test run、出報表 |
| JSON / Markdown + Git | 自動化與手動混合的團隊，diff 看得到改了什麼 |

工具決定樣式，內容結構不變。先把 7 個欄位寫好，換工具只是 copy-paste。

---

→ 深入說明：[Test Case 撰寫範本 — 從 Title 到 Expected Result 的 7 個欄位](https://9niche.com/guide/test-case-template/)
