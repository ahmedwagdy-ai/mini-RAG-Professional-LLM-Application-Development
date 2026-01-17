# 📑 mini-RAG | Tutorial 06 – Nested Routes & Environment Values 

## 1. Purpose

This Tutorial focuses on two production fundamentals:

* **Organizing routes using APIRouter (nested / modular routes)**
* **Managing configuration using environment variables (.env)**

The goal is scalability, cleanliness, and production readiness.

---

## 2. Why main.py Must Stay Small

Putting all routes in `main.py` causes:

* Poor readability
* Hard scaling
* Difficult maintenance

**Best practice:** `main.py` only creates the app, loads config, and registers routers.

---

## 3. Modular Routes with APIRouter

Create a routes package:

```
routes/
  ├── __init__.py
  └── base.py
```

`__init__.py` marks the folder as a Python package.

Inside `base.py`, use `APIRouter` instead of `FastAPI`:

```python
from fastapi import APIRouter
base_router = APIRouter()
```

APIRouter groups related endpoints and is mounted into the main app.

---

## 4. Base & Health Routes

Base routes handle lightweight endpoints such as root or health checks.

Example:

```python
@base_router.get("/")
async def root():
    return {"status": "ok"}
```

Health endpoints should be fast, simple, and dependency-free.

---

## 5. Registering Routers in main.py

Routes are inactive until registered:

```python
app.include_router(base_router, prefix="/api/v1", tags=["api-v1"])
```

Result:

* `/` → not exposed
* `/api/v1/` → active

This enables **API versioning** and clean Swagger grouping.

---

## 6. Running & Verifying

Run the server:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 5000
```

Test using browser, Swagger (`/docs`), or Postman.

---

## 7. Why Environment Variables Matter

Hardcoding config is unsafe and unscalable.

Instead, use a `.env` file:

```
APP_NAME=mini-rag
APP_VERSION=v0.1.0
```

This separates **code** from **configuration**.

---

## 8. Loading .env Correctly

Use `python-dotenv`:

```python
from dotenv import load_dotenv
load_dotenv(".env")
```

Rules:

* Call once
* Call early (before routers)

This injects values into `os.environ`.

---

## 9. Accessing Env Values Anywhere

Read values using:

```python
os.getenv("APP_NAME", "mini-rag")
```

Defaults prevent crashes if variables are missing.

Example in `base.py`:

```python
@base_router.get("/")
async def root():
    return {
        "app_name": os.getenv("APP_NAME"),
        "app_version": os.getenv("APP_VERSION"),
    }
```

---

## 10. Async-First Mindset

FastAPI is async-native.

Using `async def`:

* Improves concurrency
* Avoids blocking
* Prepares the app for future I/O

Even without `await` today, it matters later.

## Understanding Async and Await in FastAPI

In traditional synchronous programming, your code runs line-by-line. If one line takes a long time (like a database query), the entire server "blocks" and waits. **Async** allows the server to handle other tasks while waiting for I/O operations to finish.

## 1. Key Concepts

* **`async def`**: Defines a function as a "coroutine." It tells Python that this function can be paused to let other tasks run.
* **`await`**: Used inside an async function. It tells the Python interpreter: *"Pause here, do other work, and come back when this specific task is finished."*



## 2. The Analogy (The Waiter)

* **Synchronous (`def`)**: A waiter takes an order to the kitchen and **stands still** at the window waiting for the food. He cannot serve any other tables until that plate is ready.
* **Asynchronous (`async def`)**: A waiter takes an order to the kitchen, then immediately goes to **serve other tables** while the food is cooking. When the kitchen bells ring (`await` finishes), he goes back to pick up the plate.


## 3. Practical Example in FastAPI

FastAPI is designed to handle thousands of concurrent connections using this non-blocking approach.

```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/slow-data")
async def get_slow_data():
    # Simulate a network call or DB query
    await asyncio.sleep(5) 
    return {"data": "Results after 5 seconds"}

@app.get("/fast-data")
async def get_fast_data():
    return {"data": "I respond instantly!"}
