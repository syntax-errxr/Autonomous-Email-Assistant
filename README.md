# 📧 Autonomous AI Email Assistant

An intelligent, autonomous email assistant built with **LangGraph** and **local LLMs (Ollama)**. This system acts as a stateful agent that can triage incoming emails, draft contextual responses, schedule meetings, and continuously learn from human feedback using a persistent memory system.

Built with data privacy as a core tenet, this project relies entirely on local language models via Ollama, ensuring sensitive email data never leaves your infrastructure.

---

## ✨ Key Features

- **Intelligent Triage & Routing**: Automatically classifies incoming emails into actionable categories (`Respond`, `Ignore`, or `Notify`), effectively filtering out marketing spam and company-wide blasts before they consume computational resources.
- **Stateful Agent Workflows**: Built on LangGraph, the assistant uses a ReAct (Reason + Act) loop to dynamically select and execute tools (email drafting, calendar scheduling) based on strict Pydantic schemas.
- **Human-in-the-Loop (HITL)**: Execution pauses before any irreversible actions (like dispatching an email). The system yields control back to the user for approval, modification, or rejection.
- **Long-Term Memory Persistence**: The assistant actively learns. When a user modifies an LLM-generated draft, the system extracts the underlying preference and updates its persistent memory store (`BaseStore`), applying these rules to all future interactions.
- **Local AI Privacy**: Powered entirely by local models (e.g., Llama 3.1) running on Ollama, ensuring zero data leakage to external APIs and eliminating inference costs.
- **Gmail API Integration**: Securely integrates with real Gmail accounts via OAuth 2.0 for fetching live threads and dispatching emails.

---

## 🏗️ Architecture Flow

The core architecture leverages **LangGraph** to construct a cyclical, state-driven workflow:

1. **`triage_router` Node**: Evaluates the incoming email payload. Routes to `END` (if Ignore), `triage_interrupt_handler` (if Notify), or directly to the `response_agent` (if Respond).
2. **`triage_interrupt_handler` Node**: Pauses the graph to notify the user of a critical email. The user can inject instructions to force a drafted response.
3. **`response_agent` Sub-Graph**: The engine of the assistant. It evaluates the current state and decides which tools to invoke (`send_email_tool`, `schedule_meeting_tool`, etc.).
4. **`interrupt_handler` Node**: Intercepts sensitive tool calls. The graph state is serialized and paused. Once the human user reviews the action, the graph is resumed, logging feedback into the long-term memory store.
5. **`mark_as_read_node` Node**: Connects to the Gmail API to finalize the email state (marking it as read/processed) once the workflow concludes successfully.

---

## 🛠️ Technology Stack

- **Backend & Logic**: Python 3.11+
- **Orchestration**: LangChain, LangGraph
- **Local Inference**: Ollama (Llama 3.1)
- **External APIs**: Google Cloud Client Library (Gmail API, OAuth 2.0)
- **State & Memory**: LangGraph Checkpointing & BaseStore

---

## 🚀 Setup & Installation

### 1. Prerequisites
- [Python 3.11+](https://www.python.org/)
- [uv](https://github.com/astral-sh/uv) (recommended for dependency management) or `pip`
- [Ollama](https://ollama.com/) installed locally.

Pull your preferred model (default is Llama 3.1):
```bash
ollama run llama3.1
```

### 2. Install Dependencies
Clone the repository and install the required packages:

```bash
# Using uv (Recommended)
uv sync

# Alternatively, using standard pip
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -e .
```

### 3. Environment Configuration
Copy the template environment file:
```bash
cp .env.example .env
```
Update the `.env` file with your specific configurations (e.g., LangSmith tracing variables). *Note: OpenAI API keys are not required as this project relies entirely on local models.*

### 4. Gmail API Setup (Optional)
To enable real email processing:
1. Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
2. Enable the **Gmail API**.
3. Create OAuth 2.0 Client credentials.
4. Download the resulting JSON file, rename it to `credentials.json`, and place it in the project's root directory.

---

## 🚦 Usage

To interact with the various stages of the agent's development, you can run the provided scripts in the `src/email_assistant/` directory.

- `email_assistant.py`: Core routing and basic tool usage.
- `email_assistant_hitl.py`: Adds Human-in-the-Loop execution pausing.
- `email_assistant_hitl_memory.py`: Introduces persistent memory via LangGraph `BaseStore`.
- `email_assistant_hitl_memory_gmail.py`: Full implementation including active Gmail API integration.

---

## 📄 License

This project is open-source and available under the standard MIT License.
