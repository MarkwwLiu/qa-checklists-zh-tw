# Test Case 範本

欄位說明見 [README.md](README.md)。

## 空白範本

```
ID: TC-<MODULE>-<NNN>
Title: <前置條件>，<操作>，<預期>
Priority: P0 / P1 / P2
Spec ref: <spec 連結或版本>

Preconditions:
  - <狀態 1>
  - <狀態 2>
  - 系統設定：<flag / 參數>

Test Data:
  - <欄位>: <寫死的值>
  - <欄位>: <寫死的值>

Steps                           Expected Result
1. <操作>                        1. <可觀察的結果>
2. <操作>                        2. <可觀察的結果>
3. <操作>                        3. <可觀察的結果>
```

## 填好長這樣

```
ID: TC-LOGIN-007
Title: 帳號鎖定後重試 — 連續輸錯 5 次後第 6 次應顯示鎖定訊息
Priority: P0
Spec ref: confluence/login-v2

Preconditions:
  - 已存在帳號 qa-test-007@example.com，狀態為 active
  - 該帳號未曾被鎖定
  - 系統設定：MAX_FAIL_ATTEMPTS=5、LOCK_DURATION=15min

Test Data:
  - email: qa-test-007@example.com
  - 正確密碼: Test@123456
  - 錯誤密碼: wrong000

Steps                                Expected Result
1. 開啟 /login                        1. Login 表單可見
2-6. 用 wrong000 連續登入 5 次         2-6. 每次顯示「帳號或密碼錯誤」
7. 第 6 次仍用 wrong000                7. 顯示「帳號已鎖定，15 分鐘後重試」
                                         且按鈕變灰
8. 此時改用正確密碼 Test@123456         8. 仍顯示「帳號已鎖定」（不應放行）
9. 等待 15 分鐘                        9. —
10. 再以正確密碼登入                   10. 登入成功進入 /dashboard
```

第 8 步是這份 case 的重點：鎖定期間就算密碼對也不能放行。
只寫到第 7 步的 case，測不出最容易實作錯的那一段。

## 表格版（給 Excel / Sheets / TestRail）

| ID | Title | Priority | Preconditions | Test Data | Step | Expected Result |
|----|-------|----------|---------------|-----------|------|-----------------|
| TC-LOGIN-007 | 帳號鎖定後重試 | P0 | 帳號 active、未曾鎖定、MAX_FAIL_ATTEMPTS=5 | qa-test-007@example.com / wrong000 | 1. 開啟 /login | Login 表單可見 |
| | | | | | 2-6. 用 wrong000 登入 5 次 | 每次顯示「帳號或密碼錯誤」 |
| | | | | | 7. 第 6 次仍用 wrong000 | 顯示「帳號已鎖定，15 分鐘後重試」，按鈕變灰 |
| | | | | | 8. 改用正確密碼 | 仍顯示「帳號已鎖定」 |
| | | | | | 9. 等待 15 分鐘 | — |
| | | | | | 10. 以正確密碼登入 | 進入 /dashboard |

---

## 寫完自己檢查

- [ ] Title 唸出來別人聽得懂？
- [ ] Preconditions 裡沒有混進該測的操作？
- [ ] Test Data 全部寫死，可以 copy-paste？
- [ ] 每個 step 都有對應的 expected？
- [ ] Expected 都是可觀察的（而不是「正確」「成功」）？
- [ ] 這個 case 可以獨立跑，不依賴前一個？
- [ ] 有標 spec ref？

---

→ 深入說明：[Test Case 撰寫範本 — 從 Title 到 Expected Result 的 7 個欄位](https://9niche.com/guide/test-case-template/)
