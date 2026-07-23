# Skill: FastAPI Layer Pattern

---
name: fastapi-layer-pattern
description: Use this skill when building any FastAPI assignment. Defines layer responsibilities, strict rules, route pattern, service pattern, app factory, config pattern and build order.
---

## Layer Rules

routers/
- Route definitions only
- Inject service via Depends()
- No business logic, no agent calls
- HTTP only: 422 bad input, 500 agent error

services/
- Business logic only
- Calls agent, returns response model
- Input validation before agent call
- No FastAPI imports here

agents/
- One job: call Ollama, return typed Pydantic model
- No FastAPI imports
- No service logic
- Raises AgentParseError on parse failure

models/
- Pydantic schemas only
- No logic, no cross-layer imports

## Strict Rules
- No logic in models
- No schemas in agents
- No agent calls in routers
- No FastAPI imports in agents
- Always use response_model on every route
- Always use Depends() for service injection

## App Factory
def create_app() -> FastAPI:
    app = FastAPI(title="...", version="0.1.0")
    app.include_router(router)
    return app

app = create_app()

## Service Pattern
class <Domain>Service:
    def __init__(self, agent: <Domain>Agent):
        self.agent = agent

    async def process(self, request: <Request>) -> <Response>:
        try:
            result = await self.agent.extract(request.field)
            return <Response>(success=True, data=result)
        except AgentParseError as e:
            return <Response>(success=False, error=str(e))
        except Exception as e:
            return <Response>(success=False, error=f"Unexpected: {str(e)}")

def get_<domain>_service() -> <Domain>Service:
    agent = <Domain>Agent(model=settings.OLLAMA_MODEL)
    return <Domain>Service(agent=agent)

## Config Pattern
class Settings(BaseSettings):
    OLLAMA_MODEL: str = "llama3.1:8b"
    OLLAMA_BASE_URL: str = "http://localhost:11434"
    APP_ENV: str = "development"
    model_config = SettingsConfigDict(env_file=".env")

settings = Settings()
