# LangGraph Basic Chatbot

A simple AI chatbot built with LangGraph and Groq. You pick a model from the sidebar, enter your Groq API key, and chat. That's it.

---

## What this project actually does

You open a Streamlit web page in your browser. On the left sidebar, you choose which AI model you want to use (like Llama3 or Gemma2). You type a message in the chat box. The message goes through a LangGraph "graph" which sends it to the Groq API, gets a reply, and shows it on the screen.

Nothing gets saved between sessions — every time you refresh, it starts fresh.

---

## Tech stack

| Tool | What it does here |
|------|--------------------|
| **Streamlit** | Creates the chat UI in the browser |
| **LangGraph** | Manages the flow of messages (state → node → response) |
| **LangChain** | Connects LangGraph to the Groq API |
| **Groq** | The actual AI that generates responses (fast, free tier available) |
| **Python** | Everything is written in Python |

---

## Folder structure explained

```
BAsicChatbot/
├── app.py                              → Run this to start the app
├── requirements.txt                    → All Python packages needed
├── src/
    └── langgraphagenticai/
        ├── main.py                         → The "brain" — wires UI + LLM + graph together
        ├── LLMS/
        │   └── groqllm.py                  → Connects to Groq using your API key
        ├── state/
        │   └── state.py                    → Defines what "memory" the graph carries (message list)
        ├── graph/
        │   └── graph_builder.py    → Builds the LangGraph flow (START → chatbot → END)
        ├── nodes/
        │   └── basic_chatbot_node.py  → The actual node that calls the LLM
        ├── tools/                          → Empty for now, placeholder for future tools
        └── ui/
            ├── uiconfigfile.ini    → Config file — models, use cases, page title
            ├── uiconfigfile.py     → Reads the .ini config
            └── streamlitui/
                ├── loadui.py       → Renders the sidebar (API key input, model picker)
                └── display_result.py  → Shows the AI response in the chat window
```

---

## How the code flows (step by step)

```
1. app.py runs
       ↓
2. main.py loads
       ↓
3. LoadStreamlitUI → shows sidebar (pick LLM, model, use case) + chat input box
       ↓
4. User types a message and hits Enter
       ↓
5. GroqLLM → takes the API key + model name → creates a ChatGroq object
       ↓
6. GraphBuilder → builds the graph:
       START → chatbot node → END
       ↓
7. BasicChatbotNode.process() → calls llm.invoke(messages) → gets AI reply
       ↓
8. DisplayResultStreamlit → streams the reply back to the chat window
```

---

## The LangGraph part (what makes this different from a basic API call)

Normal approach: `response = llm.invoke(message)` — just call the model directly.

LangGraph approach: you define a **graph** with nodes and edges. Each node does one job. The graph manages what data (called **state**) flows between nodes.

Right now there's only one node (`chatbot`), so it's simple. But this structure makes it easy to add more nodes later — like a search tool, a memory node, or a decision node that routes to different paths.

**State** in this project is just a list of messages:
```python
class State(TypedDict):
    messages: Annotated[List, add_messages]
```
`add_messages` is a LangGraph helper that automatically appends new messages instead of overwriting the list.

---

## Setup and running locally

**Step 1 — Clone or download the project**

**Step 2 — Install dependencies**
```bash
pip install -r requirements.txt
```

**Step 3 — Get a Groq API key**

Go to [console.groq.com/keys](https://console.groq.com/keys) → sign up free → create a key → copy it.

**Step 4 — Run the app**
```bash
streamlit run app.py
```

**Step 5 — Use it**
- Open the URL shown in terminal (usually `http://localhost:8501`)
- Paste your Groq API key in the sidebar
- Pick a model
- Start chatting

---

## Available models (configured in uiconfigfile.ini)

| Model | Notes |
|-------|-------|
| `llama3-8b-8192` | Faster, lighter |
| `llama3-70b-8192` | Smarter, slightly slower |
| `gemma2-9b-it` | Google's model, good at instructions |

To add more models, just edit the `GROQ_MODEL_OPTIONS` line in `uiconfigfile.ini`.

---

## Adding a new use case (how to extend this project)

Right now only `"Basic Chatbot"` is supported. To add more:

1. Add the name to `USECASE_OPTIONS` in `uiconfigfile.ini`
2. Add a new build method in `graph_builder.py` (like `basic_chatbot_build_graph` but for your new use case)
3. Add a condition in `setup_graph()` to call that method
4. Add a display condition in `display_result.py` for how to show the output

The `tools/` folder is already there waiting — that's where you'd add things like web search, calculators, or DB lookups for agentic use cases.

---

## Known limitations

- No chat history between sessions — refresh = fresh start
- Only Groq is supported as LLM provider (OpenAI/others not wired up yet, even though `langchain_openai` is in requirements)
- No streaming animation — response appears all at once after the graph finishes

---

## Requirements (from requirements.txt)

```
langchain
langgraph
langchain_community
langchain_core
langchain_groq
langchain_openai     → installed but not used yet
faiss-cpu            → for vector search (future use)
streamlit
tavily-python        → for web search tool (future use)
```

---

## 👨‍💻 About the Developer

Built by **Harsh M Singh** — B.Tech CSE (Data Science), Lokmanya Tilak College of Engineering, Mumbai.

- 🔗 GitHub: [github.com/singh-105](https://github.com/singh-105)
- 💼 AI Intern @ Deep Cytes

Feel free to connect, star the repo, or open an issue!
