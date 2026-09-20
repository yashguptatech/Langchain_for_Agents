# LangChain for Agent Development: Multimodal AI Shopping Assistant

This repository contains the complete lab materials and interactive application for a corporate training workshop on **LangChain for Agent Development**[cite: 1]. It guides developers and engineers from foundational agent theory (the ReAct loop and memory persistence) to building a multimodal, tool-augmented AI Shopping Assistant[cite: 2, 5].

---

## 📖 Project Overview

Modern AI applications require more than static LLM prompts—they require autonomous agents that can plan, reason, invoke external APIs, query databases, and maintain state across user sessions[cite: 2, 5]. 

This project is organized into two primary learning modules:

1. **Foundational Lab (`product_query_agent_with_memory.ipynb`)**:
   * **Stateless vs. Stateful Execution:** Demonstrates why standard LLMs suffer from "conversational amnesia" when users reference past context (e.g., *"What are the reviews on this product?"*).
   * **LangGraph Checkpointing:** Introduces `InMemorySaver` and session threads (`thread_id`) to maintain conversation history across multiple turns.
   * **Tool Binding:** Teaches how to construct custom tools using the `@tool` decorator and docstring schemas.

2. **Production-Ready Application (`app.py` & `shopping_agent.py`)**[cite: 2, 5]:
   * **Dual-Model Routing:** Employs Groq's high-speed inference for text reasoning (`qwen/qwen3.8-27b`) and NVIDIA's multimodal endpoint (`meta/llama-3.2-11b-vision-instruct`) for visual product inspection.
   * **Multi-Tool Orchestration:** Equips the agent with tools to query a local SQLite database (`store.db`), fetch customer reviews, calculate dynamic aggregates, and commit order transactions[cite: 4, 5, 6].
   * **Streamlit UI with Live Execution Traces:** An interface featuring chat history and a sidebar image uploader.

---

## 🏗️ System Architecture

```text
                    +------------------------------+
                    |    Streamlit UI (app.py)     |
                    +--------------+---------------+
                                   |
                   User Prompt / Uploaded Image
                                   |
                                   v
             +--------------------------------------------+
             |        LangChain Agent Runtime             |
             |           (shopping_agent.py)              |
             |                                            |
             |  Checkpointer: MemorySaver (thread_id)     |
             +---------------------+----------------------+
                                   |
          +------------------------+------------------------+
          |                                                 |
          v                                                 v
+-------------------+                             +-------------------+
|  Groq LLM Engine  |                             | NVIDIA Vision LLM |
| (Qwen 2.5 27B)    |                             | (Llama 3.2 11B)   |
+---------+---------+                             +---------+---------+
          |                                                 |
          +-------------------+-----------------------------+
                              |
                     Invokes Registered Tools
                              |
       +----------------------+----------------------+
       |                      |                      |
       v                      v                      v
+--------------+      +----------------+     +---------------+
| search_      |      | get_rating()   |     | checkout()    |
| products()   |      | (reviews_      |     | (Commits to   |
| (store.db)   |      |  api.py)       |     |  orders table)|
+--------------+      +----------------+     +---------------+
```

---

## 🛠️ Repository Structure

```text
Langchain_for_Agents/
├── app.py                                  # Streamlit front-end chat interface
├── shopping_agent.py                       # Core LangChain agent definition, tools, and prompts
├── reviews_api.py                          # SQL aggregator utility for customer reviews
├── setup.py                                # SQLite database creation and sample data seeding
├── store.db                                # SQLite database file containing products, reviews, and orders
├── product_query_agent_with_memory.ipynb   # Interactive step-by-step training notebook
├── pyproject.toml                          # Project dependencies and environment specification
└── README.md                               # Project documentation

```



## ⚙️ Prerequisites & Environment Setup

This project uses [`uv`](https://docs.astral.sh/uv/?utm_source=gemini), an extremely fast Python package and environment manager.

### 1. Install `uv`

**macOS / Linux:**

```bash
curl -LsSf [https://astral.sh/uv/install.sh](https://astral.sh/uv/install.sh) | sh

```

**Windows (PowerShell):**

```powershell
powershell -ExecutionPolicy ByPass -c "irm [https://astral.sh/uv/install.ps1](https://astral.sh/uv/install.ps1) | iex"

```

Verify the installation:

```bash
uv --version

```

---

### 2. Clone the Repository

```bash
git clone [https://github.com/yashguptatech/Langchain_for_Agents.git](https://github.com/yashguptatech/Langchain_for_Agents.git)
cd Langchain_for_Agents

```

---

### 3. Sync Dependencies

Run `uv sync` to automatically create a virtual environment (`.venv`) and install all pinned dependencies from `pyproject.toml`:

```bash
uv sync

```

---

### 4. Activate the Virtual Environment

Before executing commands or running scripts, activate the virtual environment created by `uv`:

**macOS / Linux:**

```bash
source .venv/bin/activate

```

**Windows (Command Prompt):**

```cmd
.venv\Scripts\activate.bat

```

**Windows (PowerShell):**

```powershell
.venv\Scripts\Activate.ps1

```

*(Your terminal prompt will now display `(.venv)`).*

---

### 5. Configure Environment Variables (`.env`)

Create a `.env` file in the root directory of the project:

```bash
touch .env

```

Add your API keys to the `.env` file:

```env
# Primary LLM for Agent Reasoning
GROQ_API_KEY=your_groq_api_key_here

# Multimodal Vision LLM for Image Queries
NVIDIA_API_KEY=your_nvidia_api_key_here

```

* Obtain a Groq API key: [Groq Console](https://console.groq.com/keys?utm_source=gemini)
* Obtain an NVIDIA AI Foundation API key: [NVIDIA Build](https://build.nvidia.com/?utm_source=gemini)

---

### 6. Initialize the Database

Populate the local SQLite database (`store.db`) with mock catalog products, ratings, and customer reviews:

```bash
python setup.py

```

You should see:

```text
Database created at: .../store.db

```

---

## 💻 Running the Project

### A. Run the Interactive Lab Notebook

For lab sessions, run Jupyter and open the guided notebook:

```bash
jupyter notebook product_query_agent_with_memory.ipynb

```

Follow the numbered cells to walk through:

1. Building custom `@tool` functions with type hints and docstrings.


2. Testing tool calling on a stateless agent.


3. Observing memory failure when using pronouns like *"this product"*.


4. Attaching an `InMemorySaver` checkpointer and configuring session `thread_id` to maintain conversation state.



---

### B. Run the Full AI Shopping Assistant App

Launch the interactive Streamlit user interface:

```bash
streamlit run app.py

```

Once loaded in your browser (`http://localhost:8501`), test the following core scenarios:

1. **Multi-Constraint Search & Rating:**
> *"I want organic honey under $20 with a 4.5+ rating."*
> 


* The agent queries the database with price and organic filters, retrieves candidate reviews via `get_rating`, filters by rating threshold, and formats the options cleanly.




2. **Session Memory & Context-Aware Ordering:**
> *"Order the first one for me."*
> 


* The agent recalls the product selected in the previous turn from its checkpointed memory, resolves the ID, and executes `checkout`.




3. **Multimodal Visual Search:**
* Open the **Shop by Image** panel in the left sidebar.


* Upload an image of an item (e.g., honey, olive oil, coffee beans).


* Click **Find similar products**.


* The NVIDIA Vision model describes the item and extracts key search terms, which the agent then uses to search the database and present recommendations.