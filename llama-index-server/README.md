# LlamaIndex Server

LlamaIndexServer is a FastAPI-based application that allows you to quickly launch your [LlamaIndex Workflows](https://docs.llamaindex.ai/en/stable/module_guides/workflow/#workflows) and [Agent Workflows](https://docs.llamaindex.ai/en/stable/understanding/agent/multi_agent/) as an API server with an optional chat UI. It provides a complete environment for running LlamaIndex workflows with both API endpoints and a user interface for interaction.

## Features

- Serving a workflow as a chatbot
- Built on FastAPI for high performance and easy API development
- Optional built-in chat UI with extendable UI components
- Prebuilt development code

## Installation

```bash
pip install llama-index-server
```

## Quick Start

```python
# main.py
from llama_index.core.agent.workflow import AgentWorkflow
from llama_index.core.workflow import Workflow
from llama_index.core.tools import FunctionTool
from llama_index.server import LlamaIndexServer


# Define a factory function that returns a Workflow or AgentWorkflow
def create_workflow() -> Workflow:
    def fetch_weather(city: str) -> str:
        return f"The weather in {city} is sunny"

    return AgentWorkflow.from_tools(
        tools=[
            FunctionTool.from_defaults(
                fn=fetch_weather,
            )
        ]
    )


# Create an API server for the workflow
app = LlamaIndexServer(
    workflow_factory=create_workflow,  # Supports Workflow or AgentWorkflow
    env="dev",  # Enable development mode
    ui_config={ # Configure the chat UI, optional
        "app_title": "Weather Bot",
        "starter_questions": ["What is the weather in LA?", "Will it rain in SF?"],
    },
    verbose=True
)
```

## Running the Server

- In the same directory as `main.py`, run the following command to start the server:

  ```bash
  fastapi dev
  ```

- Making a request to the server:

  ```bash
  curl -X POST "http://localhost:8000/api/chat" -H "Content-Type: application/json" -d '{"message": "What is the weather in Tokyo?"}'
  ```

- See the API documentation at `http://localhost:8000/docs`
- Access the chat UI at `http://localhost:8000/` (Make sure you set the `env="dev"` or `include_ui=True` in the server configuration)

## Configuration Options

The LlamaIndexServer accepts the following configuration parameters:

- `workflow_factory`: A callable that creates a workflow instance for each request
- `logger`: Optional logger instance (defaults to uvicorn logger)
- `use_default_routers`: Whether to include default routers (chat, static file serving)
- `env`: Environment setting ('dev' enables CORS and UI by default)
- `ui_config`: UI configuration as a dictionary or UIConfig object with options:
  - `enabled`: Whether to enable the chat UI (default: True)
  - `app_title`: The title of the chat application (default: "LlamaIndex Server")
  - `starter_questions`: List of starter questions for the chat UI (default: None)
  - `ui_path`: Path for downloaded UI static files (default: ".ui")
  - `component_dir`: The directory for custom UI components rendering events emitted by the workflow. The default is None, which does not render custom UI components.
  - `llamacloud_index_selector`: Whether to show the LlamaCloud index selector in the chat UI (default: False). Requires `LLAMA_CLOUD_API_KEY` to be set.
- `verbose`: Enable verbose logging
- `api_prefix`: API route prefix (default: "/api")
- `server_url`: The deployment URL of the server (default is None)

## Default Routers and Features

### Chat Router

The server includes a default chat router at `/api/chat` for handling chat interactions.

### Static File Serving

- The server automatically mounts the `data` and `output` folders at `{server_url}{api_prefix}/files/data` (default: `/api/files/data`) and `{server_url}{api_prefix}/files/output` (default: `/api/files/output`) respectively.
- Your workflows can use both folders to store and access files. As a convention, the `data` folder is used for documents that are ingested and the `output` folder is used for documents that are generated by the workflow.
- The example workflows from `create-llama` (see below) are following this pattern.

### Chat UI

When enabled, the server provides a chat interface at the root path (`/`) with:

- Configurable starter questions
- Real-time chat interface
- API endpoint integration

### Custom UI Components

You can add custom UI components for your workflow by providing `component_dir` config and adding custom .jsx or .tsx files to the directory.
See [Custom UI Components](docs/custom_ui_component.md) for more details.

## Development Mode

In development mode (`env="dev"`), the server:

- Enables CORS for all origins
- Automatically includes the chat UI
- Provides more verbose logging

## API Endpoints

The server provides the following default endpoints:

- `/api/chat`: Chat interaction endpoint
- `/api/files/data/*`: Access to data directory files
- `/api/files/output/*`: Access to output directory files

## Best Practices

1. Always provide a workflow factory that creates fresh workflow instances
2. Use environment variables for sensitive configuration
3. Enable verbose logging during development
4. Configure CORS appropriately for your deployment environment
5. Use starter questions to guide users in the chat UI

## Getting Started with a New Project

Want to start a new project with LlamaIndexServer? Check out our [create-llama](https://github.com/run-llama/create-llama) tool to quickly generate a new project with LlamaIndexServer.
