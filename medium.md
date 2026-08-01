# LINE Bot 實戰：結合原生相機（Camera Action）與相簿選擇（Camera Roll Action），打造極致順暢的多模態 AI 拍照鑑定體驗

大家哈囉！如果你正在開發帶有 **多模態圖片辨識 (Multimodal AI)** 功能的 LINE Bot（例如照片辨識、商品拍照鑑定、發票掃描或水晶分析），可能都遇過一個讓人頭痛的問題：**當需要引導使用者傳送照片時，使用者往往找不到聊天室角落的相機按鈕，或是不知道可以即時拍照。**

傳統的互動方式下，使用者必須手動點擊 LINE 聊天室左下角微小的相機或相簿圖示，再切換畫面拍照或選圖。這個看似簡單的步驟，對很多不熟悉操作的使用者來說卻是極大的門檻，常常導致多模態 AI 的互動率低落、對話流暢度中斷。

自從與我的 AI 協作夥伴 **Google Antigravity** 展開合作以來，我們的智慧占星水晶 Bot 已經順利完成了多個階段的進化。這一次，為了解決引導使用者即時拍照與上傳照片的體驗痛點，我們決定全面整合 LINE 官方支援的 **`camera` (原生相機動作)** 與 **`cameraRoll` (原生相簿動作)** 快速回覆按鈕（Quick Replies）。

這篇文章會以實戰且務實的角度，記錄我們在開發過程中是如何設計流暢的拍照鑑定流程，並結合 Gemini 多模態 AI 打造零摩擦的對話體驗。

---

## 📸 理想的互動流程設計

為了解決使用者找不到相機按鈕的混亂情況，並提供高級感的對話體驗，我們設計了以下三部曲：

1. **觸發原生相機與相簿按鈕**：當使用者諮詢水晶鑑定或閱讀「使用指南」時，機器人會送出說明訊息，並在底部 Quick Reply 彈出 **「📸 拍照鑑定水晶」** 與 **「🖼️ 相簿選擇水晶」** 按鈕。
2. **一鍵喚起 LINE 原生功能**：點擊「📸 拍照鑑定水晶」後，LINE App 會自動喚起手機的原生相機全螢幕畫面；點擊「🖼️ 相簿選擇水晶」則會直接開啟手機相簿選取器，完全不需尋找聊天室邊角選單。
3. **多模態 Gemini AI 即時鑑定**：使用者完成拍攝或選圖送出後，後端 Webhook 接收到 `image` 訊息，即時將圖片下載並送交 Gemini 多模態 AI 專家，自動分析晶體能量、五行屬性與脈輪共振。

---

## 🧩 挑戰一：為什麼需要原生 Camera & Camera Roll Actions？

在開發多模態 AI LINE Bot 時，我們發現使用者在對話中最容易卡關的環節就是「傳送圖片」。

### 💡 解決方法：直接將相機與相簿入口「送到使用者眼前」
LINE Messaging API 提供了兩種強大的原生 Action 類型：
* **`type: "camera"`**：綁定後，使用者一點擊按鈕，LINE 就會直接開啟手機相機。拍攝完成按下確認，照片就會自動發送到聊天室。
* **`type: "cameraRoll"`**：綁定後，使用者一點擊按鈕，LINE 就會自動開啟手機相簿選擇器，讓使用者自由選擇已有的照片送出。

將這兩個按鈕放在訊息底部的 Quick Reply 選單中，等於將拍照與選圖的入口「直接送到使用者眼前」，將拍攝門檻降到了最低。

---

## 🧩 挑戰二：Quick Reply 按鈕的結構與相容性設計

在實作 `camera` 與 `cameraRoll` action 時，語法結構非常乾淨且直覺：

```javascript
// 1. 定義原生相機按鈕 (Camera Action)
const cameraButton = {
  type: 'action',
  action: {
    type: 'camera',
    label: '📸 拍照鑑定水晶'
  }
};

// 2. 定義原生相簿按鈕 (Camera Roll Action)
const cameraRollButton = {
  type: 'action',
  action: {
    type: 'cameraRoll',
    label: '🖼️ 相簿選擇水晶'
  }
};
```

### ⚠️ 注意事項：
* **`label` 字數限制**：Quick Reply 按鈕的 `label` 嚴格限制在 **20 個字元以內**（包含 Emoji）。建議加上簡明圖示（如 📸、🖼️）增加視覺辨識度。
* **跨平台支援**：`camera` 與 `cameraRoll` action 在 iOS 與 Android 版的 LINE App 上均有完善支援，能提供一致的原生體驗。

---

## 🚀 核心 Webhook 事件處理範例

以下是我們在後端 Webhook 中整合 Camera Action 與處理接收到的圖片訊息的核心邏輯：

```javascript
// 核心 Webhook 事件處理
async function handleEvent(event) {
  // 1. 處理文字訊息（如：使用指南、認識水晶）
  if (event.type === 'message' && event.message.type === 'text') {
    const userText = event.message.text.trim();
    
    // 定義原生相機與相簿 Quick Reply 按鈕
    const cameraButton = {
      type: 'action',
      action: {
        type: 'camera',
        label: '📸 拍照鑑定水晶'
      }
    };
    const cameraRollButton = {
      type: 'action',
      action: {
        type: 'cameraRoll',
        label: '🖼️ 相簿選擇水晶'
      }
    };

    return client.replyMessage({
      replyToken: event.replyToken,
      messages: [{
        type: 'text',
        text: '🔮 親愛的，您可以點擊下方按鈕直接開啟相機拍攝您的水晶，或是從相簿選擇照片發送給我：',
        quickReply: {
          items: [cameraButton, cameraRollButton]
        }
      }]
    });
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
    // (接續 Gemini 多模態運算邏輯)
  }
}
```

---

## 🏆 實戰總結

透過與 AI 夥伴 **Google Antigravity** 的通力協作，我們成功在 LINE Bot 中實現了極具質感的拍照鑑定流程：
* **零摩擦的操作體驗**：藉由原生 `camera` 與 `cameraRoll` action，使用者點擊 Quick Reply 即可直接拍照或選圖，徹底解決找不到相機圖示的痛點。
* **高互動率的多模態 AI**：將照片發送入口前置，大幅提升使用者主動上傳照片進行 AI 分析與鑑定的意願。
* **流暢的閉環體驗**：從「訊息引導」 $\rightarrow$ 「一鍵開啟相機/相簿」 $\rightarrow$ 「Gemini AI 圖片鑑定」一氣呵成，打造出真正直覺且現代化的聊天機器人應用。

如果你也正在開發有關「圖片辨識」、「照片分析」或「現場拍照驗證」的 LINE Bot 服務，強烈推薦使用這套「原生 Camera Action + Camera Roll Action + 多模態 AI」的組合！

---

### 📂 專案開源與完整程式碼

本專案的完整程式碼（包含 Webhook、Gemini 多模態 AI 整合與 Cloud Run 部署設定）已全面開源。歡迎到 GitHub 進行觀摩或給個 Star 支持我們：

👉 **GitHub 儲存庫：[https://github.com/zonawang/zona-line-bot-test.git](https://github.com/zonawang/zona-line-bot-test.git)**

若對開發細節有任何想法或疑問，歡迎直接在 GitHub 中發起 Issue 與我們探討交流！🌟
