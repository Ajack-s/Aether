---

# Basic example of app/main.py

```python
from fastapi import FastAPI, WebSocket
from app.agent import AetherAgent
from app.multimodal import MultimodalProcessor
from app.memory import FeedbackMemory
from app.agent_node import AgentNode
from pydantic import BaseModel
from datetime import datetime
import uuid

app = FastAPI()

class AgentRequest(BaseModel):
    input: dict
    mode: str = "standard"
    session_id: str = None

@app.post("/api/agent/execute")
async def execute_agent_task(request: AgentRequest):
    agent = AetherAgent(session_id=request.session_id)
    result = await agent.run(request.input)
    return {
        "session_id": agent.session_id,
        "result": result,
        "timestamp": datetime.now().isoformat()
    }

@app.websocket("/ws/agent/stream")
async def websocket_execution(websocket: WebSocket):
    await websocket.accept()
    agent = AetherAgent()
    while True:
        data = await websocket.receive_json()
        async for chunk in agent.stream_execution(data['input']):
            await websocket.send_json(chunk)
