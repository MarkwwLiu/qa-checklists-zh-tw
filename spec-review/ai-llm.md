# AI / LLM 功能 Spec Review Checklist

PM 在 spec 上寫「加一個 AI 助手」，然後就沒有了。

AI 功能的 spec 不能寫「要正確」，要寫「成功率達到 X%、失敗時做 Y」。
一般 spec 清單處理不了這件事，因為它假設「輸入確定 → 輸出確定」。

| 一般功能 | AI 功能 |
|---------|--------|
| 輸入確定 → 輸出確定 | 輸入確定 → 輸出是機率分佈 |
| 可重現 | 每次可能不同 |
| 二元對錯 | 品質光譜 |
| — | 會幻覺、會有 bias |
| 成本固定 | 成本隨用量跑 |
| — | 有法遵風險 |

下面 8 個必問，每個再延伸 3 題，一個 AI 功能的 spec review 大概 24 題。

---

## 1. 評估指標

- [ ] Eval set 有嗎？多少個範例？
- [ ] 通過標準是什麼？用什麼方法評（automated metric / LLM-as-judge / human）？
- [ ] 改 prompt 後的 regression 門檻？

**不能接受的寫法**：「AI 要回答正確」、「AI 不能說錯話」

**該寫成這樣**：

```
Eval set 大小: 200 個範例
Pass 條件: >= 90% 通過 LLM-as-judge
Human review: 每月抽 50 個、QA 評分 >= 4/5
Regression: 改 prompt 後 eval set 重跑、分數不能降超過 2%
```

四種評估方式搭配用：

| 方式 | 用途 |
|------|------|
| Eval set（100+ 黃金標準範例） | 基準線，每次改動都要跑 |
| Automated metrics（BLEU / ROUGE / embedding 相似度） | 便宜、可 CI、但跟人的判斷有落差 |
| LLM-as-judge | 用另一個 model 評分，涵蓋 automated metric 抓不到的語意 |
| Human review（抽 5–10%） | 看趨勢用，也是校準 LLM-as-judge 的依據 |

## 2. 失敗 Fallback

- [ ] LLM call timeout 的處理？retry 幾次？
- [ ] 全部失敗時使用者看到什麼？
- [ ] 高風險問題（健康 / 法律 / 金融）要不要直接 escalate 人工？

```
LLM call timeout (15s): 自動 retry 1 次
全部失敗: 回「目前無法回答，請聯絡客服」+ 記 log
模型明確表示不確定: 顯示「我不確定，建議改問真人」
高風險問題: 直接 escalate 人工
```

Fallback 的五個層級，spec 至少要講到第三層：

1. 重試 N 次
2. 換 model（主力掛了用備援）
3. 回傳「請稍候」或範本回覆
4. 人工 escalate
5. 預設答案

## 3. 幻覺處理

幻覺（編造事實）無法 100% 消除，只能降低加偵測。spec 要寫的是偵測機制。

- [ ] 事實聲明要不要 cite source？
- [ ] 含日期 / 數字 / 名稱的回應要不要對知識庫驗證？
- [ ] UI 上有沒有「AI 生成，可能有誤」的提示？

```
RAG 模式: 每個事實聲明必須 cite source，無 source 不回答
Fact check: 含日期 / 數字 / 名稱的回應要對知識庫驗
UI 警告: 「AI 生成、可能有誤」一律顯示
高風險 domain: 必須 cite + 人工 review
```

## 4. Prompt 管理

Prompt 改動就是 code 改動，要走一樣的流程。

- [ ] Prompt 存在哪？有 version control 嗎？
- [ ] 改 prompt 要不要 review？要不要先 A/B？
- [ ] Rollback 要多久？

```
Prompt 存 Git: 每次改有 commit + reviewer
Prompt version: 帶在每次 LLM call、log 記版本
A/B test: 新 prompt 先 5% 流量、看 7 天 metric
Rollback: 一鍵切回舊 prompt、< 30 秒
```

「prompt 寫在 code 裡、改了直接 deploy」是最常見的狀況，也是最常出事的。

## 5. 模型選擇

| 維度 | 該問 |
|------|------|
| Primary | 用哪個 model？ |
| Backup | Primary 掛了切哪個？切換是自動還是手動？ |
| Cost limit | 每次 call 的預算上限 |
| Latency | p95 要求多少？ |
| 升級策略 | 新版 model 出來時的 A/B 流程 |

