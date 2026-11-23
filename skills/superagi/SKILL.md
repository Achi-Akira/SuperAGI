---
name: superagi
description: Install, configure and use SuperAGI, an open-source autonomous AI agent framework with a web GUI, including running it with Docker Compose, filling in config.yaml (model keys, vector databases, tool keys), creating and running agents with toolkits such as web search, GitHub, email, Slack and Jira, running local LLMs on a GPU, and writing a new toolkit in Python. Use when the user mentions SuperAGI, wants to run autonomous agents locally, or needs to debug a SuperAGI install, agent run or toolkit.
---

# SuperAGI

SuperAGI runs autonomous agents that pursue goals, using toolkits (search, files, code, GitHub, email, Slack, Jira and more), with a web GUI, an action console for permissions, memory and vector databases.

The project says it's still under active development and may have issues. Check its open issues when something fails.

## Run locally

1. Copy `config_template.yaml` to `config.yaml` and fill in the keys you need (below).
2. With Docker running, from the repository folder:
   ```bash
   docker compose -f docker-compose.yaml up --build
   ```
   With a GPU and local LLMs, use `docker compose -f docker-compose-gpu.yml up --build` instead.
3. Open http://localhost:3000.

A hosted version also exists at app.superagi.com, where you add your model key under Model Providers.

## config.yaml

| Keys | Purpose |
|---|---|
| `OPENAI_API_KEY`, `OPENAI_API_BASE`, `MODEL_NAME` | The main model. `OPENAI_API_BASE` points at any OpenAI-compatible server |
| `PALM_API_KEY`, `REPLICATE_API_TOKEN`, `HUGGING_API_TOKEN` | Other model providers |
| `MAX_MODEL_TOKEN_LIMIT`, `MAX_TOOL_TOKEN_LIMIT` | Token limits |
| `PINECONE_API_KEY`, `PINECONE_ENVIRONMENT`, `WEAVIATE_USE_EMBEDDED` | Vector databases for memory |
| `DB_*`, `REDIS_URL` | Postgres and Redis, set by Docker Compose |
| `STORAGE_TYPE`, `BUCKET_NAME`, `AWS_*`, `RESOURCES_*_ROOT_DIR` | File storage, local or S3 |
| `GOOGLE_API_KEY`, `SEARCH_ENGINE_ID`, `SERP_API_KEY` | Search tools |
| `EMAIL_*`, `GITHUB_USERNAME`, `GITHUB_ACCESS_TOKEN`, `JIRA_*`, `SLACK_BOT_TOKEN`, `STABILITY_API_KEY` | Toolkit credentials |
| `ENCRYPTION_KEY`, `JWT_SECRET_KEY` | Secrets. Set strong values and don't change them after setup |

Keys come from the user. Never invent them, print them or commit `config.yaml`. Leave a toolkit's keys empty if it isn't used.

## Run an agent

In the GUI, create an agent: give it a name, goals and instructions, choose the model, add toolkits, and set the permission mode. **Ask for permission** makes the agent wait in the Action Console before each tool use. Suggest that mode for anything that sends email, posts to Slack, changes GitHub or Jira, or runs code, because autonomous agents act on their own and spend model tokens on every step. Set iteration and token limits to control cost.

## Write a toolkit

Built-in toolkits are in `superagi/tools/<name>/`. Follow `superagi/tools/duck_duck_go/` as the pattern:

- A tool class extends `BaseTool` with `name`, `description`, an `args_schema` (a pydantic `BaseModel` with a `Field(..., description=...)` for each input) and `_execute(...)`, which returns the result.
- A toolkit class extends `BaseToolkit` with `name` and `description`, `get_tools()` returning the tool instances, and `get_env_keys()` returning the `ToolConfiguration` keys it needs.
- Put the tool's Python dependencies in a `requirements.txt` in its folder, and any apt packages in `requirements_apt.txt`. `install_tool_dependencies.sh` installs both in the container.
- Read config values with `self.get_tool_config("KEY")`.

Rebuild the containers after adding or changing a toolkit.

## Troubleshooting

- **Nothing on port 3000:** check `docker compose logs`. The backend waits for Postgres and Redis (`wait-for-it.sh`), and a missing `config.yaml` stops startup.
- **Model errors:** check `MODEL_NAME`, the key, and `OPENAI_API_BASE` for OpenAI-compatible servers.
- **Tool fails:** that toolkit's keys are missing in `config.yaml` or in the toolkit's settings in the GUI.
