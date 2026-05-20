# Using Aavenir Skills with OpenAI Agents

The OpenAI Agents SDK and Assistants API both support custom domain skills via system instructions and tool definitions. The Aavenir skills install as Agent profiles with strict JSON Schema outputs.

## OpenAI Agents SDK (recommended)

The Agents SDK (`openai-agents`) is the modern integration path. Each Aavenir skill becomes an Agent.

### Install

```bash
pip install openai-agents
# or
npm install @openai/agents
```

### Single-skill agent (Python)

```python
from openai_agents import Agent, Runner
from pathlib import Path
import json

skill_dir = Path("skills/contract-intelligence")
system_prompt = (skill_dir / "prompt.md").read_text()
output_schema = json.loads((skill_dir / "schema.json").read_text())

contract_agent = Agent(
    name="aavenir-contract-intelligence",
    instructions=system_prompt,
    model="gpt-4o",
    output_schema=output_schema,
)

result = Runner.run_sync(
    contract_agent,
    input="Review this MSA and flag legal risks: ..."
)

print(result.final_output)  # Parsed JSON matching schema.json
```

### Multi-agent orchestration (AVY pattern)

The AVY skill is the orchestrator. Wire the four specialist agents as **handoffs** under AVY:

```python
from openai_agents import Agent, Runner, handoff
from pathlib import Path

def load(skill: str) -> str:
    return Path(f"skills/{skill}/prompt.md").read_text()

contract_agent = Agent(name="contract-intelligence", instructions=load("contract-intelligence"))
procurement_agent = Agent(name="procurement-intelligence", instructions=load("procurement-intelligence"))
negotiation_agent = Agent(name="negotiation-copilot", instructions=load("negotiation-copilot"))
obligation_agent = Agent(name="obligation-monitor", instructions=load("obligation-monitor"))

avy = Agent(
    name="avy",
    instructions=load("avy"),
    handoffs=[
        handoff(contract_agent, tool_description="Use for contract review, risk analysis, clause extraction"),
        handoff(procurement_agent, tool_description="Use for vendor comparison, RFP scoring, TCO"),
        handoff(negotiation_agent, tool_description="Use for clause rewriting, redlining, plain-English translation"),
        handoff(obligation_agent, tool_description="Use for post-signature tracking, obligations, renewals"),
    ],
    model="gpt-4o",
)

result = Runner.run_sync(avy, input="Which vendors with compliance below 80% have contracts renewing in Q3?")
print(result.final_output)
```

AVY now natively delegates to the matching specialist via OpenAI's handoff mechanism — preserving the same routing logic encoded in `skills/avy/SKILL.md`.

## Assistants API (legacy / batch)

For batch processing or non-interactive use:

```python
from openai import OpenAI

client = OpenAI()

with open("skills/contract-intelligence/prompt.md") as f:
    instructions = f.read()

assistant = client.beta.assistants.create(
    name="Aavenir Contract Intelligence",
    instructions=instructions,
    model="gpt-4o",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "contract_intelligence_output",
            "schema": json.load(open("skills/contract-intelligence/schema.json")),
            "strict": True
        }
    }
)

thread = client.beta.threads.create()
client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="Review this MSA: ..."
)

run = client.beta.threads.runs.create_and_poll(
    thread_id=thread.id,
    assistant_id=assistant.id
)
```

## Responses API (streaming, recommended for chat UIs)

```python
response = client.responses.create(
    model="gpt-4o",
    instructions=load("avy"),
    input=user_message,
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "avy_output",
            "schema": json.load(open("skills/avy/schema.json")),
            "strict": True
        }
    }
)

result = response.output_parsed  # Already validated against schema
```

## Tool use mode (skills as tools)

Each skill can be wrapped as an OpenAI **tool** for use inside a larger agent:

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "aavenir_contract_intelligence",
            "description": "Analyze a contract for risks, missing clauses, and obligations. Returns structured JSON.",
            "parameters": json.load(open("skills/contract-intelligence/schema.json"))
        }
    }
]
```

The parent agent can now call `aavenir_contract_intelligence` as a tool — useful when integrating Aavenir capabilities into existing OpenAI Agent stacks.

## Model recommendation

| Use case | Model |
|----------|-------|
| Batch contract review | `gpt-4o-mini` |
| Interactive Q&A | `gpt-4o` |
| Complex multi-skill orchestration (AVY) | `gpt-4o` or `o1` |
| Negotiation rewrites (creative) | `gpt-4o` at `temperature: 0.3` |

## Caching & cost

OpenAI's prompt caching (automatic for >1024-token prompts on GPT-4o) covers the system prompt portion. Cache hits reduce per-call cost by ~50% on the cached portion.

For high-volume batched workloads, use the **Batch API** for additional 50% discount on each of the five skills.
