# QA Checklists

PM 丟一句「使用者可以提現」，然後說 spec 寫好了。

這裡是一組能直接打勾的清單，幫你在開會前 15 分鐘，把那一句話拆成該問的問題。
全部繁體中文，技術術語保留英文。fork 下來改成你們團隊的版本就能用。

## 10 秒看懂這是什麼

Spec 原文只有一行：

> 使用者可以提現。

拿 [通用 Spec Review 清單](spec-review/README.md) 掃一遍，掃出來的問題長這樣：

| 清單類別 | 掃出來該問的問題 |
|---------|----------------|
| 前置條件 | 沒過 KYC 的使用者點下去會怎樣？按鈕不顯示、按鈕灰掉、還是跳提示？ |
| 輸入邊界 | 最低提現金額是多少？小數第三位四捨五入還是無條件捨去？ |
| 狀態機 | 「處理中」的提現可以取消嗎？取消後錢多久回到餘額？ |
| 錯誤處理 | 銀行 API timeout 之後，這筆算成功還是失敗？使用者再按一次會提兩次嗎？ |
| 非功能 | 提現紀錄要留幾年？使用者要求刪帳號時，這筆記錄刪不刪？ |

一行 spec 變成五個問題。這五個問題在 spec 階段問，各花 PM 兩分鐘；
等上線後才問，是一輪 hotfix 加一次對帳。

## 清單目錄

### Spec Review

| 清單 | 題數 | 什麼時候用 |
|------|------|-----------|
| [通用 Spec Review](spec-review/README.md) | 30 | 任何功能 spec，會議前掃一遍 |
| [API Spec Review](spec-review/api.md) | 30 | 後端給你 endpoint 規格時 |
| [Mobile App Spec Review](spec-review/mobile.md) | 25 | iOS / Android 功能，Web 清單救不了你 |
| [AI / LLM 功能 Spec Review](spec-review/ai-llm.md) | 24 | spec 上出現「加個 AI 助手」時 |
| [Microservices Contract Review](spec-review/microservices.md) | 8 類漏洞 | 跨服務 API、事件驅動架構 |

### 產出範本

| 範本 | 用途 |
|------|------|
| [Bug Report SOP](bug-report/README.md) | 8 個欄位，讓開發 30 秒看懂、不被退「無法重現」 |
| [Bug Report 範本](bug-report/template.md) | 可直接複製的 markdown |
| [Test Case 撰寫原則](test-case/README.md) | 7 個欄位，寫出別人照著跑結果會一樣的 case |
| [Test Case 範本](test-case/template.md) | 可直接複製的 markdown |

### 可直接安裝的 GitHub Issue Form

`.github/ISSUE_TEMPLATE/bug_report.yml` 是把 Bug Report SOP 的 8 個欄位做成的
GitHub Issue Form。複製到你的 repo：

```bash
mkdir -p .github/ISSUE_TEMPLATE
curl -o .github/ISSUE_TEMPLATE/bug_report.yml \
  https://raw.githubusercontent.com/<your-org>/<this-repo>/main/.github/ISSUE_TEMPLATE/bug_report.yml
```

推上去之後，開 issue 就會變成填表單，必填欄位沒填送不出去。

## 怎麼用

1. **會議前**：PM/PO 發 spec 後，自己對清單走一輪，把「不確定」「沒寫到」的點記下來。
2. **會議中**：用問題形式問。不要用「你這邊少了 XX」開頭，會撕破臉。
   問「未驗證 KYC 的使用者點下去會怎樣」比「你沒寫 KYC」有效十倍。
3. **會議後**：寄 follow-up，把當場澄清的點寫回 spec。沒寫回去的澄清，三週後會重新吵一次。

清單是 markdown checkbox，可以直接貼進 PR description、Jira ticket、Notion 頁面。

## 相關工具

<!-- TODO: 壓測工具連結 -->

## 關於作者

**Mark Liu** — QA Engineering Leader，10+ 年經驗。
專長 API 自動化測試、Spec Review、AI 輔助 QA。

這些清單是我在實際專案裡跑 spec review 用的固定問題集，陸續補了幾年。
更多 QA 相關的長文寫在 [9niche.com](https://9niche.com/guide/)。

→ 作者介紹：[Mark Liu](https://9niche.com/about/author/)

## License

[MIT](LICENSE) — 拿去改、拿去商用都可以，不用問我。

有踩到清單漏掉的坑，歡迎開 issue 或 PR，見 [CONTRIBUTING.md](CONTRIBUTING.md)。