## 6. 成本控制

- [ ] 每個使用者的 rate limit？
- [ ] 有沒有 cache？
- [ ] 整體支出超標時怎麼辦？
- [ ] Prompt 大小上限？

```
Per-user rate limit: 100 calls/day（free）/ 1000 calls/day（paid）
Cache: 完全相同的問題 24h 內回 cache
Cost alert: 整體每日支出 > $X 自動 throttle
Prompt 大小: 上限 4K token、超過砍掉舊 history
```

沒有 cost cap 的 AI 功能，上線第一週就可能燒掉整季預算。

## 7. 護欄 Guardrails

```
禁忌主題: 醫療診斷 / 法律建議 / 投資建議 → 拒絕並建議找專業人士
Prompt injection: 偵測 "ignore previous instructions" 這類模式、拒絕
個資輸出: 偵測 email / 電話 / 信用卡號、mask 後再回
冒充: 不能聲稱自己是真人、必須說明是 AI
```

護欄分三段，spec 要三段都寫：

| 位置 | 要做的事 |
|------|---------|
| 輸入端 | prompt injection 偵測、禁忌字過濾、個資偵測、地區法律限制 |
| 輸出端 | bias 過濾、暴力 / 性 / 仇恨內容檢查、toxic 分類器、必要時 refusal |
| 後處理 | citation 強制、disclaimer 加註、audit log |

## 8. 法遵

```
AI 用途分類: 是否屬 EU AI Act 的「高風險」？
Audit log: 保留 user / prompt / response / decision，保存 3 年
Right to explain: 使用者可以問「為什麼是這個結果」
資料保留: LLM provider 會不會拿我們的資料去訓練？合約怎麼寫？
被遺忘權: 使用者刪帳號後，相關資料怎麼處理？
```

台灣團隊額外要看個資法，金融 / 醫療領域還有金管會、健保相關規範。

---

## 實際跑一次

### Spec 原文

```
新增 AI 客服助手
- 使用者問問題、AI 回答
- 整合 GPT-4
- 期望 80% 問題自動回答
```

### 用 8 必問掃出來

```
1. 評估: 80% 怎麼量？是哪 200 個 eval question？誰定義「答對」？
2. Fallback: 答不出來的那 20% 怎麼處理？
3. 幻覺: 客戶問退款政策，AI 編造一套怎麼辦？
4. Prompt: 改一次 prompt 怎麼測影響？誰 review？
5. 模型: GPT-4 掛了用什麼？切換要多久？
6. 成本: 每個使用者一個月平均花多少？上限在哪？
7. 護欄: 使用者罵髒話 / 問醫療問題怎麼處理？
8. 法遵: 對話 log 存多久？客戶可以要求刪除嗎？
```

PM 多半答不出來，spec 退回重寫。這就是 review 的目的。

---

## AI 功能的 Definition of Ready

六項全達標才能進 sprint：

- [ ] Eval set 已建立（>= 100 範例）
- [ ] Fallback 流程已定義
- [ ] Prompt versioning 機制已就緒
- [ ] Cost cap 明確
- [ ] Guardrails 列表已列
- [ ] 法遵 review 已過

## 反模式

- 「AI 一定要 100% 正確」
- 沒有 eval set，靠直覺改 prompt
- Prompt 寫在 code 裡，沒有 versioning
- 沒有 fallback，AI 掛掉整個功能跟著掛
- 沒有成本上限
- 不做 guardrail，等著被 prompt injection
- 不寫法遵，等著被罰

## 五句話

1. AI 功能 spec 不能寫「正確」，要寫成功率
2. 沒 eval set 等於沒 spec
3. Prompt 改動就是 code 改動，要走 review
4. Fallback 比 happy path 重要十倍
5. 法遵問題 spec 沒寫，公司賠錢

---

→ 深入說明：[AI / LLM 功能 Spec Review — 幻覺 / 評估 / 成本 / 法遵 8 個必問](https://9niche.com/guide/ai-feature-spec-review/)

延伸：[用 LLM 跑 Spec Review](https://9niche.com/guide/llm-spec-review/)、[AI 共存的 QA 工具箱](https://9niche.com/guide/ai-toolkit-for-qa/)
