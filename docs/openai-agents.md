# Using Aavenir Skills with the OpenAI Agents SDK

The [OpenAI Agents SDK](https://openai.github.io/openai-agents-python/) (2026) is the modern path for production agent applications. Each Aavenir skill becomes an `Agent` with a Pydantic `output_type`, and AVY orchestrates the four specialists via `handoffs`.

## Install

```bash
pip install openai-agents
# or
npm install @openai/agents
```

## Single-skill agent (Python)

```python
from agents import Agent, Runner
from pydantic import BaseModel
from pathlib import Path

# Define the typed output (mirrors skills/contract-intelligence/schema.json)
class ContractRisk(BaseModel):
    clause: str
    severity: str   # "low" | "medium" | "high" | "critical"
    rationale: str

class ContractReview(BaseModel):
    parties: list[str]
    effective_date: str | None
    term: str | None
    risks: list[ContractRisk]
    missing_clauses: list[str]

prompt = Path("skills/contract-intelligence/prompt.md").read_text()

contract_agent = Agent(
    name="aavenir-contract-intelligence",
    instructions=prompt,
    model="gpt-5.5",
    output_type=ContractReview,
)

result = Runner.run_sync(
    contract_agent,
    input="Review this MSA and flag legal risks: ...",
)

print(result.final_output)  # ContractReview instance, already validated
```

Setting `output_type` to a Pydantic model is the **preferred** way to get structured output in the Agents SDK. The SDK auto-enables OpenAI's structured-outputs mode under the hood; you don't need to pass `response_format`.

## Multi-agent orchestration (AVY pattern)

AVY routes incoming questions to the right specialist via `handoffs`:

```python
from agents import Agent, Runner, handoff
from pathlib import Path

def load(skill: str) -> str:
    return Path(f"skills/{skill}/prompt.md").read_text()

contract_agent     = Agent(name="contract-intelligence",     instructions=load("contract-intelligence"))
procurement_agent  = Agent(name="procurement-intelligence",  instructions=load("procurement-intelligence"))
negotiation_agent  = Agent(name="negotiation-copilot",       instructions=load("negotiation-copilot"))
obligation_agent   = Agent(name="obligation-monitor",        instructions=load("obligation-monitor"))

avy = Agent(
    name="avy",
    instructions=load("avy"),
    model="gpt-5.5",
    handoffs=[
        handoff(contract_agent,    tool_description="Use for contract review, risk analysis, clause extraction"),
        handoff(procurement_agent, tool_description="Use for vendor comparison, RFP scoring, TCO"),
        handoff(negotiation_agent, tool_description="Use for clause rewriting, redlining, plain-English translation"),
        handoff(obligation_agent,  tool_description="Use for post-signature tracking, obligations, renewals"),
    ],
)

result = Runner.run_sync(
    avy,
    input="Which vendors with compliance below 80% have contracts renewing in Q3?",
)
print(result.final_output)
```

AVY natively delegates to the matching specialist — the routing logic encoded in `skills/avy/SKILL.md` becomes the orchestrator's instructions.

## Skills as callable tools

Each Aavenir skill can also be wrapped as an OpenAI **function tool** for an existing agent stack (rather than embedded as an Agent):

```python
from agents import function_tool
from pathlib import Path
import json

@function_tool
def aavenir_contract_intelligence(contract_text: str) -> dict:
    """Analyze a contract for risks, missing clauses, and obligations.
    Returns JSON conforming to skills/contract-intelligence/schema.json."""
    # In production: call an internal skill agent or a hosted endpoint
    ...

parent = Agent(
    name="legal-ops-assistant",
    instructions="You are a legal-ops orchestrator. Use Aavenir tools for contract analysis.",
    model="gpt-5.5",
    tools=[aavenir_contract_intelligence],
)
```

Useful when Aavenir capabilities are one piece of a larger workflow (e.g., a sales engineering assistant that also reads CRM and sends Slack messages).

## Direct Responses API (without the Agents SDK)

If you don't need handoffs or the agent loop, the bare Responses API with `json_schema` is the lightest option:

```python
from openai import OpenAI
import json, pathlib

client = OpenAI()
skill = pathlib.Path("skills/contract-intelligence")
prompt = (skill / "prompt.md").read_text()
schema = json.loads((skill / "schema.json").read_text())

response = client.responses.create(
    model="gpt-5.5",
    instructions=prompt,
    input="Review this MSA: ...",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "contract_intelligence_output",
            "schema": schema,
            "strict": True,
        },
    },
)

result = response.output_parsed
```

## Assistants API (legacy / batch)

For long-running threaded batch use, the Assistants API still works — but for new builds prefer the Agents SDK above.

## Model recommendations (2026)

| Use case | Model |
|---|---|
| Batch contract review | `gpt-5-mini` |
| Interactive Q&A | `gpt-5.5` |
| Complex multi-skill orchestration (AVY) | `gpt-5.5` or `o3` |
| Negotiation rewrites (creative) | `gpt-5.5` at `temperature: 0.3` |

## Caching & cost

- OpenAI prompt caching is automatic for system prompts >1024 tokens on GPT-5-class models. Cache hits reduce cost ~50% on the cached portion.
- For high-volume batched workloads, use the **Batch API** for an additional ~50% discount on each of the five skills.
- The Agents SDK reuses cached prompts across `Runner.run_sync` calls automatically when instructions match — useful for the AVY router on a long-running session.
