# linkwork-agent-sdk

English | [中文](./README_zh-CN.md)

`linkwork-agent-sdk` is the LinkWork runtime SDK for task sessions. It handles config loading, skill/MCP wiring, security checks, and event logging.

## Core Capabilities

- `AgentEngine`: task-level execution engine
- `ConfigLoader`: strict `config.json` validation (Pydantic)
- `SkillsProvider`: load skills from `/opt/agent/skills` and sync to `.claude/skills`
- `MCPProvider`: load `/opt/agent/mcp.json`
- `SecurityEnforcer`: command/tool constraints
- `RedisClient + Logger`: Redis Stream + local fallback logs

## Local Development

### 1) Requirements

- Python 3.11+

### 2) Install

```bash
cd linkwork-agent-sdk
pip install -e .
```

Install dev dependencies:

```bash
pip install -e '.[dev]'
```

### 3) Example

```python
import asyncio
from linkwork_agent_sdk import AgentEngine

async def main():
    async with AgentEngine(config_file='config.json', task_id='task-demo', workstation_id='ws-demo') as engine:
        # call engine run/session logic from upper layer
        pass

asyncio.run(main())
```

## `config.json` Example

```json
{
  "runtime": { "provider": "claude" },
  "claude_settings": {
    "env": {},
    "model": "openrouter/anthropic/claude-sonnet-4.5",
    "language": "Chinese"
  },
  "agent": {
    "name": "demo-worker",
    "max_turns": 100,
    "max_thinking_tokens": 10000,
    "permission_mode": "default",
    "allowed_tools": [],
    "disallowed_tools": [],
    "can_use_tools": [],
    "zz_enabled": false
  },
  "system_prompt": {
    "use_preset": true,
    "preset": "claude_code",
    "append": ""
  }
}
```

## Deploy Flow

### Option A: Bundle into LinkWork role image (primary path)

In `LinkWork/backend`, `build.sh` injects SDK sources into image build context and installs them into role images (`/opt/linkwork-agent-build/sdk-source`).

Use this path for production task runtime.

### Option B: Publish as Python package (optional)

```bash
cd linkwork-agent-sdk
python -m build
# twine upload dist/*   # use your internal release process
```

Use this path for external reuse or CI package distribution.
