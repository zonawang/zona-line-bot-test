# LINE Bot 實戰：結合原生相機（Camera Action）與相簿選擇（Camera Roll Action），打造極致順暢的多模態 AI 拍照鑑定體驗

大家哈囉！如果你正在開發帶有 **多模態圖片辨識 (Multimodal AI)** 功能的 LINE Bot（例如照片辨識、商品拍照鑑定、發票掃描或水晶分析），可能都遇過一個讓人頭痛的問題：**當需要引導使用者傳送照片時，使用者往往找不到聊天室角落的相機按鈕，或是不知道可以即時拍照。**

傳統的互動方式下，使用者必須手動點擊 LINE 聊天室左下角微小的相機或相簿圖示，再切換畫面拍照或選圖。這個看似簡單的步驟，對很多不熟悉操作的使用者來說卻是極大的門檻，常常導致多模態 AI 的互動率低落、對話流暢度中斷。

自從與我的 AI 協作夥伴 **Google Antigravity** 展開合作以來，我們的智慧占星水晶 Bot 已經順利完成了多個階段的進化。這一次，為了解決引導使用者即時拍照與上傳照片的體驗痛點，我們決定全面整合 LINE 官方支援的 **`camera` (原生相機動作)** 與 **`cameraRoll` (原生相簿動作)** 快速回覆按鈕（Quick Replies）。

這篇文章會以實戰且務實的角度，記錄我們在開發過程中是如何設計流暢的拍照鑑定流程、解決跨平台與等不到 Quick Reply 的實戰踩坑經驗，並結合 Gemini 多模態 AI 打造零摩擦的對話體驗。

---

## 📸 理想的互動流程設計

為了解決使用者找不到相機按鈕的混亂情況，並提供高級感的對話體驗，我們設計了以下三部曲：

1. **觸發原生相機與相簿按鈕**：當使用者諮詢水晶鑑定、閱讀「使用指南」，或是主動輸入「可以拍照鑒定水晶嗎」時，機器人會送出說明訊息，並在底部 Quick Reply 彈出 **「📸 點我開啟相機」** 與 **「🖼️ 點我開啟相簿」** 按鈕。
2. **一鍵喚起 LINE 原生功能**：點擊「📸 點我開啟相機」後，LINE App 會自動喚起手機的原生相機全螢幕畫面；點擊「🖼️ 點我開啟相簿」則會直接開啟手機相簿選取器，完全不需尋找聊天室邊角選單。
3. **多模態 Gemini AI 即時鑑定**：使用者完成拍攝或選圖送出後，後端 Webhook 接收到 `image` 訊息，即時將圖片下載並送交 Gemini 多模態 AI 專家，自動分析晶體能量、五行屬性與脈輪共振。

---

## 🧩 實戰踩坑與解決方案

在實際開發與測試過程中，我們遇到了幾個非常真實的體驗與技術踩坑，並與 **Google Antigravity** 一起歸納出了最佳解決方案：

### 坑點一：在手機或 LINE 電腦版上「等不到 Quick Reply 出現」？
* **問題根源**：
  1. LINE 電腦版（Mac / Windows）對 `camera` 與 `cameraRoll` Action **不支援渲染**，若只傳送這兩種按鈕，電腦版 LINE 會直接隱藏 Quick Reply。
  2. 若使用者不知道特定指令，在一般 AI 對話階段可能不容易等出相機 Quick Reply 按鈕。
* **💡 解決方案：雙軌制按鈕（Message Action + Camera Action）**
  我們在 Quick Reply 陣列中同時放入 **`type: "message"` 文字按鈕（`"可以拍照鑒定水晶嗎"`）** 與 **`type: "camera"` 原生相機按鈕**：
  ```javascript
  const cameraButton = {
    type: 'action',
    action: { type: 'camera', label: '📸 點我開啟相機' }
  };
  const cameraRollButton = {
    type: 'action',
    action: { type: 'cameraRoll', label: '🖼️ 點我開啟相簿' }
  };
  const cameraMsgButton = {
    type: 'action',
    action: {
      type: 'message',
      label: '可以拍照鑒定水晶嗎',
      text: '可以拍照鑒定水晶嗎'
    }
  };
  ```
  這樣做能確保不論是在 iOS、Android 還是 LINE 電腦版上，使用者隨時都能看見「可以拍照鑒定水晶嗎」文字按鈕，點擊後立刻觸發相機回應！

---

