# FastAPI Interview Questions & Answers

## Table of Contents

1. [What makes FastAPI "fast"?](#q1-what-makes-fastapi-fast)
2. [Sync vs Async Endpoints (`def` vs `async def`)](#q2-what-is-the-difference-between-def-and-async-def-endpoints)
3. [Pydantic Models](#q3-what-is-pydantic-and-how-does-fastapi-use-it)
4. [Path vs Query Parameters](#q4-what-is-the-difference-between-path-and-query-parameters)
5. [Request Body Handling](#q5-how-do-you-handle-request-bodies)
6. [Response Models & Output Filtering](#q6-what-is-a-response-model-and-why-use-it)
7. [Dependency Injection](#q7-how-does-dependency-injection-work-in-fastapi)
8. [FastAPI vs Flask](#q8-how-does-fastapi-compare-to-flask)
9. [Middleware](#q9-how-do-you-add-middleware-in-fastapi)
10. [Error Handling](#q10-how-do-you-handle-errors-globally)
11. [Form Data](#q11-how-do-you-handle-form-data)
12. [File Uploads](#q12-how-do-you-handle-file-uploads)
13. [OAuth2 & JWT Security](#q13-how-do-you-implement-authentication)
14. [Background Tasks](#q14-what-are-background-tasks-in-fastapi)
15. [WebSockets](#q15-does-fastapi-support-websockets)
16. [APIRouter (Structuring Apps)](#q16-how-do-you-structure-large-applications)
17. [Testing with TestClient](#q17-how-do-you-test-fastapi-applications)
18. [Database Sessions (SQLAlchemy)](#q18-how-do-you-manage-database-sessions)
19. [OpenAPI/Swagger Customization](#q19-can-you-customize-the-swagger-ui)
20. [Input Validation](#q20-how-do-you-add-validation-constraints)
21. [Lifespan Events (Startup/Shutdown)](#q21-how-do-you-handle-startup-and-shutdown-events)
22. [Configuration Management](#q22-how-do-you-manage-settings-and-environment-variables)
23. [Deployment (Uvicorn vs Gunicorn)](#q23-how-should-you-deploy-fastapi-in-production)
24. [Circular Dependencies in Pydantic](#q24-how-do-you-handle-circular-references-in-models)
25. [Sub-Applications](#q25-can-you-mount-other-applications-inside-fastapi)
26. [Static Files](#q26-how-do-you-serve-static-files)
27. [CORS (Cross-Origin Resource Sharing)](#q27-how-do-you-enable-cors)
28. [Field vs Query vs Path](#q28-what-is-the-difference-between-field-query-and-path)
29. [Concurrency & Thread Pool](#q29-how-does-fastapi-handle-blocking-code)
30. [Optional Parameters](#q30-how-do-you-make-parameters-optional)

---

## Q1. What makes FastAPI "fast"?
Concept:
FastAPI is built on top of **Starlette** (for web routing) and **Pydantic** (for data validation).
It uses Python's `asyncio` library for asynchronous concurrency, making it comparable in speed to NodeJS and Go.
It leverages **ASGI** (Asynchronous Server Gateway Interface) standard.

Example:
```python
# Benchmarks show it is one of the fastest Python frameworks available.
# It compiles Pydantic models to optimized C code (via Cython in v2).
```

Analogy:
Old frameworks (Flask/Django) are like a single cashier line (WSGI). FastAPI is like a digital kiosk system where thousands of orders (requests) can be processed concurrently (ASGI).

⚡ One-liner to remember:
FastAPI = Starlette (Async Web) + Pydantic (Validation) + ASGI (Concurrency).

## Q2. What is the difference between `def` and `async def` endpoints?
Concept:
- **`async def`:** Use this if your code uses `await` (e.g., DB calls, API requests). It runs directly in the event loop.
- **`def`:** Use this for blocking code (e.g., standard file I/O, heavy CPU tasks). FastAPI runs these in a separate thread pool (external thread) to prevent blocking the event loop.

Example:
```python
# Runs in main event loop
@app.get("/")
async def read_root():
    res = await some_async_db_call()
    return res

# Runs in a separate thread pool
@app.get("/sync")
def read_sync():
    time.sleep(5) # Won't block the server!
    return {"message": "Done"}
```

Analogy:
- `async def`: A waiter taking an order and immediately going to the next table while the kitchen cooks.
- `def`: A waiter who has to cook the food themselves. FastAPI hires an extra helper (thread) to do it so the main waiter isn't stuck.

⚡ One-liner to remember:
`async def` for awaitable IO; `def` for blocking synchronous code (handled via thread pool).

## Q3. What is Pydantic and how does FastAPI use it?
Concept:
Pydantic is a data validation library using Python type hints.
FastAPI uses it to:
1. Validate request bodies.
2. Convert input data (JSON) to Python objects.
3. Generate JSON Schemas for OpenAPI documentation.

Example:
```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    is_active: bool = True # Default value

@app.post("/users/")
def create_user(user: User):
    # 'user' is already validated and is a User object
    return user
```

Analogy:
A rigorous bouncer at a club. You give him your ID (Data). He checks the birthdate, name, and validity (Validation). If valid, he lets you in as a verified person (Python Object). If not, he rejects you immediately (422 Error).

⚡ One-liner to remember:
Pydantic enforces types at runtime and parses JSON into Python objects automatically.

## Q4. What is the difference between Path and Query parameters?
Concept:
- **Path Parameters:** Part of the URL path itself. Used to identify a specific resource. Mandatory.
- **Query Parameters:** Key-value pairs after the `?` in the URL. Used for filtering/sorting. Optional by default unless specified otherwise.

Example:
```python
# Path Param: /items/5
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"id": item_id}

# Query Param: /items/?skip=0&limit=10
@app.get("/items/")
def read_items(skip: int = 0, limit: int = 10):
    return {"skip": skip, "limit": limit}
```

Analogy:
- Path: The street address (e.g., 123 Main St). It points to the building.
- Query: Instructions for the delivery driver (e.g., ?ring_bell=true). It modifies how the delivery happens.

⚡ One-liner to remember:
Path identifies the resource; Query modifies the result view.

## Q5. How do you handle request bodies?
Concept:
Define a Pydantic model and declare it as a parameter in your path operation function. FastAPI treats parameters that are NOT path parameters and NOT singular types (int, str) as Request Bodies.

Example:
```python
class Item(BaseModel):
    name: str
    price: float

@app.post("/items/")
def create_item(item: Item):
    return {"name": item.name, "price_with_tax": item.price * 1.2}
```

Analogy:
Filling out a form at the doctor's office. The form structure (Pydantic Model) dictates what info you must provide. You hand the filled form (Request Body) to the receptionist.

⚡ One-liner to remember:
Create a Pydantic model and pass it as a function argument.

## Q6. What is a Response Model and why use it?
Concept:
`response_model` defines the schema of the data returned to the client.
It allows you to:
1. Filter out sensitive data (e.g., passwords).
2. Validate the output data.
3. Generate correct API documentation.

Example:
```python
class UserIn(BaseModel):
    username: str
    password: str

class UserOut(BaseModel):
    username: str
    # Password is excluded here

@app.post("/user/", response_model=UserOut)
def create_user(user: UserIn):
    return user # FastAPI automatically removes 'password' from response
```

Analogy:
A public relations officer. The company has internal secrets (Database Model), but the PR officer (Response Model) only releases the approved public statement to the press.

⚡ One-liner to remember:
Use `response_model` to sanitize and shape your API output automatically.

## Q7. How does Dependency Injection work in FastAPI?
Concept:
FastAPI has a built-in DI system using `Depends`. It allows you to declare reusable logic (like DB sessions, auth, query params) that is executed before the route handler.

Example:
```python
from fastapi import Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users/")
def read_users(db = Depends(get_db)):
    # db is injected here automatically
    return db.query(User).all()
```

Analogy:
Ordering a combo meal. You don't go to the back to fry the fries and pour the soda. You just say "I want a combo" (`Depends`), and the staff prepares the dependencies and hands you the complete tray.

⚡ One-liner to remember:
`Depends` lets you declare shared logic required by your endpoints.

## Q8. How does FastAPI compare to Flask?
Concept:
- **Flask:** Micro-framework, sync by default (WSGI), older, large ecosystem, no built-in data validation.
- **FastAPI:** Modern, async native (ASGI), built-in validation (Pydantic), auto-documentation (Swagger), faster performance.

Example:
- Flask: Need `Marshmallow` for validation, `Flasgger` for docs.
- FastAPI: Comes with everything out of the box.

Analogy:
- Flask: A classic manual transmission car. Great, reliable, you control every gear shift, but you have to do more work.
- FastAPI: A modern Tesla. Autopilot (Async), built-in navigation (Docs), and safety sensors (Validation).

⚡ One-liner to remember:
FastAPI is Flask's modern, faster, type-safe cousin.

## Q9. How do you add middleware in FastAPI?
Concept:
Use the `@app.middleware("http")` decorator or `app.add_middleware()`. Middleware runs before the request reaches the endpoint and after the response is generated.

Example:
```python
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

Analogy:
Security check at an airport. You go through security (Middleware) before boarding the plane (Endpoint). On the way back (Response), you go through customs (Middleware again).

⚡ One-liner to remember:
Wrap the request/response cycle with custom logic using `add_middleware`.

## Q10. How do you handle errors globally?
Concept:
Use exception handlers with `@app.exception_handler`. You can catch specific exceptions (like `HTTPException` or custom ones) and return a custom JSON response.

Example:
```python
from fastapi import Request, HTTPException
from fastapi.responses import JSONResponse

class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name

@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something wrong."},
    )
```

Analogy:
A complaint department. If a customer has a specific issue (Exception), they are directed to a specialist (Handler) who gives a standardized apology letter (JSON Response).

⚡ One-liner to remember:
Register custom handlers to catch exceptions and return polite JSON responses.

## Q11. How do you handle Form Data?
Concept:
Use `Form` from `fastapi`. Standard Pydantic models expect JSON body; for `application/x-www-form-urlencoded`, you must use `Form`.

Example:
```python
from fastapi import Form

@app.post("/login/")
def login(username: str = Form(), password: str = Form()):
    return {"username": username}
```

Analogy:
Mailing a letter. JSON is sending a box. Form Data is sending a postcard. The post office handles them differently.

⚡ One-liner to remember:
Use `Form()` parameter markers for standard HTML form submissions.

## Q12. How do you handle File Uploads?
Concept:
Use `File` and `UploadFile`.
- `File`: Reads the whole file into memory (bytes). Good for small files.
- `UploadFile`: Uses a "spooled" file (stored on disk if large). Exposes Python `file`-like interface. PREFERRED.

Example:
```python
from fastapi import UploadFile, File

@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile = File(...)):
    contents = await file.read()
    return {"filename": file.filename}
```

Analogy:
- `bytes`: Memorizing a whole book (hard for RAM).
- `UploadFile`: Reading a book page by page (efficient).

⚡ One-liner to remember:
Always use `UploadFile` for memory-efficient file handling.

## Q13. How do you implement authentication?
Concept:
FastAPI provides security utilities in `fastapi.security`. The most common flow is **OAuth2 with Password (and JWT)**.
Use `OAuth2PasswordBearer` to extract the token from the header.

Example:
```python
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/users/me")
def read_users_me(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

Analogy:
A VIP wristband. You get the wristband (Token) at the entrance (Login). Inside, you just show the wristband (`Depends(oauth2_scheme)`) to get into exclusive areas.

⚡ One-liner to remember:
Use `OAuth2PasswordBearer` + `Depends` to secure endpoints with tokens.

## Q14. What are Background Tasks in FastAPI?
Concept:
Allows you to run a function *after* returning the response to the user. Useful for emails, data processing, etc.

Example:
```python
from fastapi import BackgroundTasks

def write_log(message: str):
    with open("log.txt", "a") as log:
        log.write(message)

@app.post("/send-email/")
def send_email(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_log, f"Email sent to {email}")
    return {"message": "Email sent"} # Returns immediately
```

Analogy:
"I'll handle it." You tell the customer "Your order is confirmed" (Response), and then you turn around and start packing the box (Background Task) while the customer leaves happy.

⚡ One-liner to remember:
Execute code after the response is sent using `BackgroundTasks`.

## Q15. Does FastAPI support WebSockets?
Concept:
Yes, native support. Use the `WebSocket` class.

Example:
```python
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

Analogy:
A phone call. Unlike HTTP (sending letters back and forth), a WebSocket is an open line where both parties can talk instantly at any time.

⚡ One-liner to remember:
Native `WebSocket` support allows real-time, two-way communication.

## Q16. How do you structure large applications?
Concept:
Use `APIRouter`. Create separate files (modules) for different parts of your app (e.g., `users.py`, `items.py`) and "include" them in the main `app`.

Example:
```python
# routers/users.py
router = APIRouter()
@router.get("/users/") ...

# main.py
from routers import users
app.include_router(users.router)
```

Analogy:
A shopping mall. The `app` is the mall building. Each `APIRouter` is a specific store (Gap, Apple, Nike). They are independent but housed under one roof.

⚡ One-liner to remember:
Break your app into modules with `APIRouter` and mount them in main.

## Q17. How do you test FastAPI applications?
Concept:
Use `TestClient` (based on `httpx` and `Starlette`). It allows you to make requests to your app without running a live server.

Example:
```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```

Analogy:
A flight simulator. You test the plane (App) in a safe environment (TestClient) before flying real passengers (Uvicorn server).

⚡ One-liner to remember:
`TestClient` lets you run pytest against your API directly.

## Q18. How do you manage database sessions?
Concept:
Use **Dependency Injection** with `yield`. This ensures a session is created for each request and closed *after* the request is finished (even if errors occur).

Example:
```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close() # Always runs
```

Analogy:
Borrowing a book from the library. The librarian (`get_db`) hands it to you (`yield`). When you leave (Request ends), you must return it (`finally: close`), or the library runs out of books.

⚡ One-liner to remember:
Use `yield` in a dependency to open a session and close it in the `finally` block.

## Q19. Can you customize the Swagger UI?
Concept:
Yes. You can pass metadata to the `FastAPI` constructor (title, description, version) and configure tags.

Example:
```python
app = FastAPI(
    title="My Super API",
    description="This is a custom API",
    version="2.5.0",
    docs_url="/documentation" # Change URL
)
```

Analogy:
Customizing your shop's storefront. You change the sign (Title), the window display (Description), and even move the entrance door (docs_url).

⚡ One-liner to remember:
FastAPI allows full customization of the auto-generated docs via constructor arguments.

## Q20. How do you add validation constraints?
Concept:
Use `Query`, `Path`, and `Body` fields with validation arguments like `min_length`, `max_length`, `regex`, `gt` (greater than).

Example:
```python
from fastapi import Query

@app.get("/items/")
def read_items(q: str = Query(None, min_length=3, max_length=50, regex="^fixedquery$")):
    return {"q": q}
```

Analogy:
A form that turns red if your password is too short. The validation rules enforce the constraints before processing.

⚡ One-liner to remember:
Add constraints like `min_length` inside `Query()` or `Path()` functions.

## Q21. How do you handle startup and shutdown events?
Concept:
Use the `lifespan` context manager (recommended in newer versions) or `@app.on_event("startup")` (deprecated).

Example:
```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup logic (Connect DB)
    print("Starting up...")
    yield
    # Shutdown logic (Close DB)
    print("Shutting down...")

app = FastAPI(lifespan=lifespan)
```

Analogy:
Opening and closing a store. Turn on lights and unlock doors (Startup). At night, lock up and turn off lights (Shutdown).

⚡ One-liner to remember:
Use the `lifespan` context manager to manage resources across the app lifecycle.

## Q22. How do you manage settings and environment variables?
Concept:
Use Pydantic's `BaseSettings`. It reads from environment variables automatically.

Example:
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    app_name: str = "My API"
    admin_email: str
    
    class Config:
        env_file = ".env"

settings = Settings()
# settings.admin_email will come from env var ADMIN_EMAIL
```

Analogy:
Reading the user manual. The settings file defines the configuration, but if you have specific instructions written on a sticky note (Env Var), those take precedence.

⚡ One-liner to remember:
`BaseSettings` automagically loads environment variables into a typed object.

## Q23. How should you deploy FastAPI in production?
Concept:
Run **Uvicorn** (ASGI worker) managed by **Gunicorn** (Process manager). Gunicorn manages multiple Uvicorn worker processes to utilize multiple CPU cores.

Example:
```bash
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

Analogy:
Uvicorn is a super-fast worker bee. Gunicorn is the queen bee who manages 4 worker bees to get 4x the work done.

⚡ One-liner to remember:
Gunicorn as a process manager, Uvicorn as the worker class.

## Q24. How do you handle circular references in models?
Concept:
Use `typing.ForwardRef` or strings for type hints, and `model.update_forward_refs()` (Pydantic v1) or simply strings in Pydantic v2.

Example:
```python
class User(BaseModel):
    items: List["Item"] = []

class Item(BaseModel):
    owner: User
```

Analogy:
Chicken and Egg problem. You tell Python "Trust me, the 'Item' definition is coming later" using a string or ForwardRef.

⚡ One-liner to remember:
Use string type hints `List["Item"]` to reference models defined later.

## Q25. Can you mount other applications inside FastAPI?
Concept:
Yes. You can mount other WSGI (Flask) or ASGI (FastAPI/Django) apps under a specific path using `.mount()`.

Example:
```python
from fastapi import FastAPI
from flask import Flask
from werkzeug.middleware.dispatcher import DispatcherMiddleware

# ... wrap flask app with WSGIMiddleware ...
app.mount("/v1", sub_api_app)
```

Analogy:
A department store. The main store (FastAPI) has a small Starbucks (Flask app) inside it. They are separate entities but accessible from the same building.

⚡ One-liner to remember:
`app.mount("/sub", sub_app)` allows running multiple apps in one process.

## Q26. How do you serve static files?
Concept:
Use `StaticFiles` from `fastapi.staticfiles`.

Example:
```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="static"), name="static")
```

Analogy:
A public bulletin board. Anything you pin there (put in the folder) is visible to everyone walking by.

⚡ One-liner to remember:
Mount `StaticFiles` to a path to serve CSS/JS/Images.

## Q27. How do you enable CORS?
Concept:
Add `CORSMiddleware` to the application stack.

Example:
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

Analogy:
Opening the border gates. By default, the border is closed (Same Origin Policy). CORS Middleware is the border control officer allowing specific countries (Origins) to enter.

⚡ One-liner to remember:
Use `CORSMiddleware` to whitelist origins.

## Q28. What is the difference between `Field`, `Query`, and `Path`?
Concept:
They are all subclasses of `Param`.
- `Path`: For path parameters (URL).
- `Query`: For query parameters (URL).
- `Field`: For Pydantic models (Request Body).

Example:
```python
class Item(BaseModel):
    name: str = Field(description="The name")

def read(
    q: str = Query(None), 
    item_id: int = Path(...)
): ...
```

Analogy:
Different labels for different containers. `Path` labels the shelf. `Query` labels the clipboard. `Field` labels the box contents.

⚡ One-liner to remember:
`Field` for Body Models, `Query` for URL params, `Path` for URL variables.

## Q29. How does FastAPI handle blocking code?
Concept:
If you define a path operation with normal `def` (not `async def`), FastAPI runs it in a **threadpool**. This prevents blocking the main event loop.

Analogy:
A specialized waiting room. If a task takes a long time (Blocking), it gets sent to the waiting room (Threadpool) so it doesn't block the hallway (Event Loop).

⚡ One-liner to remember:
FastAPI automatically offloads synchronous `def` endpoints to a thread pool.

## Q30. How do you make parameters optional?
Concept:
Give them a default value of `None` (and type hint `Optional` or `Union[str, None]`).

Example:
```python
from typing import Optional

@app.get("/items/")
def read_items(q: Optional[str] = None):
    if q:
        return {"item": q}
    return {"item": "default"}
```

Analogy:
"Would you like a receipt?" It's an optional question. If you say nothing (None), the transaction proceeds without it.

⚡ One-liner to remember:
Default value = `None` makes a parameter optional.
