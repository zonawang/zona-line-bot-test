# LINE Bot 實戰：整合原生相機 (Camera Action) 與相簿選擇 (Camera Roll Action)，打造順暢的照片拍攝與多模態 AI 體驗

大家哈囉！如果你正在開發帶有 **多模態圖片辨識 (Multimodal AI)** 功能的 LINE Bot（例如照片辨識、商品拍照鑑定、發票掃描或水晶分析），你一定希望使用者在發送圖片時的流程越順暢越好。

通常使用者要發送圖片給 LINE Bot 時，必須手動點擊聊天室左下角的相機或相簿小圖示，但許多使用者並不熟悉這些按鈕的位置或操作。

為了提供更直覺、一鍵式的互動體驗，LINE Messaging API 提供了官方原生的 **`camera` (相機動作)** 與 **`cameraRoll` (相簿動作)** Actions。使用者只需要點擊訊息底部的 Quick Reply 按鈕，就可以直接開啟手機內建相機拍攝或選擇相簿相片！

這篇文章會記錄我們是如何在 LINE Bot 中整合原生的 **Camera Action** 與 **Camera Roll Action**，並與 Gemini 多模態 AI 結合，創造零摩擦的拍照鑑定體驗。

---

## 📸 什麼是 Camera Action 與 Camera Roll Action？

在 LINE Messaging API 中，`camera` 與 `cameraRoll` 是兩種特殊的 Action 物件：

1. **Camera Action (`type: "camera"`)**：
   - 點擊後，LINE App 會立刻喚起手機相機全螢幕畫面。
   - 使用者按下拍攝並確認後，照片會自動發送至當前聊天室。
2. **Camera Roll Action (`type: "cameraRoll"`)**：
   - 點擊後，LINE App 會自動彈出手機相簿選擇器，讓使用者直接選擇相片送出。

---

## 💡 Quick Reply 整合範例程式碼

我們可以在 Quick Reply 快速回覆選單中加入這兩個 Action，讓使用者隨時能一鍵拍攝：

```javascript
// 1. 定義 Camera Action 與 Camera Roll Action 按鈕
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

// 2. 於訊息中夾帶 Quick Reply
const replyMessage = {
  type: 'text',
  text: '親愛的，您可以點擊下方按鈕直接開啟相機拍照，或是從相簿選擇您的水晶照片發送給我。',
  quickReply: {
    items: [cameraButton, cameraRollButton]
  }
};

// 3. 發送訊息至 LINE Messaging API
await client.replyMessage({
  replyToken: event.replyToken,
  messages: [replyMessage]
});
```

---

## 🎯 體驗優化小技巧

1. **按鈕標籤文字精簡**：`label` 限制在 20 個字元以內，加上適合的 Emoji（如 📸 或 🖼️）能大幅提升辨識度。
2. **多模態事件接收處理**：當使用者透過 Camera Action 發送照片後，後端 Webhook 會接收到 `event.message.type === 'image'`，即可調用 `getMessageContent` 下載圖片並送交多模態 AI 進行分析。

透過整合 LINE 原生的 Camera Action，使用者不必再辛苦找尋相機圖示，隨手一點就能即時拍照上傳，大大提高了多模態 AI 應用的互動率與使用體驗！
