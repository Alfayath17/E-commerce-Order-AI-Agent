# E-commerce Order AI Agent

A customer-support AI agent that answers questions about orders (status, delivery dates, customer order history) using real data from an Excel sheet. It's built with [CrewAI](https://www.crewai.com/), and the order data is exposed to the agent as tools through an [MCP](https://modelcontextprotocol.io/) server.

You can use it three ways:

- **Streamlit chat UI**: `app.py`
- **REST API (FastAPI)**: `api.py`
- **Command line**: `agent.py`

## How it works

```
User question
     │
     ▼
Streamlit UI / FastAPI / CLI
     │
     ▼
CrewAI agent (gpt-4o-mini)
     │  calls tools over stdio
     ▼
MCP server (mcp_server.py) ──► orders.xlsx
```

`mcp_server.py` provides three tools:

| Tool | Description |
|------|-------------|
| `get_order_by_id(order_id)` | Full details of one order, e.g. `ORD1001` |
| `get_orders_by_customer(name_or_email)` | All orders for a customer, matched by name or email |
| `get_all_orders()` | Every order, for counts and status summaries |

The agent is told to always look up real data and never make up order details.

## Project structure

```
├── agent.py          # CrewAI agent, task and crew; connects to the MCP server
├── mcp_server.py     # MCP server exposing order lookup tools
├── app.py            # Streamlit chat interface
├── api.py            # FastAPI REST API
├── orders.xlsx       # Sample order data
├── requirements.txt
└── .env              # Your OpenAI API key (not committed)
```

## Setup

You need Python 3.10–3.13 (tested with 3.12).

```bash
# 1. Create and activate a virtual environment
python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Add your OpenAI API key
echo OPENAI_API_KEY=sk-your-key-here > .env
```

## Usage

### Streamlit chat UI

```bash
streamlit run app.py
```

Then open http://localhost:8501 and ask something like:

- *Where is my order ORD1003?*
- *Show all orders of Priya Sharma*
- *How many orders are still pending?*

### REST API

```bash
uvicorn api:app --reload
```

Interactive docs: http://localhost:8000/docs

```bash
curl -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "What is the status of order ORD1002?"}'
```

Response:

```json
{
  "question": "What is the status of order ORD1002?",
  "answer": "..."
}
```

### Command line

```bash
python agent.py
```

This runs one sample question and prints the answer.

## Order data

`orders.xlsx` has these columns:

`order_id`, `customer_name`, `email`, `product`, `quantity`, `price`, `order_date`, `status`, `delivery_date`, `city`, `total`

To use your own data, replace the file and keep the same column names. The MCP server reads the file again on every tool call, so edits show up without a restart.

## Configuration

| Variable | Description |
|----------|-------------|
| `OPENAI_API_KEY` | Required. Used by CrewAI to call the LLM. |

To use a different model, change `llm="gpt-4o-mini"` in `agent.py`.