```
Choosing the right function definition is crucial for FastAPI performance.

| Use Case | Recommended Approach | Reason |
| :--- | :--- | :--- |
| **I/O Bound** (Database queries, API calls, Reading files) | **Use `async def` + `await`** | The CPU stays idle while waiting for data; async lets it handle other requests during this gap. |
| **CPU Bound** (Image processing, Data crunching, Heavy Math) | **Use standard `def`** | These tasks require constant CPU power. FastAPI will run these in a separate **Thread Pool** to avoid freezing the main event loop. |
| **Standard Libraries** (e.g., `requests`, `boto3`) | **Use standard `def`** | These libraries are "blocking" and do not support the `await` keyword. Using `async` with them would still block the server. |


1. If you are using a modern library that supports `asyncio` (like `httpx` or `Motor` for MongoDB), use **`async def`**.
2. If you are unsure or using older libraries, use **`def`**. FastAPI is smart enough to handle standard `def` functions efficiently using threads.

---
---

# 📑 mini-RAG | Tutorial 07 – Uploading a File 


## 1. What This Tutorial Adds

This tutorial builds the **first real production endpoint**: file upload, while introducing core backend patterns:

* Scalable project structure (`src/`)
* MVC-style separation (Routes / Controllers / Models)
* Typed configuration with **Pydantic Settings**
* Dependency Injection (`Depends`)
* Secure file validation
* Async file saving with chunking

Goal: a **long-living, production-ready FastAPI app**.

---

## 2. Project Structure Upgrade (src/ Pattern)

Move all application code into `src/`:

```
root/
 ├── src/
 │    ├── main.py
 │    ├── routes/
 │    ├── controllers/
 │    ├── models/
 │    ├── helpers/
 │    └── assets/
 └── README.md
```

**Why?**

* Clear separation of code vs tooling/docs
* Easier Docker & deployment
* Industry‑standard layout

Update run command:

```
uvicorn src.main:app
```

---

## 3. Separation of Concerns (MVC for APIs)

FastAPI routes should stay thin.

Use MVC‑style separation:

* **Routes**: HTTP layer only
* **Controllers**: business logic
* **Models**: validation & schemas (Pydantic)
* **Helpers**: shared utilities (config, file utils)

Benefits: testability, reuse, maintainability.

---

## 4. Typed Configuration with Pydantic Settings

Replace raw `dotenv` with **pydantic-settings**.

`helpers/config.py`:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class AppSettings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env")
    app_name: str
    app_version: str

settings = AppSettings()
```

**Why this matters:**

* Auto‑load `.env`
* Type validation
* Clear startup errors
* IDE autocomplete



**Pydantic Settings** manages your application's configuration by automatically loading Environment Variables or `.env` files into Python objects.

### Key Importance:
* **Validation:** It automatically checks data types (e.g., ensuring a `PORT` is an `int`) and crashes early if settings are missing or wrong.
* **Security:** Keeps sensitive credentials (API keys, passwords) out of your source code by loading them from external environments.
* **Ease of Use:** Provides IDE auto-completion and a single "source of truth" for all your app's configurations.

---

## 5. Dependency Injection with Depends

Expose settings safely using FastAPI DI:

```python
def get_settings() -> AppSettings:
    return settings
```

Usage:

```python
async def root(settings: AppSettings = Depends(get_settings)):
    return {"app": settings.app_name}
```

**Advantages:**

* Centralized initialization
* Easy mocking in tests
* Clean function signatures

---

## 6. Upload Route Design

Create a dedicated data router:

```python
data_router = APIRouter(
    prefix="/api/v1/data",
    tags=["data"]
)
```

Endpoint:

```python
@data_router.post("/upload/{project_id}")
async def upload_data(
    project_id: int,
    file: UploadFile = File(...),
    settings: AppSettings = Depends(get_settings)
)
```

* `project_id`: enables multi‑tenant isolation
* `UploadFile`: async‑friendly file streaming


