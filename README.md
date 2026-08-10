# 📸 LINE Bot 原生相機、相簿與剪貼簿觸發 (Camera, Camera Roll & Clipboard Actions) 🔮

本專案專注於在 LINE Bot 中實現 **LINE 原生相機拍攝 (Camera Action)**、**原生相簿選取 (Camera Roll Action)** 與 **剪貼簿複製 (Clipboard Action)** 功能，提供使用者一鍵喚起手機相機或相簿、拍攝/上傳照片進行多模態 AI 運算與鑑定，並能快速複製提問範本或本次解析內容。

---

## 🌟 核心功能說明：相機、相簿與剪貼簿觸發

### 1. 📸 喚起原生相機拍攝 (`type: "camera"`)
* **一鍵喚起相機**：當使用者在 Quick Reply 快速回覆按鈕點擊「📸 拍照鑑定水晶」時，LINE App 會自動喚起手機的原生相機畫面。
* **零摩擦拍攝體驗**：使用者拍攝後點擊完成，照片便會直接發送至聊天室，觸發後端進行多模態 AI 圖片分析與鑑定。

### 2. 🖼️ 喚起原生相簿選擇 (`type: "cameraRoll"`)
* **直接選擇相簿照片**：當使用者點擊「🖼️ 相簿選擇水晶」按鈕時，LINE App 會自動開啟相簿選擇器，讓使用者自由挑選手機中已有的水晶或物品照片發送。

### 3. 📋 複製提問範本或解析內容 (`type: "clipboard"`)
* **一鍵複製文字**：當使用者點擊 Quick Reply 的 Clipboard 按鈕時，LINE App 會直接把指定文字複製到裝置剪貼簿。
* **實際用途**：本專案用它來提供「複製生日範本」、「複製照片提問」與「複製本次解析」等操作，讓使用者能快速保存或二次編輯內容。

---

## 💻 程式碼實現範例 (Quick Reply Integration)

在 LINE Messaging API 中，透過 Quick Reply 綁定 `camera`、`cameraRoll` 與 `clipboard` action 的語法如下：

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

const clipboardButton = {
  type: 'action',
  action: {
    type: 'clipboard',
    label: '📋 複製提問範本',
    clipboardText: '老師，我的生日是1995年10月12日，請幫我分析我的星盤、生命靈數，以及適合我的水晶。'
  }
};

// 於回覆訊息中夾帶 Quick Reply
const replyMessage = {
  type: 'text',
  text: '請選擇您要發送照片的方式：',
  quickReply: {
    items: [cameraButton, cameraRollButton, clipboardButton]
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

> `clipboard` action 需使用支援的 LINE iOS / Android 版本才能正常運作。
