# 📸 LINE Bot 原生相機與相簿觸發 (Camera & Camera Roll Actions) 🔮

本專案專注於在 LINE Bot 中實現 **LINE 原生相機拍攝 (Camera Action)** 與 **原生相簿選取 (Camera Roll Action)** 功能，提供使用者一鍵喚起手機相機或相簿、拍攝/上傳照片進行多模態 AI 運算與鑑定的極致順暢體驗。

---

## 🌟 核心功能說明：原生相機與相簿觸發 (Camera & Camera Roll Actions)

### 1. 📸 喚起原生相機拍攝 (`type: "camera"`)
* **一鍵喚起相機**：當使用者在 Quick Reply 快速回覆按鈕點擊「📸 拍照鑑定水晶」時，LINE App 會自動喚起手機的原生相機畫面。
* **零摩擦拍攝體驗**：使用者拍攝後點擊完成，照片便會直接發送至聊天室，觸發後端進行多模態 AI 圖片分析與鑑定。

### 2. 🖼️ 喚起原生相簿選擇 (`type: "cameraRoll"`)
* **直接選擇相簿照片**：當使用者點擊「🖼️ 相簿選擇水晶」按鈕時，LINE App 會自動開啟相簿選擇器，讓使用者自由挑選手機中已有的水晶或物品照片發送。

---

## 💻 程式碼實現範例 (Quick Reply Integration)

在 LINE Messaging API 中，透過 Quick Reply 綁定 `camera` 與 `cameraRoll` action 的語法如下：

```javascript
// 定義相機與相簿 Quick Reply 按鈕
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

// 於回覆訊息中夾帶 Quick Reply
const replyMessage = {
  type: 'text',
  text: '請選擇您要發送照片的方式：',
  quickReply: {
    items: [cameraButton, cameraRollButton]
  }
};
```

---

## 🛠️ 環境配置與部署

### 1. 本地環境變數設定 (`.env`)
請在專案根目錄下建立 `.env` 檔案：

```env
PORT=8080

# LINE 官方通道憑證
LINE_CHANNEL_SECRET=your_channel_secret_here
LINE_CHANNEL_ACCESS_TOKEN=your_channel_access_token_here

# Google Cloud 與 Gemini AI 設定
GCP_PROJECT=your_gcp_project_id_here
GCP_LOCATION=us-central1
VERTEX_AI_MODEL=gemini-2.5-flash
```

### 2. 本地執行與測試
```bash
npm install
npm start
```

### 3. 部署至 Google Cloud Run
```bash
gcloud run deploy line-echo-bot --source . --region asia-east1
```
