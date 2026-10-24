# Microservices Contract Review

微服務最大的痛不是寫不出來，是「服務 A 改了但 B 不知道」。

單體 app review 一個 repo、一份 spec，編譯時就抓得到不一致。
微服務是 N 個 team、N 份 spec、獨立發版，錯誤要等 runtime 才浮出來。

單一服務的 API spec 用 [API 清單](api.md)；跨服務的契約用這份。

---

## 8 個典型漏洞

### 1. Schema 漂移

Provider 加了一個 nullable 欄位，沒通知 consumer，直接 deploy：

```
Provider: 加新欄位 verified_at (nullable)
Consumer: GET /users/123
Response: {"id":123, "verified_at": null}
Consumer code: 💥 Cannot read property of null
```

Review 該問：

- [ ] 新增欄位有 nullable / default 嗎？
- [ ] Consumer 端有 null check 嗎？
- [ ] Schema 變更的通知流程是什麼？

### 2. Enum 擴張

```typescript
// Provider 加新狀態
type OrderStatus = 'pending' | 'paid' | 'shipped' | 'cancelled' | 'refunded'; // refunded 是新的

// Consumer 寫法
switch (status) {
  case 'pending':   return '處理中';
  case 'paid':      return '已付款';
  case 'shipped':   return '已出貨';
  case 'cancelled': return '已取消';
  // refunded 沒處理 → undefined
}
```

Review 該問：

- [ ] 新增 enum 值有對齊 consumer 嗎？
- [ ] Consumer 有 default case 嗎？
- [ ] Enum 擴張算不算 breaking change？團隊有分類標準嗎？

### 3. 版本不同步

```
Service A (v1) ---calls---> Service B (v2)
                              ^
                         v1 早就 deprecate
                         但 Service A 沒升
```

Review 該問：

- [ ] 版本 deprecation policy 是什麼？
- [ ] 有多少 consumer 還在用 v1？查得到嗎？
- [ ] Migration 有強制 deadline 嗎？

### 4. 錯誤碼變了

```
Provider v1: error.code = "INSUFFICIENT_BALANCE"
Provider v2: error.code = "INSUFFICIENT_FUNDS"   ← 改了

Consumer:
if (err.code === "INSUFFICIENT_BALANCE") { /* 處理 */ }
// v2 之後永遠跑不到這裡，而且不會報錯
```

Review 該問：

- [ ] 錯誤碼有 changelog 嗎？
- [ ] 改錯誤碼算 breaking change 嗎？

這類 bug 特別陰險：不會 throw，只會靜悄悄走錯分支。

### 5. Timeout 不一致

```
User → API Gateway (60s timeout)
       → Service A (10s)
         → Service B (30s)
           → DB (5s)
```

Service B 設 30s，但 A 只等 10s。B 跑到第 11 秒的工作全都白做，
使用者還是等到 60s 才看到錯誤。

Review 該問：

- [ ] 每層 timeout 是否往下 propagate？
- [ ] 有沒有統一的 deadline 概念？
- [ ] Retry 策略由誰負責？

### 6. Retry 衝突

Client 設 retry 3 次，Service A 也設 retry 3 次：

```
Client → A: POST /order (retry 3)
A → B: createOrder (retry 3)
B 慢 → A timeout → A retry 2 次
Client 也 timeout → Client retry → A 再打 B 三次
結果: 訂單建了 6 次
```

Review 該問：

- [ ] 哪一層負責 retry？（一條鏈只能一個地方做）
- [ ] 有 idempotency key 嗎？
- [ ] 有 exponential backoff 嗎？

### 7. 序列化版本差異

| 欄位 | Provider 送 | Consumer 期待 |
|------|-----------|-------------|
| `created_at` | `1734567890`（Unix sec） | `2026-06-13T10:00:00Z`（ISO 8601） |
| `price` | `99` | `99.00` |
| `status` | `"paid"` | `1`（整數） |

Review 該問：

- [ ] 日期一律 ISO 8601 嗎？
- [ ] 數字 precision 有規範嗎？
- [ ] Enum 傳 string 還是 int？

