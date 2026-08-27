# Basic Chatbot Using LangGraph

A small Python learning project that demonstrates how to build a chatbot with
[LangGraph](https://langchain-ai.github.io/langgraph/). The main implementation
is an exploratory Jupyter notebook rather than a production application.

## What This Project Demonstrates

- Defining chatbot state with `TypedDict` and `add_messages`
- Building a `StateGraph` with start, node, conditional, and end edges
- Calling a Groq-hosted chat model through `ChatGroq`
- Streaming responses from a graph
- Binding tools to an LLM with `llm.bind_tools(tools)`
- Routing tool calls through LangGraph's `ToolNode`
- Searching the web with Tavily
- Calling a local `multiply` tool

The tool-enabled flow is:

```text
User message -> LLM -> tool call -> ToolNode -> LLM -> final response
```

## Requirements

- Python 3.14 or newer
- A Groq API key
- A Tavily API key for web-search questions
- `uv` is recommended for dependency management

## Setup

Clone the repository and install its dependencies:

```powershell
uv sync
```

Alternatively, install the unpinned dependencies from `requirements.txt`:

```powershell
uv venv
uv pip install -r requirements.txt
```

Create a `.env` file in the project root:

```dotenv
GROQ_API_KEY=your-groq-api-key
TAVILY_API_KEY=your-tavily-api-key
```

Do not commit `.env` or expose either key in notebook output.

## Run The Notebook

Open [basicchatbot/one.ipynb](basicchatbot/one.ipynb) in VS Code or Jupyter and
run the cells from top to bottom. The notebook uses this Groq model:

```python
qwen/qwen3.6-27b
```

Example tool-enabled input:

```python
graph.invoke({"messages": "what happened in Bangalore today?"})
```

For a web-search request, the LLM should produce a Tavily tool call. The graph
then executes Tavily and sends its result back to the LLM for a natural-language
answer.

## Package Command

The project exposes a command through `pyproject.toml`:

```powershell
uv run basicchatbotusinglanggraph
```

At present, this command only prints a greeting. The chatbot implementation is
currently in the notebook and is not wired into the package entry point.

## Project Structure

```text
.
├── basicchatbot/
│   └── one.ipynb                         # LangGraph tutorial notebook
├── src/
│   └── basicchatbotusinglanggraph/
│       └── __init__.py                   # Current package entry point
├── pyproject.toml                         # Project metadata and dependencies
├── requirements.txt                       # Alternative dependency list
└── README.md
```