### 坑點二：使用者主動輸入「可以拍照鑒定水晶嗎」時的即時攔截
* **問題根源**：當使用者手動輸入這句話時，若交由一般 LLM 生成回應，AI 的動態追問未必會帶出 Camera Action。
* **💡 解決方案：關鍵字意圖優先攔截 (Instant Keyword Responder)**
  在 Webhook 邏輯中，優先攔截關鍵字 `/拍照|相機|鑑定水晶|鑒定水晶/`，立刻給予引導並精準附帶 Quick Reply：
  ```javascript
  if (userMessage.includes('拍照') || userMessage.includes('相機') || userMessage.includes('鑑定水晶') || userMessage.includes('鑒定水晶')) {
    isGuide = true;
    responseText = `📸 沒問題！親愛的，請點擊下方 Quick Reply 按鈕【📸 點我開啟相機】，即可立即開啟相機拍攝您的水晶：`;
  }
  ```

---

### 坑點三：程式碼改好了，手機測試卻沒反應？（Cloud Run 容器快取）
* **問題根源**：在本地端修改程式碼並 Git Push 後，如果沒有將最新容器重新部署，線上的 Cloud Run 仍會運行著舊版的 Revision。
* **💡 解決方案：更新後執行 Cloud Run 部署**
  透過 `gcloud run deploy line-echo-bot --source . --region asia-east1` 將最新的程式碼編譯並 100% 切換流量，確保線上 Webhook 即時同步最新邏輯。

---

## 🚀 核心 Webhook 事件處理範例

以下是我們最終優化完成的後端 Webhook 核心處理邏輯：

```javascript
// 核心 Webhook 事件處理
async function handleEvent(event) {
  // 1. 處理文字訊息（如：關鍵字攔截與相機 Quick Reply 觸發）
  if (event.type === 'message' && event.message.type === 'text') {
    const userMessage = event.message.text.trim();
    
    // 定義 Quick Reply 按鈕組合
    const cameraButton = { type: 'action', action: { type: 'camera', label: '📸 點我開啟相機' } };
    const cameraRollButton = { type: 'action', action: { type: 'cameraRoll', label: '🖼️ 點我開啟相簿' } };
    const cameraMsgButton = { type: 'action', action: { type: 'message', label: '可以拍照鑒定水晶嗎', text: '可以拍照鑒定水晶嗎' } };

    if (userMessage.includes('拍照') || userMessage.includes('相機') || userMessage.includes('鑑定水晶') || userMessage.includes('鑒定水晶')) {
      return client.replyMessage({
        replyToken: event.replyToken,
        messages: [{
          type: 'text',
          text: '📸 沒問題！親愛的，請點擊下方按鈕即可即時拍攝或選取照片：',
          quickReply: {
            items: [cameraButton, cameraRollButton, cameraMsgButton]
          }
        }]
      });
    }
  }

  // 2. 處理使用者發送的圖片訊息 (包含透過相機/相簿發送的照片)
  if (event.type === 'message' && event.message.type === 'image') {
    const messageId = event.message.id;
    console.log(`📸 收到圖片訊息 (ID: ${messageId})，準備下載並送交 Gemini AI 鑑定...`);

    // 從 LINE 伺服器下載圖片 Buffer
    const stream = await blobClient.getMessageContent(messageId);
    const chunks = [];
    for await (const chunk of stream) {
      chunks.push(chunk);
    }
    const imageBuffer = Buffer.concat(chunks);
    const base64Image = imageBuffer.toString('base64');

    // 呼叫 Gemini 2.5 Flash 多模態 AI 進行鑑定與解析...
  }
}
```

---

## 🏆 實戰總結

透過與 AI 夥伴 **Google Antigravity** 的通力協作，我們成功在 LINE Bot 中實現了極具質感的拍照鑑定流程：
* **零摩擦的操作體驗**：藉由原生 `camera` 與 `cameraRoll` action，使用者點擊 Quick Reply 即可直接拍照或選圖，徹底解決找不到相機圖示的痛點。
* **全平台高相容性**：透過 Message Action + Camera Action 雙軌設計，保證在 iOS、Android 與 LINE 電腦版上皆能完美顯示與觸發。
* **即時關鍵字攔截**：輸入「可以拍照鑒定水晶嗎」秒回專屬按鈕，大幅提升使用者主動上傳照片進行 AI 分析與鑑定的意願。

如果你也正在開發有關「圖片辨識」、「照片分析」或「現場拍照驗證」的 LINE Bot 服務，強烈推薦使用這套「原生 Camera Action + Camera Roll Action + 雙軌 Quick Reply + 多模態 AI」的組合！

---

### 📂 專案開源與完整程式碼

本專案的完整程式碼（包含 Webhook、Gemini 多模態 AI 整合與 Cloud Run 部署設定）已全面開源。歡迎到 GitHub 進行觀摩或給個 Star 支持我們：

👉 **GitHub 儲存庫：[https://github.com/zonawang/zona-line-bot-test.git](https://github.com/zonawang/zona-line-bot-test.git)**

若對開發細節有任何想法或疑問，歡迎直接在 GitHub 中發起 Issue 與我們探討交流！🌟
