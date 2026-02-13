# Server Logs Kaise Dekhein

## 📍 Logs Kahan Dikhenge?

Server logs **terminal/console** mein dikhenge jahan aap server start karte hain.

## 🚀 Server Start Karne Ka Tareeqa

### Option 1: Terminal Se (Recommended)
```bash
cd server
npm run dev
```

### Option 2: Production Mode
```bash
cd server
npm start
```

## 📊 Logs Mein Kya Dikhega?

Jab aap chatbot use karenge, ye logs dikhenge:

```
🤖 Attempting Gemini API call...
🔧 Initializing Gemini model...
✅ Model initialized successfully
📝 History: 2 messages
📋 System instruction length: 1234 chars
💬 Starting chat session...
📤 Sending message to Gemini API...
📥 Received response from Gemini API
✅ Gemini response extracted (456 chars)
✅ Gemini API call successful, response length: 456
```

Agar error aaye:
```
❌ Gemini AI Error: [error message]
❌ Full error: [details]
⚠️ Falling back to keyword-based responses...
```

## 🔍 Important Logs

1. **Gemini API Status**: `🤖 Attempting Gemini API call...`
2. **Success**: `✅ Gemini API call successful`
3. **Error**: `❌ Gemini AI Error: ...`
4. **Quota Error**: `429 Too Many Requests` ya `quota exceeded`

## 💡 Tips

- Server terminal ko open rakhein taake real-time logs dikhein
- Agar server background mein run ho raha hai, to logs nahi dikhenge
- Cursor ke terminal panel mein bhi logs dikh sakte hain

