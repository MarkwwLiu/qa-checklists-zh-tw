# API Spec Review Checklist — 30 題

API spec 不能套[通用清單](README.md)。
Endpoint contract、錯誤碼、版本控制、idempotency 是 API 獨有的痛點。

會議前 20 分鐘掃完這 30 題。

---

## Contract（5 題）

- [ ] 每個 endpoint 的 HTTP method 對嗎？（GET / POST / PUT / PATCH / DELETE 各有語意）
- [ ] Path 符合 REST 慣例？`/users/{id}/orders` 不是 `/getUserOrders?uid=`
- [ ] Request schema 含每欄型別、必填、長度、格式？
- [ ] Response schema 含 success 與 error 兩種？
- [ ] 巢狀物件結構合理？（flat vs nested 的取捨）

### 太薄的 spec 長這樣

```yaml
POST /orders
Request: { items, total }
Response: { id, status }
```

該追問的：`items` 是什麼結構？`total` 包稅嗎？幣別？`id` 是 int 還 string？
`status` 有哪些可能值？

### 寫到這個程度才夠

```yaml
POST /api/v1/orders
Request:
  Content-Type: application/json
  Body:
    items:
      type: array
      minItems: 1
      maxItems: 50
      item:
        sku: string (3-20 chars, /^[A-Z0-9-]+$/)
        qty: integer (1-99)
    payment_method: enum [credit_card, paypal, apple_pay]
    coupon_code: string (optional, 6-12 chars)
    shipping_address: <ShippingAddress object>
Response 201:
  id: string (UUID v4)
  status: enum [pending, confirmed]
  total: number (含稅, TWD)
  estimated_delivery: string (ISO 8601)
```

## Error（6 題）

- [ ] 列出每個 endpoint 可能的 HTTP status？（200 / 201 / 400 / 401 / 403 / 404 / 409 / 422 / 429 / 500）
- [ ] 錯誤 payload 統一格式？還是各 endpoint 自己一套？
- [ ] 錯誤碼有完整列表？`ERR_INSUFFICIENT_BALANCE` 還是 `error.code = 1042`？
- [ ] 錯誤訊息多語系策略？後端傳 code 還是已翻譯字串？
- [ ] 4xx 與 5xx 的邊界清楚？
- [ ] Retry 策略寫了嗎？哪些錯能 retry，哪些不行？

### 建議的錯誤 payload

```json
{
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Your balance is too low for this purchase",
    "details": {
      "current_balance": 100,
      "required": 500,
      "currency": "TWD"
    },
    "trace_id": "abc-123-def",
    "doc_url": "https://docs.example.com/errors/INSUFFICIENT_BALANCE"
  }
}
```

`code` 給程式判斷、`message` 給開發 debug、`details` 給 UI 顯示、
`trace_id` 給 support 查、`doc_url` 給串接的人自助。整個 API 共用同一個格式。

### HTTP status 對照（最常被搞混的部分）

| Status | 正確用法 | 常見誤用 |
|--------|---------|---------|
| 400 Bad Request | Request 格式錯（JSON 壞、missing required） | 拿來表達「業務邏輯失敗」 |
| 401 Unauthorized | 沒帶 token / token 無效 | 拿來表達「沒權限」 |
| 403 Forbidden | 有身份但沒權限 | 拿來表達「token 過期」 |
| 404 Not Found | 資源不存在 | 代替 403（為防資訊洩漏可以，但要寫在 spec 裡） |
| 409 Conflict | 業務狀態衝突（重複建立、狀態不允許） | 用 400 表達 |
| 422 Unprocessable | Schema 對但業務驗證 fail | 用 400 表達 |
| 429 Too Many Requests | Rate limit | — |
| 500 Internal | Server bug | 業務邏輯錯誤拿 500 當擋箭牌 |

spec review 時看到「所有錯誤都回 400」就舉手。

## Auth（5 題）

- [ ] 認證方式明確？Bearer JWT / OAuth / API key / Session cookie
- [ ] Token 怎麼取得？怎麼 refresh？TTL 多久？
- [ ] 每個 endpoint 需要什麼 scope / role，有列表嗎？
- [ ] 多租戶隔離怎麼做？Tenant ID 放 token、URL、還是 header？
- [ ] Service-to-service 怎麼認證？跟使用者 API 同一套 token 嗎？

常見漏洞：

- **Token 過期沒有明確錯誤**：都回 401，客戶端分不出該 refresh 還是重新登入
- **403 與 404 混用**：有時 403 有時 404，前端無法寫統一處理
- **跨租戶資料**：`GET /users/{id}` 沒驗 caller 與 user 是否同租戶。這是資安問題，不是設計問題