**Multi-tenancy** is a software architecture where a single instance of an application serves multiple customers (called "tenants"), while keeping their data strictly isolated from each other.

### Key Importance:
* **Cost Efficiency:** Reduces expenses by sharing hardware, database, and infrastructure resources among many users.
* **Simplified Maintenance:** Updates or bug fixes applied to the single application instance automatically reach all tenants at once.
* **Scalability:** Makes it easier to manage thousands of users (like SaaS platforms such as Slack or Shopify) from a central codebase.

---

## 7. Security First: Never Trust the Client

All uploads **must be validated**.

In `.env`:

```
FILE_ALLOWED_EXTENSIONS=txt,pdf
FILE_MAX_SIZE=10
```

Model validation:

```python
from enum import Enum

class AllowedMimeType(str, Enum):
    TEXT = "text/plain"
    PDF = "application/pdf"
```

Checks performed:

* File extension
* File size (MB limit)
* MIME type

Enums guarantee **type‑safe allowed values**.

---

## 8. Controllers Pattern

Base controller:

```python
class BaseController:
    def __init__(self):
        self.settings = settings
```

Data controller:

```python
class DataController(BaseController):
    async def fit_upload_file(self, file, project_id):
        # validation + save
        return True
```

**Why controllers?**

* Shared config
* Reusable logic
* Cleaner routes

---

## 9. Async File Saving with aiofiles

Use non‑blocking I/O:

```python
async with aiofiles.open(path, "wb") as f:
    while chunk := await file.read(1024 * 1024):
        await f.write(chunk)
```

**Chunked upload benefits:**

* Low memory usage
* Supports large files
* Production‑safe

Directories are created dynamically per `project_id`.

---

## 10. Running & Testing

Run:

```
uvicorn main:app --reload --host 0.0.0.0 --port 5000
```

Test with Postman:

* Method: POST
* URL: `/api/v1/data/upload/123`
* Body: `form-data`
* Key: `file`

---



## Final Outcome

You now have a **secure, async, validated upload endpoint** built with:

* Clean architecture
* Typed configuration
* Dependency injection
* Production‑grade file handling

This is a **real backend foundation** for any RAG system.

---

# 📑 mini-RAG | 08 | File Processing


## Tutorial Overview

This Tutorial extends the upload workflow by introducing **file processing**: loading raw files, extracting text, and splitting it into chunks suitable for embeddings and retrieval.

**Technologies covered**:

* Pydantic Schemas (request validation)
* LangChain File Loaders
* Recursive Text Chunking

Repository (branch `tut-005`): mini-rag

---

## Review: Upload Improvements

Previously, files were uploaded to:

```
assets/uploads/{project_id}/
```

### Unique Filename Enhancement

`generate_unique_filename` was updated to return:

```
(new_filename, file_id)
```

* `new_filename`: actual stored filename
* `file_id`: unique identifier used for downstream processing

Route example:

```python
new_filename, file_id = controller.generate_unique_filename(file)
return {"success": True, "file_id": file_id}
```

This enables deterministic file tracking without exposing real filenames.

---

## New Feature: File Processing Endpoint

A new branch is created:

```
git checkout -b tutorial-005
```

A new endpoint is introduced:

```
POST /api/v1/data/process/{project_id}
```

Purpose:

* Take a `file_id`
* Load file contents
* Split content into chunks

---

## Request Validation with Pydantic

### ProcessRequest Schema

```python
from pydantic import BaseModel
from typing import Optional

class ProcessRequest(BaseModel):
    file_id: str
    chunk_size: Optional[int] = 1000
    overlap_size: Optional[int] = 20
    reset: Optional[bool] = False
```

### Why this matters

* Rejects invalid input automatically (422 errors)
* Provides defaults for optional parameters
* FastAPI auto-parses JSON into typed objects

Usage in route:

```python
async def process_file(
    project_id: int,
    request: ProcessRequest,
):
    file_id = request.file_id
    chunk_size = request.chunk_size
```

---

## Process Controller Design

### ProcessController

