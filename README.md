# MCP Server with LangChain

A small Python demo showing how to connect **Model Context Protocol (MCP)** tools to a **LangChain** agent. The client connects to two MCP servers—a local math server over standard input/output (stdio) and a weather server over Streamable HTTP—then asks a Groq-hosted chat model to use their tools.

> This is an educational example. The weather tool currently returns a fixed demo response; it does not fetch live weather data.

## Features

- Exposes addition and multiplication as tools from a local MCP server.
- Exposes a sample weather lookup from an HTTP MCP server.
- Discovers and calls both servers' tools from a LangChain agent.
- Uses Groq for model inference and loads the API key from a local `.env` file.

## Workflow

```text
Terminal 1                           Terminal 2
weather.py                            client.py
FastMCP HTTP server                   LangChain agent
http://localhost:8000/mcp  <--------  MultiServerMCPClient
                                      |
                                      +-- mathserver.py (started by client over stdio)
                                      |
                                      +-- Groq chat model
                                           |
                                           +-- Math question -> math tools
                                           +-- Weather question -> weather tool
```

1. Start `weather.py`. It serves the weather MCP tool over Streamable HTTP.
2. Run `client.py`. The client connects to the weather server and starts `mathserver.py` as a stdio MCP subprocess.
3. The client loads the available tools and gives them to the LangChain agent.
4. The agent sends two sample questions to Groq and can call the matching tools.

## Tech stack

| Technology | Use |
| --- | --- |
| Python | Application and MCP server implementation |
| [Model Context Protocol](https://modelcontextprotocol.io/) | Standard interface for exposing and connecting to tools |
| [FastMCP](https://github.com/modelcontextprotocol/python-sdk) | Defines the math and weather MCP tools |
| [LangChain MCP Adapters](https://github.com/langchain-ai/langchain-mcp-adapters) | Connects the LangChain client to MCP servers |
| [LangGraph](https://github.com/langchain-ai/langgraph) | Runs the tool-using agent |
| [ChatGroq](https://python.langchain.com/docs/integrations/chat/groq/) | Connects the agent to a Groq-hosted chat model |
| python-dotenv | Loads `GROQ_API_KEY` from `.env` |

## Requirements

- Python **3.13 or newer** (as specified in `pyproject.toml`)
- A [Groq API key](https://console.groq.com/keys)
- Git, if cloning the repository

## Setup

Clone the repository and enter its directory:

```powershell
git clone https://github.com/krishnegowda9/MCP_server_Langchain.git
cd MCP_server_Langchain
```

Create and activate a virtual environment, then install the dependencies:

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If PowerShell blocks activation, allow it only for the current terminal session and activate again:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

Create a `.env` file in the project root (the same directory as `client.py`) with your own Groq API key:

```dotenv
GROQ_API_KEY=your_groq_api_key
```

Do not commit or share `.env`, and never put a real API key in source code, screenshots, or this README. If a key is exposed, revoke it and create a replacement.

## Run

Open **two PowerShell terminals** in the project directory and activate the virtual environment in each.

In **Terminal 1**, start the weather MCP server and leave it running:

```powershell
.\.venv\Scripts\Activate.ps1
python weather.py
```

In **Terminal 2**, start the client:

```powershell
.\.venv\Scripts\Activate.ps1
python client.py
```

The client starts `mathserver.py` automatically. It prints the agent's responses to a math question and a weather question.

## Project structure

```text
.
├── client.py          # Connects to MCP servers and runs the LangChain agent
├── mathserver.py      # Math tools served over stdio
├── weather.py         # Demo weather tool served over Streamable HTTP
├── requirements.txt   # Python package dependencies
├── pyproject.toml     # Project metadata and Python requirement
└── .env               # Local Groq API key; create this file yourself
```

## Configuration

The client currently configures the Groq model as `openai/gpt-oss-120b`. Model availability can depend on your Groq account and can change over time. If Groq reports that the model is unavailable, check the [Groq models page](https://console.groq.com/docs/models) and set `ChatGroq(model="...")` in `client.py` to a model ID available to your account.

The weather server URL is configured in `client.py` as `http://localhost:8000/mcp`. Start `weather.py` before running the client so the HTTP MCP server is reachable.

## Troubleshooting

- **`GROQ_API_KEY` is missing or invalid:** Check that `.env` is in the project root, contains `GROQ_API_KEY=...`, and uses a valid, active key.
- **Groq returns a model not found/available error:** Choose an active model ID enabled for your account from the [Groq models page](https://console.groq.com/docs/models).
- **Connection error to `localhost:8000`:** Start `python weather.py` in a separate terminal and keep it running.
- **Python version error:** Use Python 3.13 or newer, as specified by this project's `pyproject.toml`.
- **`create_react_agent` deprecation warning:** This is a warning from the installed LangGraph version, separate from Groq/API errors. The client currently uses that API; update the agent construction when migrating to the newer LangChain agent API.

## License

See [LICENSE](./LICENSE).