## Versioning（5 題）

- [ ] 版本怎麼標？URL（`/v1/`）、Header（`API-Version: 2`）、Accept（`application/vnd.example.v1+json`）
- [ ] 什麼算 breaking change，有定義嗎？
- [ ] Breaking change 的廢棄流程？deprecation period 多久？怎麼通知？
- [ ] Backward compatibility 保證多久？
- [ ] 新欄位怎麼加？預設值？optional？

### 怎麼判斷 breaking

| 改動 | 是否 breaking |
|------|-------------|
| 加新欄位 | 否，客戶端可忽略 |
| 改欄位型別 | 是，必須 deprecate |
| 改 enum 值 | 是，必須 deprecate |
| 刪欄位 | 是，必須 deprecate |
| 改錯誤碼 | 看客戶端是否依賴字串，多半算 breaking |
| 加新 endpoint | 否 |

### Deprecation 的標準節奏

```
v1 上線
v2 上線（v1 仍可用，response 加 Deprecation header）
v1 六個月後正式下架
```

API spec 沒寫 deprecation policy，等於沒有 versioning。

## Performance（5 題）

- [ ] 有 rate limit 嗎？多少 req/sec/user/IP？429 回什麼 payload？
- [ ] List endpoint 有分頁嗎？cursor-based 還是 offset？最大 page size？
- [ ] Cache header 怎麼設？`Cache-Control`、`ETag`、`Last-Modified`
- [ ] 大檔案上傳 / 下載？streaming？multipart？S3 pre-signed URL？
- [ ] 同步還是非同步？long-running task 要不要拆成 job + status endpoint？

### Pagination 怎麼選

| 方式 | 優點 | 缺點 |
|------|------|------|
| Offset（`?page=2&size=20`） | 簡單、可跳頁 | 資料新增時頁面會位移 |
| Cursor（`?after=abc&size=20`） | 穩定、適合 infinite scroll | 不能跳頁 |
| Token | 後端可控、cursor 可加密 | 客戶端不能自己解 |

使用者 feed 用 cursor、後台管理頁用 offset。spec 沒寫就追問。

## Data / Idempotency（4 題）

- [ ] POST 是 idempotent 嗎？同一份 payload 打兩次會建兩筆？怎麼防？
- [ ] 時區處理？API 用 UTC 還是 local？輸入接受哪些格式？
- [ ] 敏感資料（email、phone、身分證）masked 還是明碼？日誌會印嗎？
- [ ] 欄位 nullable 規則？null、missing、empty string 三者分得清楚嗎？

### Idempotency 的標準做法

```
POST /orders
Headers:
  Idempotency-Key: <UUID>   # 客戶端產生

Server 行為:
  - 第一次：建單、儲存 (Idempotency-Key, order_id)
  - 重複請求：回傳一樣的 order_id、不重複建單
  - Key TTL：24 hr（避免無限儲存）
```

spec 沒寫 Idempotency-Key 流程，POST 重複扣款只是時間問題。

---

## 用 OpenAPI 的話，很多題自動被逼著回答

OpenAPI 3.0 有結構化欄位強迫你填：`required`、`enum`、`format`、`pattern`、
`minimum`/`maximum`、`minLength`/`maxLength`。

對 QA 的額外好處：

- `openapi-generator` 自動生 client，用來對 contract
- `Dredd` / `Schemathesis` 自動測 contract，抓 spec 與實作的差異
- Swagger UI / Redoc 讓 PM 跟前端都能 review

沒用 OpenAPI 的話，這 30 題就得人工問。

## API spec 的警訊

看到這幾句話要提高警覺：

1. **「依前端需求調整」** — 沒設計，見步行步，最後變亂麻
2. **「跟 v1 一樣」** — v1 漏掉的東西原封不動繼承到 v2
3. **「等開發完再寫 doc」** — 不會寫的
4. **「Postman collection 就是文件」** — Postman 不是 contract spec
5. **錯誤碼用整數** — `error_code: 1042` 沒可讀性、難維護
6. **欄位命名混亂** — `user_id` / `userId` / `uid` 三種寫法同時出現
7. **時間用 unix timestamp** — 客戶端跨時區算錯，該用 ISO 8601 string

---

→ 深入說明：[API Spec Review Checklist — 30 個問題抓住 Contract / Error / Version 漏洞](https://9niche.com/guide/api-spec-checklist/)
