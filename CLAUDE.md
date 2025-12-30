# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a LinkedIn Learning course repository for "Hands-On AI: Building Agents with Google's Agent Development Toolkit (ADK)". It contains Jupyter notebooks demonstrating how to build AI agents using Google's ADK framework.

## Running Notebooks

```bash
# Install dependencies
pip install google-adk

# Optional dependencies for specific notebooks
pip install langchain_community python-dateutil requests
```

Notebooks are designed to run in Jupyter environments. Each notebook installs its own dependencies via `%pip install`.

## Environment Configuration

Required environment variables:
- `GOOGLE_GENAI_API_KEY` - Google AI API key (get from https://aistudio.google.com/apikey)
- `OPEN_WEATHER_API_KEY` - OpenWeatherMap API key (required for weather agent notebooks, get from https://openweathermap.org/api)

Set `GOOGLE_GENAI_USE_VERTEXAI=False` to use the public Gemini API directly (default in all notebooks).

## Notebook Structure

Notebooks follow a chapter/lesson naming convention (`XX_YY.ipynb`):

- **01_03**: ADK quickstart - basic agent definition, Runner, InMemorySessionService
- **02_03-02_04**: LLM agents with packing list and itinerary examples, google_search tool integration
- **03_01-03_03**: Built-in tools (google_search), custom function tools (weather API integration), third-party integrations
- **04_02-04_06**: Workflow agents (ParallelAgent, SequentialAgent), output_key for agent communication, stateful agents with tool_context.state

## Key ADK Patterns

### Basic Agent Definition
```python
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService

agent = Agent(
    name="agent_name",
    model="gemini-2.0-flash",
    description="Agent description",
    instruction="Agent behavior instructions",
    tools=[],  # Optional: list of tools
    output_key="key_name"  # Optional: for passing output to other agents
)
```

### Running Agent Interactions
```python
session_service = InMemorySessionService()
await session_service.create_session(app_name=app_name, user_id=user_id, session_id=session_id)

runner = Runner(agent=agent, app_name=app_name, session_service=session_service)
events = runner.run(user_id=user_id, session_id=session_id, new_message=content)

for event in events:
    if event.is_final_response():
        response = event.content.parts[0].text
```

### Workflow Agents
```python
from google.adk.agents import ParallelAgent, SequentialAgent

# Run agents concurrently
parallel = ParallelAgent(name="parallel", sub_agents=[agent1, agent2])

# Chain agents sequentially (outputs passed via output_key)
sequential = SequentialAgent(name="sequential", sub_agents=[parallel, finalizer])
```

### Stateful Tools
Custom tools can access session state via `tool_context` parameter:
```python
def stateful_tool(city: str, tool_context) -> dict:
    preference = tool_context.state.get("user_preference", "default")
    tool_context.state["last_value"] = city
    return {"status": "success", "data": ...}
```

## Common Imports

```python
from google.adk.agents import Agent, ParallelAgent, SequentialAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import google_search
from google.genai import types
```
