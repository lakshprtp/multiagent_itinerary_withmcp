# 🌍 Autonomous Multi-Agent Travel Planning System

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![LangGraph](https://img.shields.io/badge/LangGraph-v1.2-orange.svg)](https://github.com/langchain-ai/langgraph)
[![MCP](https://img.shields.io/badge/Model_Context_Protocol-Integrated-green.svg)](https://modelcontextprotocol.io/)
[![Streamlit](https://img.shields.io/badge/Streamlit-UI-red.svg)](https://streamlit.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

An intelligent, production-ready travel planning system powered by **LangGraph**, **Groq LLMs**, and the **Model Context Protocol (MCP)**. It coordinates specialized autonomous agents (Flights, Hotels, Weather, Budget, Itinerary) managed by a Supervisor Agent with strict input guardrails, persistent PostgreSQL session memory, and interactive **Human-in-the-Loop (HITL)** approval.

---

## 📑 Table of Contents

- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Specialized Agents](#-specialized-agents)
- [Model Context Protocol (MCP) Tools](#-model-context-protocol-mcp-tools)
- [Tech Stack](#-tech-stack)
- [Prerequisites & API Keys](#-prerequisites--api-keys)
- [Installation & Setup](#-installation--setup)
- [Running the Application](#-running-the-application)
- [Example Usage](#-example-usage)
- [Project Structure](#-project-structure)
- [Troubleshooting & FAQs](#-troubleshooting--faqs)

---

## 🚀 Key Features

* **Hierarchical Multi-Agent Graph**: Built with **LangGraph** StateGraph, coordinating tasks dynamically via supervisor routing.
* **Input Guardrail**: Evaluates and validates incoming user requests before executing agent subroutines.
* **Model Context Protocol (MCP)**: Communicates with external tools via standard MCP interfaces:
  * **Tavily MCP**: Live web discovery for hotels, neighborhoods, and local insights.
  * **AviationStack MCP**: Real-time flight search, airports, and airline schedules.
  * **Weather MCP**: Local FastMCP server providing live weather conditions and 5-day forecasts via OpenWeatherMap.
* **State Persistence & Memory**: Checkpointed state using **PostgreSQL** (`PostgresSaver`) allowing session resumption across threads.
* **Human-in-the-Loop (HITL)**: Uses LangGraph's `interrupt()` pattern to generate a draft plan, present it to the traveler for review/feedback, and polish the final itinerary accordingly.
* **Interactive Web Interface**: Streamlit UI featuring thread-based sessions, live status spinners, and modular inspection of individual agent outputs.

---

## 🏛 System Architecture

```mermaid
flowchart TD
    User([Traveler Request]) --> Guardrail{Input Guardrail}
    Guardrail -- Invalid Query --> Reject[Reject & Block Request]
    Guardrail -- Allowed --> Supervisor[Supervisor Agent]
    
    Supervisor --> Route{Conditional Router}
    Route --> Flight[Flight Agent<br><i>AviationStack MCP</i>]
    Route --> Hotel[Hotel Agent<br><i>Tavily Search MCP</i>]
    Route --> Weather[Weather Agent<br><i>OpenWeather FastMCP</i>]
    Route --> Budget[Budget Agent<br><i>Cost Analysis</i>]
    
    Flight --> Hotel
    Hotel --> Weather
    Weather --> Budget
    Budget --> Itinerary[Itinerary Agent<br><i>Draft Schedule Creation</i>]
    
    Itinerary --> HITL{Human Approval Step<br><i>LangGraph Interrupt</i>}
    HITL -- Approved --> Final[Final Response Agent<br><i>Polished Travel Plan</i>]
    HITL -- Feedback Provided --> Final
    Final --> Display([Complete User Itinerary])
```

---

## 🤖 Specialized Agents

| Agent | Responsibility | Primary Data Source |
| :--- | :--- | :--- |
| **Supervisor Agent** | Validates user queries via input guardrail and selects required specialist agents and trip constraints. | Groq LLM |
| **Flight Agent** | Identifies departure/arrival airports, potential airlines, flight durations, and booking advice. | AviationStack MCP |
| **Hotel Agent** | Researches top neighborhoods and accommodations matching traveler preferences and budget constraints. | Tavily Search MCP |
| **Weather Agent** | Fetches live weather conditions and forecasts to recommend seasonal activities and packing essentials. | Weather FastMCP (OpenWeather) |
| **Budget Agent** | Synthesizes flight, accommodation, and daily expenses to provide realistic feasibility assessments. | Groq LLM |
| **Itinerary Agent** | Generates a structured, day-by-day draft itinerary ready for human review. | Groq LLM |
| **Human Approval** | Pauses graph execution with LangGraph `interrupt()` to collect user approval and revision requests. | Streamlit UI |
| **Final Response Agent** | Produces the final polished travel plan incorporating human feedback. | Groq LLM |

---

## 🔌 Model Context Protocol (MCP) Tools

The system uses `MultiServerMCPClient` from `langchain-mcp-adapters` to manage tool execution:

1. **Tavily MCP** (`streamable_http`): Connected directly to Tavily's hosted MCP endpoint for search, extraction, and content scraping.
2. **AviationStack MCP** (`stdio`): Runs as a subprocess executing the `aviationstack_mcp` server.
3. **Weather MCP** (`stdio`): Local FastMCP server (`mcp_server.py`) querying the OpenWeatherMap API for live conditions and 5-day forecasts.

---

## 🛠 Tech Stack

- **Framework**: LangGraph, LangChain Core, LangChain Community
- **LLM Engine**: Groq Cloud (`openai/gpt-oss-20b` or `openai/gpt-oss-120b`)
- **Protocol**: Model Context Protocol (MCP) via `langchain-mcp-adapters`
- **Database**: PostgreSQL (with `psycopg` and `langgraph-checkpoint-postgres`)
- **Frontend**: Streamlit
- **APIs**: Tavily, OpenWeatherMap, AviationStack

---

## 🔑 Prerequisites & API Keys

Before running the project, obtain API keys from the following services:

* **Groq Cloud**: [https://console.groq.com/](https://console.groq.com/)
* **Tavily AI**: [https://tavily.com/](https://tavily.com/)
* **OpenWeatherMap**: [https://openweathermap.org/](https://openweathermap.org/) *(Note: New keys can take 1–2 hours to activate)*
* **AviationStack**: [https://aviationstack.com/](https://aviationstack.com/)
* **PostgreSQL**: Local or hosted instance (e.g., Supabase, Neon, or local Postgres)

---

## 📦 Installation & Setup

### 1. Clone the Repository
```bash
git clone https://github.com/<your-username>/multiagent_itinerary_planner.git
cd multiagent_itinerary_planner
```

### 2. Create and Activate a Virtual Environment
```bash
# macOS/Linux
python3 -m venv .venv
source .venv/bin/activate

# Windows
python -m venv .venv
.venv\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure Environment Variables
Copy the template file to `.env`:
```bash
cp .env.example .env
```
Fill in your credentials in `.env`:
```env
GROQ_API_KEY=gsk_your_groq_api_key
GROQ_MODEL=openai/gpt-oss-20b

TAVILY_API_KEY=tvly-your_tavily_key
OPENWEATHER_API_KEY=your_openweather_key
AVIATION_STACK_API_KEY=your_aviationstack_key

DATABASE_URL=postgresql://postgres:postgres@localhost:5432/multiagent_itinerary_planner
```

### 5. Setup PostgreSQL Database
Ensure PostgreSQL is running and create the database:
```sql
CREATE DATABASE multiagent_itinerary_planner;
```
*(The LangGraph checkpointer will automatically initialize tables when the app launches).*

### 6. Setup MCP Servers

#### Weather MCP Server:
Ensure your `weather_mcp` environment has dependencies installed:
```bash
pip install mcp requests python-dotenv
```

#### AviationStack MCP Server:
Clone and sync dependencies using `uv`:
```bash
git clone https://github.com/Pradumnasaraf/aviationstack-mcp.git
cd aviationstack-mcp
uv sync
```

*(Ensure the paths in `mcp_client.py` point to your local virtual environments and server scripts).*

---

## 🖥 Running the Application

Launch the Streamlit web application:

```bash
streamlit run frontend.py
```

Open your browser at `http://localhost:8501`.

---

## 💡 Example Usage

1. **Enter a prompt in the text area**:
   > *"Plan a 4-day trip to Tokyo with budget hotels and weather forecast under $1,500."*
2. **Click "Create Draft Plan"**:
   * The Supervisor routes the request.
   * Flights, accommodations, weather, and budget are analyzed.
   * A draft itinerary is displayed.
3. **Human Approval**:
   * Review the draft plan under the **Human Approval** section.
   * Select **"Yes"** to approve or **"No, revise it"** with custom feedback (e.g., *"Focus more on cultural shrines and street food"*).
4. **View Final Plan**:
   * Click **Submit Approval** to receive the final refined itinerary.

---

## 📂 Project Structure

```text
├── agents.py           # Specialized agent definitions and supervisor logic
├── config.py           # Environment loading and Groq LLM setup
├── frontend.py         # Streamlit web interface with HITL controls
├── graph.py            # LangGraph StateGraph, conditional edges, and Postgres checkpointer
├── mcp_client.py       # MultiServerMCPClient configuration (Tavily, AviationStack, Weather)
├── state.py            # TravelState TypedDict schema definition
├── requirements.txt    # Python dependencies
├── .env.example        # Environment variable template
├── .gitignore          # Git exclusion rules
└── README.md           # Project documentation
```

---

## ❓ Troubleshooting & FAQs

* **OpenWeather returns HTTP 401**:
  * Newly created OpenWeather API keys take 1–2 hours to activate globally. Verify with:
    `curl "https://api.openweathermap.org/data/2.5/weather?q=Tokyo&appid=YOUR_KEY"`
* **Groq Rate Limit Exceeded (Error 413 - TPM Limit)**:
  * In `.env`, switch to `GROQ_MODEL=openai/gpt-oss-20b`, which offers significantly higher token limits than `openai/gpt-oss-120b`.
* **AviationStack `function_access_restricted`**:
  * The free tier of AviationStack restricts reference data endpoints (`/airports`, `/airlines`). The flight agent will gracefully use its knowledge base to estimate flights.
* **MCP Stdio Connection Errors**:
  * Check [mcp_client.py](mcp_client.py) to ensure the Python interpreter path and server script path are valid on your operating system.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

