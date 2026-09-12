# ðŸ§ª Autonomous Email Assistant: Testing Guide & Roadmap

This guide provides a structured roadmap to test every major feature of your AI Email Assistant using the LangSmith Studio. 

## ðŸ“ Step 0: Setup in LangSmith Studio
1. Open the LangSmith Studio UI (`http://127.0.0.1:2024` or the link provided in your terminal).
2. In the top-left dropdown, select the **`email_assistant_hitl_memory_gmail`** graph. *(This graph uses the full Gmail API tools, but we've configured it to safely "simulate" emails if you don't have credentials yet).*
3. At the bottom-left **Input** panel, ensure you click **"View Raw"** (so the button says "View Rendered") and the dropdown says **JSON**. Enter the test payloads exactly as written below.

---

## ðŸ›‘ Test Case 1: The "Ignore" Logic (Spam/Newsletters)
**Goal:** Verify that the assistant can filter out noise and save computational resources.

**Input Payload:**
```json
{
  "email_input": {
    "from": "Marketing Team <newsletter@techdeals.com>",
    "to": "Rohit <rohit@example.com>",
    "subject": "ðŸ”¥ Huge Discounts on Mechanical Keyboards!",
    "body": "Hey Rohit, don't miss out on our summer sale. Up to 50% off all keyboards. Click here to buy now!",
    "id": "spam_id_123"
  }
}
```

* **What to expect:** The graph will execute very quickly. It will start at `__start__`, go to `triage_router`, and immediately proceed to `__end__`.
* **Under the Hood Logic:** The `triage_router` node feeds the email to Ollama along with your system prompt. Ollama uses structured outputs (Pydantic `RouterSchema`) to return exactly `"classification": "ignore"`. The conditional edge sees "ignore" and safely terminates the workflow.

---

## ðŸ”” Test Case 2: The "Notify" Logic (Company Announcements)
**Goal:** Verify that the assistant flags important emails but doesn't automatically reply unless you tell it to.

**Input Payload:**
```json
{
  "email_input": {
    "from": "HR Department <hr@company.com>",
    "to": "All Employees <all@company.com>",
    "subject": "Action Required: Update your emergency contacts",
    "body": "Please log into the portal by Friday to update your emergency contact information.",
    "id": "hr_id_456"
  }
}
```

* **What to expect:** The graph routes to `triage_interrupt_handler` and **PAUSES**. In the Studio, it waits for your input (an Interrupt).

**How to Resume:**
Just like Test Case 3, you won't see actual clickable buttons. You must provide your decision via JSON in the bottom-right resume box (ensure the dropdown says **JSON**).

If you want to just **Ignore** the notification and end the graph:
```json
[
  {
    "type": "ignore"
  }
]
```

If you want to **Respond** to the notification anyway:
```json
[
  {
    "type": "response",
    "args": "Thanks for the heads up, I will update my contacts."
  }
]
```

---

## ✍️ Test Case 3: The "Respond" Logic & Human-in-the-Loop
**Goal:** Verify that the agent can autonomously draft an email, but is blocked from actually sending it without your permission.

**Input Payload:**
```json
{
  "email_input": {
    "from": "Sarah Manager <sarah@company.com>",
    "to": "Rohit <rohit@example.com>",
    "subject": "URGENT: Update on Backend Migration required",
    "body": "Hi Rohit, I need you to reply to this email right now with a quick status update on the LangGraph migration. Are we on track for Friday?",
    "id": "manager_id_789"
  }
}
```

* **What to expect:** 
  1. The `triage_router` classifies it as `"respond"`. *(Note: If the local Llama model decides to be overly cautious and classifies it as "notify" instead, it will pause at `triage_interrupt_handler`. If that happens, just use the "response" JSON from Test Case 2 to force it to draft a reply!)*
  2. The graph moves to `response_agent`, which invokes the LLM.
  3. The LLM decides to use the `write_email` tool. 
  4. The graph routes to `interrupt_handler` and **PAUSES**.
  5. In the Studio UI, you will see the exact email draft the AI created in the right-hand panel.

**How to Approve/Resume:**
Because this project expects a specific array format for interrupts, you must resume it using this exact JSON payload in the bottom-right resume box (ensure the dropdown says **JSON**, not RAW):
```json
[
  {
    "type": "accept"
  }
]
```
Click **Resume** and the agent will simulate sending the email!

---

## ðŸ§  Test Case 4: Long-Term Memory (Editing the Draft)
**Goal:** Verify that the assistant learns your personal preferences forever.

**Continuation of Test Case 3:**
1. While the graph is paused from Test Case 3, look at the AI's email draft in the Studio UI.
2. Select the **"Edit"** or **"Response"** action.
3. Change the AI's draft to something specific, for example, add a signature: *"Yes, we are on track. \n\nBest,\nRohit (Sent via Autonomous Agent)"*
4. Submit the edit to resume the graph.

* **What to expect:** The graph resumes, executes the tool with *your* edited text, and finishes.
* **Under the Hood Logic (The Magic):** When you submit an edit, the code detects that human intervention occurred. It takes the *original* AI draft and *your* edited draft, feeds both to a background LLM call, and asks: *"What is the user's preference here?"* 
The LLM realizes you like to sign off with *"Best, Rohit (Sent via Autonomous Agent)"* and saves this to LangGraph's persistent `BaseStore`.

**To verify the memory worked:**
Run **Test Case 3** again with a slightly different email. You will notice the AI now automatically includes your custom signature in its new drafts!