### 8. 訊息順序（async）

```
Producer 順序送:
  msg1: order.created
  msg2: order.paid
  msg3: order.shipped

Consumer 收到:
  msg3（網路剛好快）
  msg1
  msg2

Consumer 處理 msg3 時 → 「order 不存在」
```

Review 該問：

- [ ] 訊息有 sequence number 嗎？
- [ ] Consumer 有 idempotency 嗎？
- [ ] Out-of-order 的容錯機制是什麼？

---

## Consumer-Driven Contract Testing

上面八個漏洞有一半能靠 CDC 在 CI 階段擋掉。

Consumer 先寫下自己的期望，產出 pact file；Provider 在 CI 跑 verify。
Provider 改了 schema 不符合期望，build 直接紅燈。

```javascript
// Consumer 端：寫期望
import { Pact } from '@pact-foundation/pact';

const provider = new Pact({
  consumer: 'web-app',
  provider: 'user-service',
});

await provider.addInteraction({
  state: 'user 123 exists',
  uponReceiving: 'a request for user 123',
  withRequest: { method: 'GET', path: '/users/123' },
  willRespondWith: {
    status: 200,
    body: { id: 123, email: 'a@b.com' },
  },
});
```

```bash
# Provider 端：verify 跑 pact file
pact-verifier --provider-base-url http://localhost:8080 \
              --pact-url ./pacts/web-app-user-service.json
```

關鍵在於期望由 consumer 定義。Provider 自己寫的測試，永遠測不到
「consumer 以為你會回什麼」。

---

## 跨服務 Spec Review 清單

### Contract
- [ ] 欄位增刪有通知流程？
- [ ] Enum 擴張有對齊 consumer？
- [ ] Schema 嚴格度（額外欄位是忽略還是報錯）？

### Versioning
- [ ] 這次改動算 breaking 嗎？判準是什麼？
- [ ] Deprecation period 多久？
- [ ] Migration plan 有 deadline 嗎？

### Error
- [ ] 錯誤碼穩定嗎？改動有 changelog？
- [ ] 4xx 與 5xx 的分類一致嗎？
- [ ] Error payload 結構跨服務統一嗎？

### Timeout
- [ ] 每層 timeout 數字列出來了嗎？
- [ ] Deadline 有往下傳嗎？
- [ ] 有 circuit breaker 嗎？

### Retry
- [ ] 哪一層負責？
- [ ] Idempotency key 怎麼產生？
- [ ] Exponential backoff 參數？

### Async
- [ ] 訊息格式有 schema registry 嗎？
- [ ] 順序保證到什麼程度？
- [ ] 重複處理的防護？

### Observability
- [ ] Trace ID 跨服務傳遞？
- [ ] 每一跳都有 log？
- [ ] Metric 有暴露出來？

### Auth
- [ ] Service-to-service 認證方式？
- [ ] Token TTL？
- [ ] Permission 怎麼往下傳？

---

## 工具

| 工具 | 用途 |
|------|------|
| Pact | REST + async contract test |
| Schemathesis | OpenAPI fuzz testing |
| Dredd | OpenAPI 與實作比對 |
| GraphQL Inspector | GraphQL schema diff |
| Postman Mocks | 模擬 provider |
| WireMock | 自架 mock server |

## 反模式

- 每次改 schema 不通知
- Consumer 跟 provider 放同一個 repo，掩蓋了版本問題
- 每一層都做 retry
- 不寫 deprecation policy
- 錯誤碼想改就改
- Timeout 各服務各自設
- 沒有 trace ID

## 五句話

1. 微服務的 bug 不在單一服務，在介面
2. Contract test 為主、Integration test 為輔
3. 沒 deprecation policy 等於沒 versioning
4. 一條呼叫鏈只能有一個地方做 retry
5. Trace ID 不傳，debug 不可能

---

→ 深入說明：[Microservices Contract Review — 跨服務 API 不一致的 8 個典型漏洞](https://9niche.com/guide/microservices-contract-review/)
