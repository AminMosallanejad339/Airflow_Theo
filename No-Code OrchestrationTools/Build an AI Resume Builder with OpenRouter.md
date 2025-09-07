# 🎯 Project: Auto-Resume Parser with OpenRouter AI using n8n

In this tutorial, you'll build a workflow that:

> 📩 Monitors your Gmail inbox → 📎 Detects PDF resumes → 📄 Extracts text → 🧠 Sends it to an AI model → 📊 Parses structured data → 📥 Saves to Google Sheets

All powered by **n8n** and **OpenRouter AI**.

---

## 🧠 What This Workflow Does (Overview)

1. Watches for incoming Gmail messages.

2. Filters emails with `.pdf` attachments.

3. Extracts text from the PDF.

4. Sends it to OpenRouter AI using their API and your selected model.

5. Parses the structured AI response (e.g., name, skills, education, experience).

6. Appends the parsed result to a Google Sheet for later review.

---

## 🧩 Node Summary Table

| Node # | Node Type            | Purpose                                       |
| ------ | -------------------- | --------------------------------------------- |
| 1      | **Gmail Trigger**    | Triggers when a new email is received         |
| 2      | **IF**               | Checks if the email contains a PDF attachment |
| 3      | **Move Binary Data** | Converts PDF to binary format for processing  |
| 4      | **PDF Extract**      | Extracts text content from the PDF            |
| 5      | **HTTP Request**     | Sends resume text to OpenRouter AI            |
| 6      | **JSON Parse**       | Parses structured response from the AI        |
| 7      | **Google Sheets**    | Appends parsed data to a Google Sheet         |

---

## 🔧 Prerequisites

- A local running instance of **n8n**

- A Google Account with access to **Gmail** and **Google Sheets**

- An account at [https://openrouter.ai](https://openrouter.ai/) and an **API key**

---

## 🧱 Step-by-Step Workflow Setup

### ✅ Step 1: Gmail Trigger Node

- Add **Gmail Trigger**

- Set authentication with **OAuth2**

- Scope: `https://www.googleapis.com/auth/gmail.readonly`

- Configure to trigger on: `New Email`

- Filter by label or sender if desired (optional)

---

### ✅ Step 2: IF Node – Check for PDF

- Add an **IF Node**

- Check: `binary[0].mimeType contains 'application/pdf'`

This ensures only emails with PDF attachments proceed.

---

### ✅ Step 3: Move Binary Data Node

- Add a **Move Binary Data** node

- Enable: “Include All Binary Data”

- This converts binary for PDF extraction in next step

---

### ✅ Step 4: PDF Extract Node

- Add **PDF Extract Node**

- Input: `binary`

- Output: Raw text of the resume file

Now you have the plain text resume body to send to the AI.

---

### ✅ Step 5: HTTP Request – Send to OpenRouter AI

- Add **HTTP Request** node

**Config:**

- Method: `POST`

- URL: `https://openrouter.ai/api/v1/chat/completions`

- Headers:
  
  - `Authorization: Bearer YOUR_OPENROUTER_API_KEY`
  
  - `Content-Type: application/json`

- Body Type: `RAW` → JSON

- JSON Payload:

```json
{
  "model": "mistralai/mistral-7b-instruct",
  "messages": [
    {
      "role": "system",
      "content": "You are an HR assistant. Extract the following information from resumes: full name, work experience, education, and skills. Reply in JSON format."
    },
    {
      "role": "user",
      "content": "{{ $json['text'] }}"
    }
  ]
}
```

This sends the resume content to the AI for analysis.

---

### ✅ Step 6: Parse JSON Response

- Add a **Set Node** or **Function Node**

- Extract fields like:

```js
{
  "name": $json.choices[0].message.content.name,
  "skills": $json.choices[0].message.content.skills,
  "education": $json.choices[0].message.content.education,
  "experience": $json.choices[0].message.content.experience
}
```

Adjust this structure depending on the actual AI response.

---

### ✅ Step 7: Google Sheets Node

- Connect your **Google Sheets** account

- Choose: “Append”

- Select or create a Google Sheet

- Map the fields:
  
  - `Name` → `name`
  
  - `Skills` → `skills`
  
  - `Education` → `education`
  
  - `Experience` → `experience`

---

## 🔄 Optional Enhancements

- Add a **Slack Node** to notify when a resume is parsed

- Add a **Notion Node** to store candidate cards

- Add error-handling with **Catch/Error Trigger**

---

## 🛡️ Notes on Security and Privacy

- Do not send personal info to models you don’t trust

- You can self-host your own LLM or use paid models like Claude, GPT-4 if needed

---

## ✅ You're Done!

Now every new email with a PDF resume will be processed and neatly structured into a spreadsheet using **n8n + AI**.


