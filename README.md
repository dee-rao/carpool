# carpool
Carpool iOS and Android app with a FastAPI backend.

## Backend API

The backend is a **Hello World** FastAPI application that serves as the starting point for the Carpool API.

### Requirements

- Python 3.11+ (Python 3.12 also works)
- `pip` / `venv`

### Setup

```bash
# Create and activate a virtual environment
python3 -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate

# Install runtime dependencies
pip install -r requirements.txt

# Install development dependencies (includes test tools)
pip install -r requirements-dev.txt
```

### Run the API

```bash
uvicorn api.main:app --reload
```

The API will be available at <http://localhost:8000>.

| Endpoint  | Description          |
|-----------|----------------------|
| `GET /`   | Hello World response |
| `GET /health` | Health check     |
| `GET /docs` | Interactive Swagger UI |

### Run Tests

```bash
pytest tests/ -v
```

---

## ASGI vs WSGI

### WSGI — Web Server Gateway Interface

**WSGI** (PEP 3333) is the traditional Python web server standard. It was designed for **synchronous** request handling:

- Each request is handled one at a time in a single thread (or process).
- The server calls your application as a simple callable: `app(environ, start_response)`.
- Works well for CPU-bound or traditional web applications (Django, Flask).
- **Cannot** natively handle long-lived connections such as WebSockets or Server-Sent Events.

### ASGI — Asynchronous Server Gateway Interface

**ASGI** is the modern successor to WSGI. It supports **asynchronous** request handling:

- Built around Python's `async`/`await` syntax and the `asyncio` event loop.
- Handles HTTP, WebSocket, and other protocols in a single server process.
- The server calls your application as an async callable: `await app(scope, receive, send)`.
- Enables high concurrency without threading, making it ideal for I/O-bound workloads (DB calls, external APIs).
- Used by **FastAPI**, Starlette, Django Channels, and Quart.

### Key Differences

| Feature              | WSGI                    | ASGI                        |
|----------------------|-------------------------|-----------------------------|
| Execution model      | Synchronous             | Asynchronous (`async/await`) |
| WebSocket support    | No                      | Yes                         |
| Concurrency          | Thread/process-based    | Single-threaded event loop  |
| Frameworks           | Django, Flask           | FastAPI, Starlette, Django Channels |
| Server examples      | Gunicorn, uWSGI         | Uvicorn, Hypercorn, Daphne  |

### Why FastAPI uses ASGI

FastAPI is built on **Starlette** (an ASGI framework) and uses **Uvicorn** as its ASGI server. This means:

1. Route handlers can be `async def` functions, allowing non-blocking I/O.
2. WebSocket endpoints are first-class citizens.
3. A single Uvicorn worker can serve thousands of concurrent connections.

For the Carpool app, ASGI is the right choice because features like real-time ride tracking and notifications benefit from WebSocket and long-polling support.
