# Skill: Ollama Agent Pattern

---
name: ollama-agent-pattern
description: Use this skill when building any agents/ file. Defines ChatOllama setup, async invoke pattern, JSON parse with fence stripping, AgentParseError, and system prompt rules.
---

## Setup
from langchain_ollama import ChatOllama

class AgentParseError(Exception):
    pass

class <Domain>Agent:
    def __init__(self, model: str):
        self.llm = ChatOllama(
            model=model,
            temperature=0,
            format="json"
        )

## Async Invoke Pattern
async def extract(self, input_text: str) -> <OutputModel>:
    prompt = self._build_prompt(input_text)
    response = await self.llm.ainvoke(prompt)
    return self._parse(response.content)

## JSON Parse Pattern
Always strip markdown fences before parsing.
Always raise AgentParseError with raw response on failure.

def _parse(self, raw: str) -> <OutputModel>:
    clean = raw.strip()
    if clean.startswith("```"):
        clean = clean.split("```")[1]
        if clean.startswith("json"):
            clean = clean[4:]
    try:
        data = json.loads(clean.strip())
        return <OutputModel>(**data)
    except (json.JSONDecodeError, ValidationError) as e:
        raise AgentParseError(f"Parse failed: {e}\nRaw: {raw}")

## System Prompt Rules
- Always instruct: return ONLY valid JSON, no explanation, no markdown
- Always show the expected JSON structure in the prompt
- Be explicit about null rules if fields are optional
- Keep prompt focused — one clear instruction set

## Prompt Build Pattern
def _build_prompt(self, input_text: str) -> list:
    return [
        SystemMessage(content="<your system prompt>"),
        HumanMessage(content=input_text)
    ]
