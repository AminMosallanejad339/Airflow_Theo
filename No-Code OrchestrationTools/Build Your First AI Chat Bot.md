# Build Your First AI Agent — n8n Workflow Tutorial

---

## Overview

This workflow creates your very first AI Agent — a conversational chatbot powered by Google Gemini language model that can not only chat but also perform actions using connected tools.

Think of this AI Agent as a smart assistant that understands your queries, fetches real-world data, and automates tasks by invoking various integrated services. This is a great starting point to explore AI-powered automation with n8n.

---

## What It Does

- **Chat Interface:** Lets you chat with your AI Agent via a clean chat trigger.

- **AI Brain:** The agent processes messages, decides which tool to use, and replies intelligently.

- **Language Model:** Uses Google Gemini to understand and generate natural language.

- **Tools Included:**
  
  - **Get Weather:** Fetches real-time weather forecasts using Open-Meteo API.
  
  - **Get News:** Reads RSS feeds from popular news sources.

- **Memory:** Remembers recent conversation history to maintain context.

- **Customization:** You can modify the system message to tweak the AI’s personality and behavior.

- **Extensible:** Add more tools like Gmail, Google Calendar, etc.

---

## Workflow Nodes Description

| Node Name               | Type                           | Purpose                                                                         |
| ----------------------- | ------------------------------ | ------------------------------------------------------------------------------- |
| **Introduction Note**   | Sticky Note                    | Introduction and usage instructions from the creator, Lucas Peyrin              |
| **Example Chat**        | Chat Trigger                   | Public chat interface node to send and receive messages                         |
| **Your First AI Agent** | AI Agent (Langchain)           | Core AI node that handles messages, selects tools, generates responses          |
| **Connect Gemini**      | Language Model (Google Gemini) | Connects to Google Gemini API for natural language understanding and generation |
| **Conversation Memory** | Memory Buffer                  | Stores last 30 messages to keep conversation context                            |
| **Get Weather**         | HTTP Request Tool              | Fetches weather data from Open-Meteo API                                        |
| **Get News**            | RSS Feed Read Tool             | Reads latest headlines from configured RSS feed URLs                            |
| **Sticky Notes**        | Sticky Notes                   | Various notes providing tips, instructions, and reminders                       |

---

## Setup Instructions

### Step 1: Get Google AI API Key

1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey).

2. Click **“Create API key in new project”**.

3. Copy the generated API key.

### Step 2: Configure Credentials in n8n

1. Open the workflow in n8n.

2. Click on the **Connect Gemini** node.

3. In the credentials dropdown, select **+ Create New Credential**.

4. Paste your copied API key into the **API Key** field.

5. Save the credential.

### Step 3: Activate Workflow

- Activate the workflow to enable the chat interface.

- Optionally, share the public chat URL (enabled in the **Example Chat** node) so others can interact with your AI Agent.

---

## How to Use

- Click the **🗨 Open Chat** button on the **Example Chat** node.

- Start typing messages like:
  
  - "What's the weather in Paris?"
  
  - "Get me the latest tech news."
  
  - "Give me ideas for n8n AI agents."

- The AI Agent will understand your request, pick the appropriate tool, fetch the data, and respond naturally.

---

## How the AI Agent Works Internally

- Receives your chat message from the **Example Chat** node.

- Uses the **Google Gemini** model for language understanding.

- Refers to recent conversation history stored in **Conversation Memory**.

- Decides which tool to invoke:
  
  - **Get Weather** tool for weather-related queries.
  
  - **Get News** tool for news-related queries.

- Calls the appropriate tool node.

- Combines tool results and replies back through the chat.

---

## Customization Tips

- **Edit System Message:**  
  In the **Your First AI Agent** node, modify the **System Message** to change the AI’s personality, instructions, and behavior.

- **Add More Tools:**  
  Click the ➕ button under the AI Agent’s tool inputs to add tools like:
  
  - Google Calendar (Get Upcoming Events)
  
  - Gmail (Send an Email)

- **Adjust Memory Settings:**  
  Change how many recent messages the AI remembers by editing the **Conversation Memory** node’s context window.

---

## Notes and Best Practices

- Keep the AI Agent’s toolset focused (under 10–15 tools) to maintain reliability.

- Agents are ideal for user-facing interactions, while complex backend processes may need structured workflows.

- Regularly update API keys and credentials.

- Monitor usage limits of the Google Gemini API.

- Share your activated workflow URL carefully if public access is enabled.

---

## Additional Resources

- [n8n Documentation](https://docs.n8n.io/)

- [Google Gemini API Documentation](https://developers.google.com/palm)

- [n8n AI Agent Blog](https://n8n.io/blog/ai-agents)

- [Coaching & Support](https://api.ia2s.app/form/templates/coaching?template=Very%20First%20AI%20Agent)


