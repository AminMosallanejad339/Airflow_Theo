# 📰 Tutorial: Build an AI-Powered Tech News Digest with n8n + Telegram

This tutorial walks you through building an **automated news summarizer and notifier** that:

- Reads the latest posts from **Product Hunt** and **TechCrunch AI** via RSS

- Summarizes them using **Google Gemini AI via OpenRouter**

- Sends digest summaries to your **Telegram channel**

> 📦 Tools used: RSS Feed, Google Gemini, AI Agent Nodes, Telegram

---

## ⚙️ What the Workflow Does

| Step | Node             | Description                                                                |
| ---- | ---------------- | -------------------------------------------------------------------------- |
| 1    | Schedule Trigger | Runs the workflow periodically (e.g., daily)                               |
| 2    | RSS Read         | Fetches Product Hunt RSS feed                                              |
| 3    | RSS Read1        | Fetches TechCrunch AI RSS feed                                             |
| 4    | Edit Fields      | Prepares Product Hunt article content                                      |
| 5    | Edit Fields1     | Prepares TechCrunch article content                                        |
| 6    | AI Agent         | Summarizes Product Hunt posts using Gemini with a casual, one-liner format |
| 7    | AI Agent1        | Summarizes TechCrunch articles in simple Persian                           |
| 8    | Telegram1        | Sends Product Hunt summary to Telegram                                     |
| 9    | Telegram         | Sends TechCrunch summary to Telegram                                       |

---

## 🧱 Step-by-Step Workflow Setup

### ✅ 1. Schedule Trigger

- Node: **Schedule Trigger**

- Set it to run daily, hourly, or at your preferred interval.

### ✅ 2. Fetch RSS Feeds

- **RSS Read**
  
  - URL: `https://www.producthunt.com/feed`
  
  - Options: Enable `Ignore SSL` if needed

- **RSS Read1**
  
  - URL: `https://techcrunch.com/category/artificial-intelligence/feed`

### ✅ 3. Format RSS Content

- **Edit Fields (for Product Hunt)**
  
  - Set value `content = {{$json.content}}`

- **Edit Fields1 (for TechCrunch)**
  
  - Same setting: `content = {{$json.content}}`

### ✅ 4. Summarize via AI Agent

#### 🧠 AI Agent (Product Hunt)

- Prompt (in Persian):
  
  > "تو یک ایجنت خبره‌ای که اطلاعات محصولات جدید منتشرشده در Product Hunt را تحلیل می‌کنی..."
  
  - Uses Gemini model: `models/gemini-2.5-flash-preview-05-20`
  
  - Format output with emoji headers, product name + link + short sentence + category

#### 🧠 AI Agent1 (TechCrunch)

- Prompt (in Persian):
  
  > "تو نقش یک خبرنگار متخصص فناوری رو داری..."
  
  - Summarizes articles in simple Persian, adds title, explanation, and link

### ✅ 5. Send to Telegram

#### Telegram1 (Product Hunt Summary)

- `chatId`: your Telegram chat/channel ID

- `text`: `={{ $json.output }}` (summary from AI Agent)

- `parse_mode`: `HTML`

#### Telegram (TechCrunch Summary)

- Same setup as above, but gets input from AI Agent1

---

## 🛠 Setup Notes

- **Google Gemini API**: Use OpenRouter with your API Key.

- **Telegram**: Create a bot with [@BotFather](https://t.me/BotFather), get the token and chat ID.

- **LangChain Agent Nodes**: You need the Langchain nodes installed and configured.

- **Gemini Model**: Use `models/gemini-2.5-flash-preview-05-20`

---

## ✅ Final Result

- Once scheduled, the workflow fetches both feeds

- Summarizes with AI using custom prompts

- Delivers digest to your Telegram in a friendly, well-formatted way


