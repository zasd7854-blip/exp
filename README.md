# Chatbot 服務失敗與內疚式道歉：實驗網站

## 1. 這份檔案可以做什麼

這是**純 HTML / CSS / JavaScript** 的可執行研究情境網站，不需要 Python、Node.js 或資料庫；直接開啟 `index.html` 即可進行互動。

情境：受試者使用客服 Chatbot 預訂「北京 → 峇里島，2027 年 8 月 1 日」機票，第一次因 Chatbot 自身處理錯誤而失敗，Chatbot 道歉，使用者要求重試，Chatbot 完成訂票。**沒有真的連線航空公司，也不會產生實際訂單。**

兩個實驗條件只有一段道歉文字不同：

- **A 組（一般道歉）**：很抱歉，剛才我在處理您的訂票需求時發生錯誤，導致預訂未成功。請您再試一次。
- **B 組（內疚式道歉）**：很抱歉，剛才我在處理您的訂票需求時發生錯誤，導致預訂未成功。對於自己的這次失誤，我感到很內疚。請您再試一次。

兩組的聊天介面、按鈕、處理等待時間、使用者回覆、錯誤事件及重試成功內容皆相同。問卷變數（ABI、Chatbot 寬恕、持續使用意願、操弄檢定、注意力檢查）**不在本網站收集**，請放在 LimeSurvey。

## 2. 直接測試

1. 用 Chrome、Edge、Safari 或 Firefox **直接開啟 `index.html`**。不需要架設網站。
2. 網址沒有指定分組時，網站會自動以約 50:50 機率分配 A／B；同一瀏覽器分頁重新整理時分組不變。
3. 想分別看兩組，請在網址後加上 `?condition=neutral&preview=1` 或 `?condition=guilt&preview=1`。
4. 在測試模式可看到目前組別和「清除本次測試進度」按鈕；**正式受試者連結不要加 `preview=1`**。

建議研究者使用不同匿名測試編號，避免兩次測試共用進度，例如：

- `index.html?condition=neutral&rid=test-A-01&preview=1`
- `index.html?condition=guilt&rid=test-B-01&preview=1`

（上述是直接開啟本地檔案時網址列可以使用的查詢參數；實際正式使用應部署於 HTTPS 網站。）

## 3. 正式部署

將 `index.html` 上傳到支援靜態 HTML 的 HTTPS 網站，例如 GitHub Pages、Netlify 或學校主機。**純靜態站不會記錄作答資料，也不具備防止受試者偽造連結或重複參與的伺服器驗證機制。**

不要直接用 `file://` 本地網址發給受試者；他們無法存取你的電腦。

## 4. 與 LimeSurvey 銜接（建議操作）

**建議讓 LimeSurvey 執行受試者的正式隨機分組，並把組別以 URL 參數傳入本網站。**這樣組別與正式答卷可以在問卷系統內對應，不依賴本網站臨時抽籤。

例如假設網站部署網址是 `https://example.edu/chatbot/index.html`：

A 組實驗情境連結：

`https://example.edu/chatbot/index.html?condition=neutral&rid=匿名受試者代碼`

B 組實驗情境連結：

`https://example.edu/chatbot/index.html?condition=guilt&rid=匿名受試者代碼`

**請把「匿名受試者代碼」換成 LimeSurvey 產生的唯一代碼；不要使用姓名、電子郵件、學號。**

參考操作流程：

1. LimeSurvey 研究說明與知情同意。
2. LimeSurvey 隨機指定 A（neutral）或 B（guilt），並在答案資料中保存分組。
3. 顯示對應的網站連結／嵌入網站，由受試者完成互動。
4. **在完成互動之後**，受試者才能回答 ABI、寬恕與持續使用意願的量表。
5. 再放置內疚情緒操弄檢定、情境理解檢查、基本資料，完成問卷。

### 可選：完成後返回問卷

網站支援 `return` 參數。可把完成後應前往的網址先進行 URL 編碼，再放在連結中。例如：

`https://example.edu/chatbot/index.html?condition=guilt&rid=R001&return=https%3A%2F%2Fsurvey.example.edu%2Fcontinue`

完成訂票後才會出現「返回問卷，繼續填答」。系統會在返回網址加上：

- `exp_completed=1`
- `exp_condition=guilt` 或 `neutral`
- `exp_rid=...`（若有提供）
- `exp_duration_s=...`

**注意：這些網址參數不會自動寫入 LimeSurvey 答案，也不代表原本的作答工作階段一定能恢復。**返回網址必須是你已設定、測試過，能讓相同受試者繼續完成問卷的有效連結；若無法設定，建議在 LimeSurvey 的同一頁嵌入本網站，受試者完成後再操作問卷下一步。

### 可選：嵌入頁面完成通知

如果網站放在 iframe 中且帶入 `parent_origin=https%3A%2F%2Fsurvey.example.edu`，完成互動後會向指定的父頁面發送 `postMessage`：

```js
{
  type: 'CHATBOT_APOLOGY_COMPLETE',
  version: 1,
  completed: true,
  condition: 'guilt', // 或 'neutral'
  rid: 'R001',
  duration_seconds: 45
}
```

**這只是完成事件。**必須由 LimeSurvey 頁面的自訂 JavaScript 接收事件、驗證 `event.origin` 為網站主機網域、紀錄事件並解鎖下一步，才算真正串接完成。一般 LimeSurvey 設定不會自動收取這個事件。研究者也不可直接將前端事件當作防作弊的強驗證。

## 5. 研究品質注意事項

- 本網站只改變內疚句子，其他條件保持一致；**請不要對 B 組另外加入表情符號、不同語氣或額外補償**。
- 網站會在同一個瀏覽器分頁內保留互動進度，重新整理不會改變已選組別；受試者若使用不同裝置／無痕模式仍可能重新參與。
- 正式樣本的**唯一身分驗證、避免重複填答、真實完成紀錄、問卷答題與分組紀錄**，都需要由 LimeSurvey 或學校伺服器處理；單一 HTML 無法獨立保證。
- 若未指定 `condition`，網站會自行隨機分配；約 50:50 **不保證兩組樣本量完全相等**。正式施測以問卷系統的隨機分派紀錄為準。
- 請先測試桌面和手機版、重新整理、中途關閉後返回、兩種道歉的文字、以及問卷返回流程，再開始收樣。
- 網頁本身不蒐集姓名、學號或支付資訊。若 `return` 由研究者自行設置，請確認網址不會外洩可識別資訊。

## 6. 你可以修改的地方

在 `index.html` 搜尋：

- `const APOLOGY`：兩組道歉刺激內容。
- `const TEXT`：**兩組共用**對話。修改時不可以只改一組。
- `const delayMs = 1100`：兩組共用的處理等待時間（毫秒）。

更換文案後，請再次確認只有操弄變數產生差異。

## 7. 重要限制

此版是**可以直接執行的互動情境原型**，不是附資料庫、伺服器稽核、實驗後台或已配置 LimeSurvey 問卷的完整受試系統。若要達到「受試者不完成對話，就無法進入正式問卷」的強制條件，仍須在實際 LimeSurvey 頁面配置驗證，並完成跨網站測試。
