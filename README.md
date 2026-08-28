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
- Pausing execution for human assistance with LangGraph interrupts
- Resuming a paused conversation with checkpointed state

The tool-enabled flow is:

```text
User message -> LLM -> tool call -> ToolNode -> LLM -> final response
```

The human-in-the-loop flow in `two.ipynb` is:

```text
User message -> LLM -> human_assistance -> interrupt
									  |
						  Command(resume=...) -> LLM -> final response
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

## Run The Notebooks

Open either notebook in VS Code or Jupyter and run its cells from top to bottom.
Both notebooks use this model through the Groq provider:

```python
llm = init_chat_model(
	"qwen/qwen3.6-27b",
	model_provider="groq",
)
```

### Basic Chatbot

[basicchatbot/one.ipynb](basicchatbot/one.ipynb) demonstrates basic graph
construction, streaming, Tavily web search, and tool routing.

Example tool-enabled input:

```python
graph.invoke({"messages": "what happened in Bangalore today?"})
```

For a web-search request, the LLM should produce a Tavily tool call. The graph
then executes Tavily and sends its result back to the LLM for a natural-language
answer.

### Human In The Loop

[basicchatbot/two.ipynb](basicchatbot/two.ipynb) adds a `human_assistance` tool.
When the model calls this tool, `interrupt(...)` pauses execution and the graph
stores its state with `MemorySaver`:

```python
memory = MemorySaver()
graph = graph_builder.compile(checkpointer=memory)
```

Use a stable `thread_id` when starting the conversation:

```python
config = {"configurable": {"thread_id": "12345"}}
```

After the graph pauses, resume it with a human response using the same config:

```python
human_response = "Please provide more details about your implementation."
human_command = Command(resume={"data": human_response})
graph.stream(human_command, config=config, stream_mode="values")
```

The notebook currently uses a fixed sample response. It does not yet collect
the human response through an interactive prompt or external operator UI.

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
│   ├── one.ipynb                          # Basic chatbot and tool routing
│   └── two.ipynb                          # Human-in-the-loop chatbot
├── src/
│   └── basicchatbotusinglanggraph/
│       └── __init__.py                   # Current package entry point
├── pyproject.toml                         # Project metadata and dependencies
├── requirements.txt                       # Alternative dependency list
└── README.md
```

