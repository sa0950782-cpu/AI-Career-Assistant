# 🧠 AI Freelance Assistant

A clean, **fully local** AI assistant that helps freelancers analyze client
requests and prepare professional proposals — built with **Python, Gradio,
and Ollama (`qwen2.5:3b`)**. No paid APIs, no data leaving your machine.

It takes a raw client message (e.g. from Upwork, email, or a discovery call)
and walks it through a structured pipeline:

```
Client Request
   → Analyze Request
   → Extract Requirements
   → Identify Missing Information
   → Estimate Complexity
   → Generate Client Questions
   → Generate Professional Proposal
```

Then it gives you a chat window to ask follow-up questions, where the LLM
itself decides — via real tool-calling, not hardcoded buttons — which of the
5 tools to invoke.

![complexity badges](https://img.shields.io/badge/complexity-Beginner%20%7C%20Intermediate%20%7C%20Advanced-informational)
![local](https://img.shields.io/badge/runs-100%25%20locally-brightgreen)
![model](https://img.shields.io/badge/model-qwen2.5%3A3b-blue)

---

## ✨ Why this project

Freelancers constantly have to:
- Figure out *what* a client is actually asking for.
- Notice what critical info is missing (budget, deadline, access, scope).
- Judge how complex the job really is, honestly.
- Ask smart clarifying questions before committing.
- Write a proposal that sounds professional and doesn't overpromise.

This assistant automates that workflow end-to-end, using a **small, local
model** (`qwen2.5:3b`) so it's fast, private, and free to run — a good fit for
a freelance portfolio piece that shows practical LLM engineering (structured
outputs, tool calling, prompt design for small models) rather than just a
ChatGPT wrapper.

---

## 🧰 The 5 tools

| # | Tool | What it does |
|---|------|---------------|
| 1 | `analyze_client_request(request)` | Identifies the **project type**, the **main goal**, the **required technologies**, and the **expected deliverables**. |
| 2 | `extract_requirements(request)` | Splits the request into **functional requirements**, **technical requirements**, and explicitly lists **missing information** the client didn't provide. |
| 3 | `estimate_project(request)` | Estimates **complexity** (`Beginner` / `Intermediate` / `Advanced`), the **skills required**, and the **main tasks** — grounded strictly in what was actually described, never invented. |
| 4 | `generate_client_questions(request)` | Produces the **clarifying questions** a freelancer should ask before accepting the job (scope, budget, timeline, access/credentials, data, success criteria). |
| 5 | `generate_proposal(request)` | Writes a full **professional proposal**: project understanding, proposed solution, deliverables, technologies, questions/assumptions, and a closing paragraph. |

Each tool:
- Calls Ollama in **JSON mode** with a short, tailored system prompt.
- Validates the output against a **Pydantic schema**, so the UI never breaks
  even if the small model returns slightly malformed JSON.
- Explicitly refuses to invent facts — if the client didn't say it, the tool
  says `"Not specified"` or lists it under missing information, instead of
  guessing.

The **"Analyze Request"** button runs all 5 tools as one multi-tool pipeline.
The **chatbot** below it uses Ollama's native tool-calling: the model decides
on its own which tool(s) a follow-up question needs (or none, for a plain
chat reply), calls it with the right arguments, and then phrases the final
answer in natural language.

---

## 📁 Project structure

```
ai-freelance-assistant/
├── app.py                     # Gradio UI and event wiring
├── llm.py                     # Orchestration: full pipeline + tool-calling chat
├── pyproject.toml             # Dependencies
├── README.md
├── tools/
│   ├── schemas.py             # Pydantic models for each tool's output
│   ├── analyze.py             # Tool 1
│   ├── requirements_tool.py   # Tool 2
│   ├── estimate.py            # Tool 3
│   ├── questions.py           # Tool 4
│   ├── proposal.py            # Tool 5
│   └── definitions.py         # Tool registry (specs + function lookup) for tool-calling
├── services/
│   └── ollama_client.py       # Thin wrapper around the local Ollama API
├── prompts/
│   └── prompts.py             # Short, structured prompts tuned for qwen2.5:3b
└── data/
    └── demo_requests.py       # Example client requests for the UI
```

---

## ⚙️ Setup instructions

### 1. Install Ollama
Download and install Ollama from [ollama.com](https://ollama.com) for your OS,
then make sure the Ollama server is running:

```bash
ollama serve
```
(On macOS/Windows the desktop app usually starts this for you.)

### 2. Pull the model

```bash
ollama pull qwen2.5:3b
```

### 3. Set up the Python project

```bash
git clone <your-repo-url> ai-freelance-assistant
cd ai-freelance-assistant

python3 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -e .                 # installs gradio, ollama, pydantic
```

Or, without `pyproject.toml`:

```bash
pip install gradio ollama pydantic
```

### 4. Run the app

```bash
python app.py
```

Gradio will print a local URL (default `http://127.0.0.1:7860`) — open it in
your browser.

---

## 📦 Dependencies

- [`gradio`](https://www.gradio.app/) — web UI
- [`ollama`](https://github.com/ollama/ollama-python) — Python client for the local Ollama server
- [`pydantic`](https://docs.pydantic.dev/) — structured/validated tool outputs

All defined in `pyproject.toml`. Python 3.9+ required.

---

## 🧪 Two demo client requests

**1. n8n automation**
> "We get leads from a Facebook Ads form and a website contact form. Right
> now someone manually copies them into a Google Sheet and then emails our
> sales team. We want this automated with n8n: whenever a new lead comes in
> from either source, it should be added to our CRM (HubSpot) and a Slack
> notification should be sent to the sales channel. We'd also like a simple
> email to be sent to the lead confirming we received their request."

**2. API integration / business workflow automation**
> "Our finance team receives vendor invoices by email as PDF attachments.
> Someone has to manually check the amount against the purchase order, get
> manager approval over email, and then enter the invoice into our
> accounting software (QuickBooks). We'd like to automate this whole
> approval workflow, ideally with some kind of dashboard to track invoice
> status."

Three more (AI automation, Python scripting, API sync) are built into the
app's example dropdown — see `data/demo_requests.py`.

---

## 🖥️ Using the app

1. Paste a client request into the **Client Request** box (or pick a demo
   example from the dropdown).
2. Click **🔍 Analyze Request**. The pipeline runs all 5 tools and fills in:
   - **Analysis** — project type, goal, technologies, deliverables
   - **Requirements** — functional & technical requirements
   - **Complexity** — Beginner/Intermediate/Advanced with reasoning, skills, tasks
   - **Missing Info** — what the client didn't tell you
   - **Client Questions** — what to ask before accepting the job
   - **Generated Proposal** — a full, ready-to-send draft
3. Use the **chat box** at the bottom to ask follow-ups like *"What's the
   complexity and why?"* or *"Make the proposal shorter."* — the model
   automatically calls the right tool.

---

## 🛡️ Design principles

- **No invented facts.** Every prompt explicitly instructs the model not to
  assume client requirements, budget, or freelancer experience that wasn't
  stated. Gaps are labeled as missing information, not filled in.
- **Structured outputs everywhere.** Since `qwen2.5:3b` is a small model,
  every tool call uses Ollama's `format="json"` mode plus Pydantic validation
  with safe fallbacks, so the UI degrades gracefully instead of crashing.
- **Low temperature (0.1)** for consistent, less "creative" outputs — this is
  a business tool, not a chatbot for brainstorming.
- **Real tool-calling**, not just UI buttons: the follow-up chatbot uses
  Ollama's native function-calling API to select and invoke tools.
- **Safe on bad input.** Empty or whitespace-only requests are caught before
  ever reaching the model, with clear messages in the UI.
- **Lightweight.** Three dependencies, no vector DB, no cloud services — easy
  to read, easy to run, easy to extend.

---

## 🧩 Extending it

Some natural next steps if you want to build on this:
- Add a 6th tool, e.g. `estimate_price_range(request, hourly_rate)`.
- Persist analyses to disk (`data/`) to build a history of past proposals.
- Swap `qwen2.5:3b` for a larger local model (e.g. `qwen2.5:7b`) by changing
  `MODEL_NAME` in `services/ollama_client.py` — everything else stays the same.
- Export the generated proposal to a `.docx` or `.pdf` file.

---

## 📄 License

MIT — free to use, modify, and include in your freelance portfolio.
# AI-Career-Assistant
