# agent2agent-client
This repository is a wrapper for building google a2a client, without explicitly copying the code to the native development codebase. This repository support - a2a client core, - a2a client utils,  - a2a client types under the package called commons. Infact this package is direct lift and shift from google's repository.

# How to install
### clone this repository
* option-1
    - `git clone https://github.com/pavanjava/agent2agent-client.git`
    - `pip install -e .`
* option-2

    - `pip install agent2agent-client` from pypi.org

# How to use this?
```python
# streaming_echo_client.py (Modifications based on echo_client.py)
import asyncio
import logging
from uuid import uuid4

from common_client.client import A2AClient
from common_client.types import Message, TextPart, TaskStatusUpdateEvent, TaskArtifactUpdateEvent

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

SEARCH_SERVER_URL = "http://localhost:8001/search-a2a"


async def main():
    client = A2AClient(url=SEARCH_SERVER_URL)

    task_id = f"search-task-{uuid4().hex}"
    user_text = "Impact of AI on on Supply Chain and Shipping Lines"

    user_message = Message(role="user", parts=[TextPart(text=user_text)])

    send_params = {
        "id": task_id,
        "message": user_message,
    }

    try:
        logger.info(f"Sending task {task_id} to {SEARCH_SERVER_URL}...")

        # Use the client's send_task_streaming method
        response = await client.send_task(payload=send_params, timeout=300)
        print(response)
        if response.error:
            # Errors might be sent as part of the stream in some implementations
            logger.error(f"Received error in stream for task {task_id}: {response.error.message}")

        elif response.result:
            event = response.result
            if isinstance(event, TaskStatusUpdateEvent):
                logger.info(f"Task {task_id} Status Update: {event.status.state}")
                if event.status.message and event.status.message.parts:
                    part = event.status.message.parts[0]
                    if isinstance(part, TextPart):
                        logger.info(f"  Agent Message: {part.text}")
                if event.final:
                    logger.info(f"Task {task_id} reached final state.")

            elif isinstance(event, TaskArtifactUpdateEvent):
                logger.info(f"Task {task_id} Artifact Update: {event.artifact.name}")
                # Process artifact parts...
            else:
                logger.warning(f"Received unknown event type : {type(event)}")
        else:
            logger.error(f"Received unexpected empty response for task {task_id}")


    except Exception as e:
        logger.error(f"An error occurred during task communication: {e}")


if __name__ == "__main__":
    asyncio.run(main())

```
