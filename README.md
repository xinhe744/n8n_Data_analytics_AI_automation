# 🤖 AI Data Analysis Agent — n8n Workflow

> An n8n automation that connects a conversational AI agent to a live Google Sheets dataset. Users interact via a public chat interface; the agent reads the sheet on demand, reasons over the data using GPT-4o, and replies with structured, factually grounded answers — while remembering context across the conversation.

---

## 📐 Architecture Overview

<!-- Add your workflow diag<img width="518" height="311" alt="N8n-Data Analysis" src="https://github.com/user-attachments/assets/69350db9-6fe2-474b-ab6c-1da375c8d25d" />
ram image here -->

---

## 🗂️ Repository Structure

```
n8n-ai-data-analyst/
├── README.md               # Project documentation (this file)
├── My_workflow.json        # Importable n8n workflow
└── architecture.md         # Full architecture breakdown
```

---

## 🔄 How a Query Flows

1. A user opens the public chat link and sends a question.
2. The **Chat Trigger** passes the message to the AI Agent as `$json.chatInput`.
3. The Agent reads the injected **system prompt** and decides whether to call the Google Sheets tool.
4. If it calls the tool, the sheet data is returned to the model as context.
5. GPT-4o reasons over the data and generates a response bound by the accuracy rules in the prompt.
6. The reply is sent back to the chat. **Window Buffer Memory** stores the exchange for the next turn.

---

## 🧠 System Prompt — What Keeps the Agent Honest

The agent prompt is deliberately strict. Key rules baked in:

- ✅ Answer only from data that exists in the sheet
- ✅ If the data doesn't support an answer, say so explicitly: *"I cannot find this information in the provided data."*
- ✅ General knowledge is allowed **only** to explain concepts — never to fill data gaps
- ✅ Factual accuracy over speculation, always

> This prevents the model from hallucinating numbers or trends that aren't actually in the dataset.

---

## ⚙️ Configuration

### Prerequisites

- n8n instance (self-hosted or cloud)
- OpenAI API key with GPT-4o access
- Google Sheets OAuth2 credentials connected in n8n
- A Google Sheet with a named tab

### Node Settings to Update Before Running

| Node | Setting | What to Change |
|---|---|---|
| `OpenAI Chat Model` | Model | Swap `gpt-4o` → `gpt-4o-mini` to reduce API costs |
| `Google Sheets1` | `documentId` | Replace with your own spreadsheet ID |
| `Google Sheets1` | `sheetName` | Point to the correct tab in your sheet |
| `When Chat Message Received` | `initialMessages` | Customise the bot name and greeting |
| `AI Agent` | `text` (system prompt) | Tailor for your dataset's domain and column names |

---

## 💡 Design Decisions & Trade-offs

**Why Google Sheets as the data source?**
Non-technical collaborators can update the dataset without touching the workflow. The agent always reads the latest version — no ETL step needed.

**Why Window Buffer Memory and not full history?**
Full conversation history inflates token usage quickly, especially with large sheets loaded into context. The window keeps recent turns (typically last 5–10 messages) — enough for follow-up questions without ballooning costs.

**Why GPT-4o?**
The workflow uses `gpt-4o` by default. For cost-sensitive deployments, `gpt-4o-mini` handles structured data Q&A well and is significantly cheaper — worth testing before going to production.

**What this workflow does NOT do:**
- It does not write back to the sheet
- It does not generate charts or visualisations
- It does not handle multi-sheet joins natively (requires prompt engineering or additional tool nodes)


