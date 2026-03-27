# linkwork-agent-sdk

`linkwork-agent-sdk` 是 LinkWork Agent 运行时 SDK，负责单任务会话中的配置加载、技能装配、MCP 装配、安全检查与日志上报。

## 核心能力

- `AgentEngine`：任务级执行引擎
- `ConfigLoader`：`config.json` 严格校验（Pydantic）
- `SkillsProvider`：加载 `/opt/agent/skills` 下技能并同步到 `.claude/skills`
- `MCPProvider`：加载 `/opt/agent/mcp.json`
- `SecurityEnforcer`：命令与工具权限约束
- `RedisClient + Logger`：Redis Stream 与本地回退日志

## 本地开发

### 1) 环境要求

- Python 3.11+

### 2) 安装

```bash
cd linkwork-agent-sdk
pip install -e .
```

开发依赖：

```bash
pip install -e '.[dev]'
```

### 3) 运行示例

```python
import asyncio
from linkwork_agent_sdk import AgentEngine

async def main():
    async with AgentEngine(config_file='config.json', task_id='task-demo', workstation_id='ws-demo') as engine:
        # 按业务调用 engine.run(...) / session loop（由上层调度）
        pass

asyncio.run(main())
```

## 配置文件示例（`config.json`）

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

## Deploy 流程

### 方案 A：随 LinkWork 角色镜像发布（主路径）

在 `LinkWork/back` 的构建流程中，`build.sh` 会将 SDK 源码打入构建上下文并安装到角色镜像（`/opt/linkwork-agent-build/sdk-source`）。

适用场景：生产任务运行时。

### 方案 B：作为 Python 包发布（可选）

```bash
cd linkwork-agent-sdk
python -m build
# twine upload dist/*  # 按你们内部仓库流程执行
```

适用场景：给外部项目或 CI 独立复用 SDK。