```python
class ProcessController(BaseController):
    def __init__(self, project_id: int):
        super().__init__()
        self.project_id = project_id
```

Responsibilities:

* Resolve file paths
* Select correct loader
* Run chunking logic

### File Extension Detection

```python
def get_file_extension(self, file_id: str) -> str:
    return os.path.splitext(file_id)[1]
```

---

## LangChain File Loaders

### Loader Selection

```python
def get_file_loader(self, file_id: str):
    ext = self.get_file_extension(file_id)
    file_path = os.path.join(self.project_path, file_id)

    if ext == ".txt":
        return TextLoader(file_path)
    elif ext == ".pdf":
        return PyMuPDFLoader(file_path)
    raise ValueError("Unsupported extension")
```

### Loader Output

```python
loader.load() -> List[Document]
```

Each `Document` contains:

* `page_content`
* `metadata` (page number, source, etc.)

---

## Text Chunking with RecursiveCharacterTextSplitter

### Content Extraction

```python
file_docs = loader.load()
```

### Chunking Logic

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=chunk_size,
    chunk_overlap=overlap_size,
    length_function=len,
)

texts = [doc.page_content for doc in file_docs]
metadatas = [doc.metadata for doc in file_docs]

chunks = splitter.create_documents(texts, metadatas=metadatas)
```

### Why Chunking is Required

* LLM context limits
* Efficient retrieval (RAG)
* Lower token cost

### Recursive Strategy

* Splits on paragraphs/sentences when possible
* Falls back to character boundaries
* Uses overlap to preserve context continuity

---

## Final Route Logic

```python
controller = ProcessController(project_id)
file_content = controller.get_file_content(file_id)
chunks = controller.process_file_content(
    file_content,
    file_id,
    chunk_size,
    overlap_size,
)

if not chunks:
    raise HTTPException(400, detail="No content")

return {
    "success": True,
    "chunks_count": len(chunks),
    "signal": "chunks_generated",
}
```

---

## Testing

### Postman Request

```
POST /api/v1/data/process/123
```

Body:

```json
{
  "file_id": "abc123.txt",
  "chunk_size": 400,
  "overlap_size": 20
}
```

Expected Result:

* Chunk count
* Metadata preserved per chunk

---

## Summary

This Tutorial establishes the **processing backbone** for RAG:

* Strict request validation
* Loader abstraction per file type
* Deterministic chunk generation

The output is now ready for:

* Embedding
* Vector storage
* Retrieval pipelines

---

## Verbally Mentioned Details (Appendix)

* `file_id` is treated as a logical identifier; a database will later map it to physical paths.
* Chunk size should be tuned based on embedding model token limits, not arbitrarily.
* PDF loaders may return empty pages; filtering may be required in production.
* Recursive splitting prioritizes semantic boundaries over fixed character cuts.
* Processing is synchronous here but designed to move to background jobs later.
* This step intentionally avoids embeddings to keep concerns separated.

---

# 📑 mini-RAG | 09 | Docker – MongoDB – Motor

## 1. Episode Context – From Chunks to Persistence

Previously, the system processed files and generated chunks **in memory only**.

```
Stateless → Data lost on restart
Stateful  → Data persisted across restarts
```

**Design shift**:

* Move from transient chunks → persistent storage
* Enable reuse, querying, and embeddings later

**Chosen solution**: **MongoDB (Document-based NoSQL)**

Why?

* Flexible schema
* Ideal for semi-structured data (chunks + metadata)
* Horizontal scalability

---

## 2. MongoDB – Deep NoSQL Theory

### Relational vs Document Databases

```
Relational (SQL)
├── Fixed schema (tables)
├── Strong relations (joins)
├── ACID transactions
└── Vertical scaling

