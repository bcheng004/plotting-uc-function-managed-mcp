# Plotting with Unity Catalog Functions and Managed MCP

This repository demonstrates how to build an AI agent that can query data and create visualizations using Databricks Genie Spaces, Unity Catalog Functions via Databricks Managed MCP (Model Context Protocol) servers. The agent uses LangGraph to orchestrate tool calls and is deployed using Mosaic AI Agent Framework.

## Repository Contents

### Notebooks
- [01-uc-plotting-function.ipynb](01-uc-plotting-function.ipynb) - Creates a Unity Catalog function that transforms Genie query results into Plotly charts
- [02-langgraph-mcp-tool-calling-agent.ipynb](02-langgraph-mcp-tool-calling-agent.ipynb) - Builds, tests, and deploys a LangGraph agent that uses MCP to connect to Genie and UC functions
- [03-deploy-run-databricks-app.ipynb](03-deploy-run-databricks-app.ipynb) - Deploys the agent as a Databricks App with a chat interface

### Python Files
- [agent.py](agent.py) - Main agent implementation with MCP tool integration
- [databricks_apps/databricks_chat_app/app.py](databricks_apps/databricks_chat_app/app.py) - Streamlit chat interface for the Databricks App
- [databricks_apps/databricks_chat_app/agent_endpoint_client.py](databricks_apps/databricks_chat_app/agent_endpoint_client.py) - Client for querying the deployed agent endpoint

## Prerequisites

- Databricks workspace with access to:
  - Unity Catalog
  - Model Serving endpoints
  - Databricks Apps
  - Genie Space
- A dataset to query (example uses `samples.nyctaxi.trips`)
- LLM endpoint (example uses `databricks-claude-3-7-sonnet`)

## Setup Instructions

Follow these notebooks in order:

### Step 1: Create the Plotting Function
Run [01-uc-plotting-function.ipynb](01-uc-plotting-function.ipynb)

This notebook creates a Unity Catalog function called `genie_to_chart` that:
- Takes JSON output from Genie MCP queries
- Transforms the data into Plotly visualizations
- Supports bar, line, and pie chart types

**Configuration:**
- Set the `catalog`, `schema`, and `space_id` widgets at the top of the notebook
- The function will be created at `{catalog}.{schema}.genie_to_chart`

### Step 2: Create a Databricks Genie Space
**IMPORTANT: Before running notebook 02, create a Genie Space in your Databricks workspace**

1. Navigate to your Databricks workspace UI
2. Go to the Genie Spaces section
3. Create a new Genie Space with access to your data (e.g., `samples.nyctaxi.trips`)
4. Copy the Genie Space ID from the URL (format: `01f0ab8c079d17b8a00584e70d2ac18c`)
5. You will need this Space ID for the next notebook

### Step 3: Build and Deploy the MCP Agent
Run [02-langgraph-mcp-tool-calling-agent.ipynb](02-langgraph-mcp-tool-calling-agent.ipynb)

This notebook:
- Configures MCP connections to both Genie Space and UC Functions
- Creates a LangGraph agent that can query data and create charts
- Tests the agent locally with MLflow tracing
- Registers the agent to Unity Catalog
- Deploys the agent as a Model Serving endpoint

**Configuration:**
- Set the `catalog`, `schema`, and `space_id` widgets
- Update `GENIE_SPACE_ID` in the agent code (cell 5) with your Genie Space ID
- Optionally configure OAuth for custom MCP servers (most users can skip this)

**Key workflow:**
1. Agent receives user request (e.g., "Show me trips by day as a line chart")
2. Calls `query_space` tool to get data from Genie
3. Calls `genie_to_chart` UC function to create visualization
4. Returns the Plotly chart to the user

### Step 4: Deploy the Chat Application
Run [03-deploy-run-databricks-app.ipynb](03-deploy-run-databricks-app.ipynb)

This notebook:
- Deploys a Streamlit-based chat interface as a Databricks App
- Connects the chat UI to the agent endpoint from Step 3

**Configuration:**
- Set the `app_name` widget (default: `managed-mcp-plotting-app`)
- Set the `source_code_path` widget to point to the chat app directory

After deployment, you'll have a user-friendly chat interface to interact with your agent.

## Example Usage

Once deployed, you can ask the agent questions like:
- "How many trips were taken each day in 2016? Show me a line chart"
- "What were the total trips by borough? Create a bar chart"
- "Show me payment types as a pie chart"

The agent will:
1. Query your Genie Space for the data
2. Transform the results into a Plotly visualization
3. Display the interactive chart

## Architecture

```
User Query
    ↓
LangGraph Agent (with MCP)
    ↓
├── Managed MCP: Genie Space (query_space tool)
│       ↓
│   Returns data as JSON
│
├── Managed MCP: UC Functions (genie_to_chart)
│       ↓
│   Transforms JSON → Plotly chart
│
└── Returns visualization to user
```

## Troubleshooting

- **Authentication errors**: Ensure your workspace has proper permissions for MCP, Model Serving, and Apps
- **Tool not found**: Verify that the Genie Space ID and UC function names match in the agent code
- **Chart rendering issues**: Check that the Genie query returns data in the expected format
- **Deployment failures**: Wait for endpoints to fully deploy (can take 10-20 minutes)

## Learn More

- [MCP on Databricks](https://docs.databricks.com/aws/en/generative-ai/mcp/)
- [Mosaic AI Agent Framework](https://docs.databricks.com/aws/generative-ai/agent-framework/author-agent)
- [Unity Catalog Functions](https://docs.databricks.com/en/sql/language-manual/sql-ref-functions-udf.html)
- [Databricks Apps](https://docs.databricks.com/en/dev-tools/databricks-apps/)
