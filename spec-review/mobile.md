# Mobile App Spec Review Checklist — 25 題

「Web 都會寫 spec review 了，Mobile 應該差不多。」

不一樣。Mobile 多出九個維度：權限、推播、deep link、offline、跨平台一致性、
OS 碎片、電量、網路、store 審核。每一個都能自己炸出三個 bug。

---

## A. 權限（5 題）

- [ ] **首次觸發時機**：什麼時候彈權限對話框？
- [ ] **拒絕之後的 fallback**：使用者選「拒絕」後 UX 怎樣？App 還能用嗎？
- [ ] **「不要再問」處理**：iOS 第三次會自動拒絕，需要引導去 Settings
- [ ] **權限說明文案**：iOS 強制要寫 `NSPhotoLibraryUsageDescription` 這類文案，寫了嗎？
- [ ] **請求前的理由解釋**：彈視窗前先解釋為什麼需要，否則使用者多半直接拒絕

權限流程至少有四條分支，spec 只寫「允許」那一條就是沒寫完：

```
觸發功能 → 需要權限？
            ├─ 允許        → 正常流程
            ├─ 拒絕        → fallback 流程（spec 常漏）
            └─ 不要再問     → 引導去 Settings（spec 幾乎都漏）
```

## B. 推播（3 題）

- [ ] **三種 App 狀態**：foreground / background / killed 收到推播分別怎麼處理？
- [ ] **推播裡的 deep link**：點推播打開 App 後跳哪個畫面？
- [ ] **訂閱 token 管理**：使用者改帳號、登出時 token 怎麼處理？

登出沒清 token，前一個使用者的推播會送到下一個使用者手上。這題一定要問。

## C. Deep Link / Universal Link（3 題）

- [ ] **哪些 URL pattern 該開 App？**
- [ ] **App 沒安裝時的 fallback**：開 App Store 還是開 web？
- [ ] **登入狀態判斷**：deep link 進入時還沒登入，先跳 login 再回來，還是直接擋？

## D. Offline（3 題）

- [ ] **完全離線時哪些功能可用？**
- [ ] **資料同步策略**：重新上線後怎麼處理衝突？
- [ ] **網路恢復通知**：怎麼告訴使用者「現在可以送出了」？

## E. 跨平台一致性（3 題）

- [ ] **iOS 與 Android UI 差異**：原生元件不同（picker、navigation bar），spec 有寫差異嗎？
- [ ] **手勢一致性**：左滑返回是 iOS native 行為，Android 沒有
- [ ] **平台專屬功能**：iOS Face ID、Android 多視窗，各自要不要做？

## F. OS 升級與碎片（3 題）

- [ ] **支援最低 OS 版本**：iOS 15+？Android 9+？怎麼決定的？
- [ ] **新 OS 升級的影響**：下一版 iOS 出來會壞掉什麼？
- [ ] **OEM 行為差異**：Samsung 的「電池優化」會殺背景行程，怎麼處理？

## G. 電量與網路（2 題）

- [ ] **背景處理的電量成本**：地點追蹤、推播、健康資料，背景跑多少？
- [ ] **慢網路體驗**：4G、高鐵上、飛行模式，fallback UX 是什麼？

## H. App Store 審核（2 題）

- [ ] **隱私 manifest**：Apple 要求 `PrivacyInfo.xcprivacy`，這次改動有更新嗎？
- [ ] **App Tracking Transparency**：iOS 14+ 追蹤前要先問，有彈嗎？

## I. 更新策略（1 題）

- [ ] **強制更新還是軟更新**：critical bug 怎麼辦？rollback 怎麼做？

---

## 實際跑一次給你看

### Spec 原文（典型寫法，看起來好像寫完了）

```
功能：使用者上傳大頭照
- 使用者點 profile 頁的相機 icon
- 開啟相機選擇照片
- 上傳到伺服器
- 顯示新照片
```

### 用上面的清單掃出來的問題

| # | 問題 | 來自哪一類 |
|---|------|-----------|
| 1 | 「相機 icon」是 iOS 還是 Android 樣式？跨平台一致嗎？ | E |
| 2 | 點 icon 出現「相簿 / 拍照」選單還是直接開相機？ | E |
| 3 | 第一次點要彈 `NSCameraUsageDescription`，文案是什麼？ | A |
| 4 | 使用者拒絕相機權限 → fallback 是只開放相簿，還是整個功能 disable？ | A |
| 5 | 相簿權限也被拒絕 → 怎麼處理？ | A |
| 6 | iOS 14+ 允許「只選特定照片」，這條路徑的 UX 一樣嗎？ | A |
| 7 | 照片大小限制多少？超過怎麼處理？ | — |
| 8 | 上傳中網路斷掉怎麼處理？ | D |
| 9 | 上傳中切到背景、回前景能繼續嗎？ | — |
| 10 | 要不要推播「上傳完成」？ | B |
| 11 | Offline 時能不能先存本機、之後同步？ | D |
| 12 | 從 deep link 直接開 profile 頁再點相機，流程一樣嗎？ | C |
| 13 | iOS 與 Android 的照片 EXIF 旋轉處理一致嗎？ | E |
| 14 | 上傳失敗的 retry 策略？ | — |
| 15 | 需要重新 Face ID 驗證嗎？ | E |
| 16 | Android 不同廠商的相機 app 行為一致嗎？ | F |
| 17 | 隱私 manifest 要更新嗎？ | H |

問完這 17 題，PM 才會發現這個功能其實沒寫完 spec。

---

## 三類 App 要 review 的東西不一樣

| 類型 | Review 重點 |
|------|-----------|
| Native iOS / Android | 平台一致性要另外談，實務上需要兩份 spec 段落 |
| Cross-platform（RN / Flutter） | 同一份 spec，但平台差異要附錄列出 |
| Hybrid WebView | 權限透過 WebView 取得，OS feature 受限，要先確認能不能做 |

## QA 工具地圖

| 工具 | 用途 |
|------|------|
| Firebase Analytics | 看使用者的 OS / 裝置分佈，決定支援範圍 |
| Sentry / Bugsnag | Crash 報告 |
| TestFlight | iOS beta |
| Firebase App Distribution | Android / iOS beta |
| BrowserStack | 跨真機測試 |
| Charles Proxy | 攔網路封包 debug |

## Mobile spec 的反模式

- 完全照 Web spec 寫
- 忽略權限拒絕分支
- 推播沒寫 deep link 目的地
- 不寫 offline 行為
- 不指定最低 OS 版本
- 不分平台差異
- 不寫 Store 審核要求

## 五句話

1. 每個功能至少問 5 個權限 / 網路 / OS 相關的問題
2. iOS 跟 Android 要有各自的 spec 段落
3. TestFlight beta 階段是 spec review 的第二次機會
4. Store 審核被拒，九成是 spec 漏掉的事
5. OS 升級永遠在打破你的 spec

---

→ 深入說明：[Mobile App Spec Review — 比 Web 多 5 倍痛點的審查 checklist](https://9niche.com/guide/mobile-app-spec-review/)

延伸：[Mobile App Testing 入門（Appium vs Detox）](https://9niche.com/guide/mobile-testing-appium-detox/)