Document (NoSQL)
├── Flexible schema (JSON-like)
├── Embedded documents
├── Horizontal scaling (sharding)
└── Schema-on-read
```

### MongoDB vs CouchDB

| Feature     | MongoDB              | CouchDB            |
| ----------- | -------------------- | ------------------ |
| Querying    | Mongo Query Language | MapReduce Views    |
| Replication | Primary–Secondary    | Multi-master       |
| Typical Use | Real-time apps       | Offline-first apps |

**Why MongoDB for RAG?**

* Chunk structure may evolve
* Metadata varies by file type
* High read throughput for retrieval

---

## 3. Docker – Containerization Foundation

### Why Docker?

```
Container = App + Dependencies + Runtime
Image     = Blueprint
```

Benefits:

* Environment consistency
* Zero local Mongo installation issues
* Easy cloud deployment later

### Dockerfile vs Docker Compose

```
Dockerfile → Build one service
Compose    → Orchestrate multiple services
```

We use **Docker Compose** to manage MongoDB.

---

## 4. docker-compose.yml – MongoDB Service

```yaml
services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    ports:
      - "27017:27017"
    volumes:
      - ./mongodb:/data/db
    networks:
      - backend
    restart: always
```

### Section-by-Section Theory

| Section  | Purpose           | Why Important                |
| -------- | ----------------- | ---------------------------- |
| image    | MongoDB version   | Official + secure            |
| ports    | Host access       | Local tools connect          |
| volumes  | Persist data      | Container delete ≠ data loss |
| networks | Service discovery | App connects by name         |
| restart  | Resilience        | Auto recovery                |

---

## 5. Running Docker Compose

```bash
cd docker/
docker compose up
```

Useful commands:

```
docker compose up -d   # Background
docker compose ps     # Status
docker compose down   # Stop
```

### Persistence Guarantee

* Data stored in `./mongodb/`
* Survives restarts and machine changes

---

## 6. MongoDB Client Tools

You can inspect data using:

* **MongoDB Compass**
* **Studio 3T**

Connection:

```
Host: localhost
Port: 27017
```

Databases created on first insert:

```
admin, config, local
mini-rag (user database)
```

---

## 7. Motor – Async MongoDB Driver

### Why Motor?

```
FastAPI = ASGI (async)
PyMongo = blocking
Motor   = non-blocking
```

### Installation

```bash
pip install motor
```

### Connection Lifecycle

```python
from motor.motor_asyncio import AsyncIOMotorClient

async def startup_db_client():
    client = AsyncIOMotorClient(settings.mongodb_url)
    app.state.mongodb_client = client
    app.state.db = client[settings.mongodb_database]

async def shutdown_db_client():
    app.state.mongodb_client.close()
```

### App Events

```python
@app.on_event("startup")
async def startup():
    await startup_db_client()

@app.on_event("shutdown")
async def shutdown():
    await shutdown_db_client()
```

**Why app.state?**

* Global DB access
* Single connection pool

---

## 8. Pydantic Schemas for MongoDB

### Project Schema

```python
class Project(BaseModel):
    id: Optional[ObjectId] = None
    project_id: str = Field(min_length=1)

    class Config:
        arbitrary_types_allowed = True
        json_encoders = {ObjectId: str}
```

### Chunk Schema

```python
class Chunk(BaseModel):
    id: Optional[ObjectId] = None
    chunk_text: str
    metadata: dict
    chunk_order: int = Field(gt=0)
    project_id: ObjectId
```

### Validation Power

| Feature           | Purpose            |
| ----------------- | ------------------ |
| Field constraints | Data integrity     |
| ObjectId encoding | JSON compatibility |
| Custom validators | Business rules     |

---

## 9. Collections Design Philosophy

```
projects → project metadata
chunks   → text chunks + metadata
```

MongoDB principle:

```
Insert data freely
Validate in application layer (Pydantic)
```

Example document:

```json
{
  "chunk_text": "Sample text",
  "metadata": {"page": 3},
  "chunk_order": 1,
  "project_id": "ObjectId(...)"
}
```

---

## 10. Architecture Summary

```
Docker Compose → MongoDB
FastAPI → Motor (async)
Pydantic → Validation
app.state → Global DB access
Lifecycle events → Safe connect/disconnect
```

