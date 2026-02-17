________________________________________
name: python-best-practices description: > Comprehensive Python best practices for coding, reviewing, and maintaining code. Ensures use of Python 3.10+, full type hinting, strict static analysis (Mypy/Pyright), consistent formatting (Black, Ruff with 120-char lines, double quotes, sorted imports), proper logging (structured, contextual), robust architecture (clear imports/project structure, dependency injection), safe async patterns (non-blocking, FastAPI-friendly, shared client lifecycles), solid exception handling (chaining, fail-fast, no silent errors), secure coding (no secrets in logs, env config), and thorough testing (pytest, fixtures, high coverage). Use this skill for any Python-related task to enforce these standards.
________________________________________
Python Best Practices Skill
Use this skill whenever working on Python code (writing new code, reviewing, or refactoring). It provides step-by-step guidelines to ensure code quality, maintainability, and compliance with modern best practices.
Ensure Modern Python Version
•	Python 3.10+: Always use the latest stable Python version (at least 3.10, ideally newer). Newer versions bring performance improvements and language features (e.g. pattern matching in 3.10, improved typing in 3.11+). If the environment Python is older, plan to upgrade to the latest stable release. For example, check the version with python --version and if it's outdated, instruct to update (via pyenv, apt, etc., as appropriate). Using the latest Python ensures access to modern syntax and stdlib enhancements.
•	Environment Update: If you encounter old syntax or deprecated modules, update the code to modern equivalents (e.g. use pathlib instead of os.path, f-strings instead of % formatting, etc.). Recommend updating project dependencies to support the newer Python if needed. Aim for compatibility with the newest Python release.
•	Example:

 	$ python --version
Python 3.8.10
 	If the version is below 3.10, prompt to upgrade: e.g., use pyenv install 3.12.0 or appropriate system package to install Python 3.12.
Type Hinting and Static Analysis
•	Full Type Hints: All functions, methods, and modules should have complete type annotations. This includes function parameters, return types, class attributes, and variables where helpful. Use Python’s typing features extensively: generics (TypeVar, generic classes/functions), Optional for nullable types, Union (or | in 3.10+ syntax), etc. For complex structures, use TypedDict for dicts with fixed schema, and Protocol or abstract base classes to define interfaces (for dependency injection). Leverage advanced types like Literal for finite sets of values, and Final for constants. Avoid using Any unless absolutely necessary; prefer more specific types or generics.
•	Mypy in strict mode: Always run static type checking in the strictest setting. Use Mypy with --strict which enables a collection of rigorous checks[1][2]. This ensures that no type errors are hiding – you'll only get runtime type errors if you've explicitly silenced Mypy (which we avoid). Mypy’s strict mode turns on many options (like disallowing untyped defs, requiring return types, etc.) to catch potential issues. If third-party libraries lack type stubs, configure Mypy (e.g. ignore_missing_imports) on a case-by-case basis rather than globally ignoring type errors. Always address type errors by improving the code or types; do not just ignore them.
•	Pyright: Consider using Pyright (the static analyzer behind VS Code/Pylance) in addition to or instead of Mypy. Pyright is very fast and sometimes more strict/correct in certain scenarios[3]. It can be integrated into editors or CI for instant feedback. Ensure no type errors are reported by Pyright either. In VS Code, use the Pylance extension with "python.analysis.typeCheckingMode": "strict" for real-time strict checking. Both Mypy and Pyright should pass with 0 errors on the codebase.
•	No # type: ignore: Do not scatter # type: ignore comments unless absolutely necessary. Each ignore is effectively an admission of a type issue. If you must use one (e.g. for a bug in the type checker or a third-party stub issue), target it to a specific error code and add a TODO to revisit it[4]. Our goal is to eventually remove all ignore comments. Mypy’s --warn-unused-ignores flag should be enabled to catch stale ignore comments[5]. Instead of ignoring errors, prefer to refactor the code or add the appropriate annotations to satisfy the type checker.
•	Generics and casting: Use generics to make functions and classes type-safe for different types (e.g. from typing import TypeVar, Generic). Avoid overly broad types – for example, if a function returns a dictionary with fixed keys, define a TypedDict for that. Use Protocol for duck-typed APIs (e.g. if a function can accept any object with a .read() method, define a Protocol instead of using Any). If you need to downcast or convince the type checker of a type, use typing.cast rather than ignore comments, and document why the cast is safe. Again, casts should be rare. Leverage Final for constants to prevent reassignment.
•	Example: A properly annotated function and TypedDict:

 	from typing import TypedDict, Protocol, TypeVar, Sequence, Literal

class UserData(TypedDict):
    id: int
    name: str
    active: bool

T = TypeVar("T")
def first_item(seq: Sequence[T]) -> T:
    """Return the first item of a sequence (non-empty)."""
    return seq[0]

# Using the TypedDict:
def welcome_user(user: UserData) -> str:
    status: Literal["active", "inactive"] = "active" if user["active"] else "inactive"
    return f"Welcome, {user['name']}! Your account is {status}."
 	This function and TypedDict have full hints. Mypy/Pyright will catch if, say, we access a wrong dict key or misuse the types.
•	Static Analysis Commands:
After writing code, run type checks:

 	$ mypy --strict .
# (Should produce no output if all is well.)
 	If there are issues, Mypy will list them with file and line, e.g.:

 	src/app.py:45: error: Incompatible types in assignment (expression has type "str", variable has type "int")
Found 1 error in 1 file (checked 42 source files)
 	Fix all such errors instead of ignoring them.
Alternatively, run Pyright (if installed) for a second opinion:

 	$ pyright
 	Pyright will output any type errors or possibly additional insights (it might catch unsafely uninitialized attributes, etc.).
•	Benefit: With strict type checking, you gain high confidence that your code is type-safe. Mypy’s philosophy: if run with --strict, you'll essentially never get a runtime type error unless you bypass the checker[6][2]. Embrace this by not bypassing it!
Code Formatting and Linting
•	Use Black for Formatting: Adopt Black as the auto-formatter to enforce a uniform code style. Configure Black with a 120 character line length (the default is 88, but we use 120 for this skill’s standards). Black will reformat code to have consistent indentation, spacing, and line breaks. Notably, Black prefers double quotes for strings and will replace single quotes with double in most cases[7] (unless escaping would increase). This aligns with our preference for double quotes for consistency. Ensure Black is configured to not skip string normalization (we do want it to normalize quotes to double). Also, Black automatically adds trailing commas in collections and dicts when multiline, and it keeps functions to one parameter per line when formatting multiline signatures. Accept these defaults as they improve diffs and readability.
•	Use Ruff for Linting: Employ Ruff as a fast linter and code quality tool. Ruff can replace Flake8, isort, etc., providing over 800 rules at high speed[8][9]. Configure Ruff in pyproject.toml (or ruff.toml) to match Black’s style: set line-length = 120 to be consistent across tools. Also set quote-style = "double" so Ruff enforces double quotes for strings[10]. Import sorting is handled by Ruff’s built-in isort rules: ensure they are enabled (Ruff enables basic rules by default; to be safe, include I for isort in the select rules). Group imports into standard library, third-party, and local, separated by blank lines[11] – both Black and Ruff will help enforce this (Black won't reorder, but Ruff (with isort) will sort imports within groups and can enforce grouping). No unused imports or variables should remain – Ruff (via Pyflakes rules) will flag those (e.g., unused imports show as F401). Also enable Ruff’s other valuable rule sets: e.g. "B" (bugbear for common pitfalls), "E" and "W" (pycodestyle errors/warnings), "N" (PEP8 naming), "D" (docstring style), "S" (Bandit security checks), "C4" (comprehensions), etc., to catch a wide range of issues[12]. The goal is a clean lint: zero warnings or errors.
•	No Manual Formatting: Rely on Black (and Ruff’s autofix for some lints) to format code, rather than manual style tweaks. This eliminates debates about code style. Black’s output is idiomatic and consistent (PEP8 compliant in spirit[13], aside from the line length which we configure). We specifically use 120 characters line length (wider than PEP8’s default 79/99[14] but acceptable for modern displays and agreed by our team). Be mindful that extremely long lines can hurt readability, so if you find yourself disabling the line-length check often, consider if the code can be refactored or split. But in general, 120 chars is our max. Double quotes are enforced (Black will do this automatically[7], and Ruff’s Q000 rule can flag any stray single quotes if we ever disable Black’s normalization).
•	Import Organization: Always put imports at the top of the file (after any module comments or docstrings)[15]. Use one import per line (no combining unrelated imports on one line)[16]. Order imports by groups: (1) Standard library, (2) third-party libraries, (3) local application imports[11]. Within each group, sort alphabetically by module name. Insert a blank line between each group[11]. Do not use wildcard imports (from x import *); always import explicitly what is needed. If an import is unused, remove it – Ruff will catch unused imports (F401). This keeps the namespace clean and avoids dragging in unwanted dependencies.
•	Lint/Format Commands:
Format the code with Black:

 	$ black --line-length 120 .
reformatted 10 files
All done! ✨ 🍰 ✨
 	(Black will list how many files it reformatted; if run with --check, it would list files that would be reformatted without changing them.)
Lint the code with Ruff:

$ ruff check .
Example output if issues are found:

src/utils.py:5:8: F401 `os` imported but unused
src/utils.py:6:1: E302 expected 2 blank lines, found 1
src/utils.py:10:27: F841 local variable `temp` is assigned to but never used
Found 3 errors. 
The codes indicate the rule (e.g., F401 unused import, E302 blank lines issue, F841 unused variable). Fix these: remove unused imports/vars, add blank line, etc. Ruff can fix some automatically:

$ ruff check --fix .
Fixed 2 errors.
After autofix, remaining issues (if any) must be fixed manually. Finally, run Ruff again until it reports no issues. You can also use ruff format . to apply Ruff's auto-formatting (which includes import sorting and some formatting akin to Black). We integrate Black and Ruff, so both should be run (Black for full formatting, Ruff mainly for lint rules and as a safety net).
•	Consistent Style: By using these tools, the code style will be consistent: 4-space indent, spaces around operators, etc. Black enforces one true style (no option to tweak most things, which is good). For example, Black will transform:
 	if(x<5): print("hi")
 	into
 	if x < 5:
    print("hi")
 	automatically. Trust these tools for formatting so we can focus on logic. Code reviews then focus on correctness, not spacing.
•	Note on Configuration: Include a pyproject.toml with Black and Ruff settings. For example:

 	[tool.black]
line-length = 120
skip-string-normalization = false  # ensure double quotes
[tool.ruff]
line-length = 120
select = ["E", "W", "F", "B", "I", "C4", "N", "D", "S"]
ignore = []  # (if we have specific ignores, list by code)
[tool.ruff.format]
quote-style = "double"
 	This config enforces our choices. The select list above enables various rule categories: E/W (pycodestyle), F (pyflakes), B (bugbear), I (isort/imports), C4 (comprehension improvements), N (naming convention), D (docstring format), S (security bandit rules). These help catch potential bugs and enforce best practices across the board[12].
Logging Best Practices
•	Use the logging Library: Avoid using print statements for anything other than quick debugging. In production code, use Python’s built-in logging module (or a framework like structlog for structured logging). Set up a logger for your module (logger = logging.getLogger(__name__)) and use appropriate log levels (logger.debug, info, warning, error, critical). Configure logging early (basicConfig or a YAML/dict config for complex setups) so that logs are captured. Logs should include timestamps, log level, and module info by default (basicConfig can do this). This provides context when reading logs.
•	Structured and Contextual Logging: Wherever possible, log structured data instead of free-form strings. For example, rather than logger.info("User %s logged in", user_id), you can do logger.info("User logged in", extra={"user_id": user_id}) or use structlog to bind user_id as a context variable. This makes it easier to filter logs by fields. In web apps, include request identifiers or user identifiers in log context to trace actions. Less is more: log the key events and errors, not every minor step (avoid log clutter). Strive for one log entry per significant action (sometimes called canonical log lines); a single well-structured log entry per request can be more useful than dozens of lines[17][18].
•	Log to stdout (and/or a file): For containerized or 12-factor apps, logging to unbuffered stdout is a good default[19]. It allows external systems (Docker, Kubernetes, etc.) to capture logs centrally. Ensure the logger is not set to buffer output too long (use logging.StreamHandler(sys.stdout) with no buffering, or flush on each write if needed for real-time logs). If file logging is needed, set up rotation (don’t let logs grow indefinitely).
•	No Sensitive Data in Logs: Never log secrets or sensitive personal data. This is crucial for security and compliance. Review log statements to ensure they don’t include passwords, API keys, tokens, or personal identifiable info (PII) like user passwords, credit card numbers, etc. An innocent debug log can leak an API key if not careful[20][21]. If you need to log data objects, consider sanitizing or redacting sensitive fields. For example, if logging a user object, omit or mask the password field. Using tools like Pydantic’s SecretStr for configuration secrets is helpful – it will display as "SecretStr('**********')" when printed, preventing accidental exposure[22]. Also be cautious not to log the content of environment variables or config values that are secrets. It's easy to accidentally do logger.debug("%s", os.environ) and leak secrets – avoid that.
•	Use Appropriate Levels: Use DEBUG for verbose diagnostic information (generally turned off in production), INFO for normal significant events (startup, shutdown, key actions), WARNING for something unexpected but not fatal, ERROR for handled errors, and CRITICAL for system-wide failures. Do not abuse levels (e.g., don’t use ERROR for control flow or normal situations). This ensures that filtering by level is meaningful.
•	Exception Logging: When catching exceptions, use logger.exception(...) to automatically log the stack trace[23]. This uses the current exception info and is equivalent to logger.error(..., exc_info=True). Always provide some context in the log message. For example:
 	try:
    result = do_important_thing()
except ImportantError as e:
    logger.exception("Failed to do important thing: %s", e)
    raise  # or raise a custom exception after logging
 	This will log the traceback, making debugging easier. Never catch an exception without logging it (unless you truly handle it fully). If you choose to handle an error silently (very rare), at least consider logging a warning so it's not completely hidden.
•	Structured Logging: For advanced use, consider using structlog or the standard library’s logging.LoggerAdapter/LogRecord attributes to log in key-value format (JSON logs). In production, JSON logs are easier to aggregate and search (ELK stack or other aggregators parse them readily)[24][25]. Structlog can output JSON and let you bind context easily (e.g., logger = structlog.get_logger().bind(user_id=user.id)). Adopting structured logging is highly encouraged for larger applications.
•	Logging Configuration: Ensure that the logging configuration doesn’t degrade performance. For instance, avoid very expensive operations in log messages (since even if a message is DEBUG level and debug is off, the f-string will still be evaluated if you write logger.debug(f"Result: {expensive_computation()}") – prefer the % formatting or , style which delays interpolation). Also, in async contexts, use thread-safe handlers or ensure no blocking handlers in the event loop.
•	Example:
 	import logging
logger = logging.getLogger("myapp")
logger.setLevel(logging.INFO)
handler = logging.StreamHandler()
formatter = logging.Formatter("%(asctime)s %(levelname)s [%(name)s] %(message)s")
handler.setFormatter(formatter)
logger.addHandler(handler)

# Usage:
logger.info("Service started", extra={"version": "1.2.3"})
try:
    value = process_data(data)
    logger.debug("Processed data size=%d", len(value))
except Exception as e:
    logger.exception("Error processing data for id=%s", data.get("id"))
    raise
 	This sets up a basic logger. In the exception case, logger.exception will log the traceback. The log outputs might look like:

 	2026-02-03 11:05:04,123 ERROR [myapp] Error processing data for id=42
Traceback (most recent call last):
  ...
 	And possibly a JSON if structlog was used. The key is that we've recorded the error with context.
•	Don’t Duplicate Logs: If an exception propagates through multiple layers, avoid logging it at every layer (which causes repeated stack traces). Ideally, log at the point where you handle it or where it’s most useful. For example, in a web app, you might catch exceptions at the boundary (framework level) to return an HTTP response and log the error once[26][27]. Duplicate logging makes noise and can confuse by appearing as multiple errors.
•	Testing Logs: In tests, you can assert that certain actions log warnings or errors using caplog (pytest) or by configuring test loggers. This ensures your code does log important events.
Imports and Project Structure
•	Project Structure: Organize code into modules and packages logically. A common and recommended pattern is the src layout: have a top-level src/ directory and put your packages inside it (e.g., src/your_project/...). This prevents import confusion (you won’t accidentally import from a local module when you meant a standard library one, because the project isn’t on sys.path until you add src)[28]. Ensure each directory intended as a package has an __init__.py. Structure your project by feature or layer (for example, your_project/api/, your_project/core/, your_project/utils/, etc.) rather than one giant module. Each module should have a clear purpose.
•	Module Naming: Modules (filenames) should be lowercase, and use underscores if multiple words (e.g., data_processing.py). Keep module names short but descriptive. Avoid names that clash with standard library modules (like don’t name your module logging.py or json.py). If you have a package, you can have a module inside with the same name as the package for package-level definitions, but be cautious with import cycles.
•	Avoid Cyclic Imports: Design modules such that you don’t have circular import dependencies. If you find two modules importing each other, it’s a sign they might need to be refactored or combined. Sometimes using dynamic imports inside functions can break a cycle, but better to reorganize code to eliminate cycles.
•	Absolute Imports: Prefer absolute imports for clarity (from your_project.utils import helper rather than relative from ..utils import helper), especially for imports across packages[29]. Absolute imports are clearer about where the module is. Use relative imports only for closely related modules within the same package, when it improves clarity or to avoid very long import paths[30]. Even then, limit to one level up (from . import sibling_module). Consistency is key.
•	Import Grouping: As mentioned in formatting, always group imports into three sections: standard library, third-party, and internal[11]. Example:
 	# stdlib
import os
import sys
from datetime import datetime

# third-party
import requests
from sqlalchemy import create_engine

# local application
from myapp.config import settings
from myapp.utils import helper
 	Each group is separated by a blank line. Within a group, ideally sort alphabetically by module name. This makes it easy to see dependencies. Tools (isort/ruff) will enforce this ordering.
•	One Module, One Purpose: Try not to make modules too large. If a single .py file exceeds a few hundred lines or has many classes/functions covering different concerns, consider splitting it. For instance, if you have models.py that defines database models and also utility functions, maybe separate into models.py and model_utils.py, or a package. Smaller modules are easier to navigate. Code splitting improves maintainability[31]. On the other hand, don’t split just for the sake of tiny files – group related things logically.
•	Project README/Documentation: Maintain a high-level README or docs that explain the project structure and conventions, so new contributors know where to add things. E.g., "API routes are in api/ package, business logic in services/, DB models in models/, etc."
•	Example Structure:
 	myproject/
  pyproject.toml
  README.md
  src/
    myproject/
      __init__.py
      api/
        __init__.py
        routes.py
        middleware.py
      core/
        __init__.py
        models.py
        services.py
      utils/
        __init__.py
        helpers.py
      tests/
        ... (if co-located; or could be top-level tests/ folder)
 	In this example, internal imports use from myproject.core.models import X (absolute). This structure avoids naming conflicts and clarifies separation of concerns.
•	Init files: Use __init__.py to expose a clean public interface for your packages if needed. For instance, in myproject/core/__init__.py, you might import certain classes (from .models import BaseModel) to allow users to do from myproject.core import BaseModel. But avoid heavy logic in __init__.py (don’t put complex code there, it runs on import and can cause issues).
•	No Side-Effects on Import: Modules should generally not execute significant code on import (no starting servers, no major computations). Importing a module should ideally just define classes/functions. If you need to run setup code, do it under a if __name__ == "__main__": or in an initialization function that the application calls explicitly. This prevents surprises and makes tests/imports predictable.
•	Dependencies and Packaging: Keep a requirements file or pyproject dependencies section updated. Pin versions if necessary. Ensure the project is installable (with a proper setup.cfg/pyproject specifying packages=find_namespace_packages() or similar). This isn’t exactly code style, but it’s part of best practices for a Python project.
•	Grouping Code by Layer: Within modules, group related code together and use blank lines to separate. For example, in a class, group methods logically (maybe all getters together, etc.). At the module top, possibly define constants, then classes, then standalone functions – or whatever ordering makes sense, but be consistent.
Async and Concurrency Patterns
•	Non-Blocking Async: When writing asyncio-based code (e.g., FastAPI endpoints, any async def functions), never call blocking functions directly. Blocking calls (CPU-intensive work or I/O that isn’t using await) will block the entire event loop, defeating the purpose of async concurrency[32]. Common pitfalls:
•	Using standard library or blocking libraries in async code (e.g. calling time.sleep() instead of await asyncio.sleep(), or using requests to make HTTP calls instead of an async client like httpx).
•	Heavy CPU computations in async code (they will freeze the event loop).
To handle these: - Use async equivalents of I/O operations: databases -> use async DB drivers or ORMs (e.g. databases library or SQLAlchemy 1.4 async engine), HTTP -> use httpx.AsyncClient or aiohttp, file I/O -> use aiofiles or run file operations in a thread executor if needed. - If you must call a blocking function inside async code, offload it to a thread or process. For example, in Python 3.9+, you can do: result = await asyncio.to_thread(blocking_func, *args). Or use loop.run_in_executor(None, blocking_func, *args) for older versions. This runs the blocking call in a separate thread so the event loop isn’t stuck. - For CPU-bound tasks that are intensive, consider using concurrent.futures.ProcessPoolExecutor or an external task queue (like Celery or RQ) rather than letting them run in the async loop.
•	FastAPI (or any async web framework): In FastAPI, if an endpoint function is declared async def, it is expected to be non-blocking. If you use normal sync DB library (like regular SQLAlchemy session), declare your path function as def (not async); FastAPI will run it in a threadpool automatically[33][34]. Conversely, if using an async DB, use async def. Mixing sync DB calls in async def is a bug – it will block the loop and prevent other requests from being handled. So follow the rule: use async def only with truly async operations inside. If uncertain, it’s safer to use normal def so FastAPI knows to offload to a thread[33]. You can mix async and sync routes in FastAPI; it will do the right thing in terms of running them[34].
•	Shared Client Lifecycles: For services like database connections, HTTP clients, etc., it’s best to create a single instance and reuse it rather than creating on each request. In FastAPI, you can use the startup event to initialize a DB engine or an HTTP client and store it (e.g. as app.state.db or as a global). Or use FastAPI’s dependency injection with yield to create a single instance for the app. For example, create a single httpx.AsyncClient() at startup and reuse it for all requests (closing it on shutdown). This avoids overhead of reconnecting each time and respects connection pooling. Similarly, for a DB, use a single engine or connection pool. Creating clients per request is inefficient and can exhaust resources (e.g., too many open connections).
•	Example (FastAPI):

 	app = FastAPI()
@app.on_event("startup")
async def startup_event():
    app.state.http_client = httpx.AsyncClient()
    app.state.db_engine = create_async_engine(DB_URL)

@app.on_event("shutdown")
async def shutdown_event():
    await app.state.http_client.aclose()
    await app.state.db_engine.dispose()

@app.get("/users/{id}")
async def get_user(id: int):
    # reuse the clients from state
    resp = await app.state.http_client.get(f"https://api.example.com/users/{id}")
    data = resp.json()
    async with AsyncSession(app.state.db_engine) as session:
        user = await session.get(User, id)
        ...
    return {"external": data, "local": user.as_dict()}
 	In this example, we don’t create a new client or engine on each call; we reuse the ones set up at startup. This pattern should be applied for any heavy resource.
•	Avoid Global Event Loop Calls: Do not use asyncio.run() inside an already running async context (like inside FastAPI route) – that will spawn a nested loop and is generally wrong. Instead, use await directly. Also avoid low-level loop manipulations unless necessary; rely on high-level APIs.
•	Concurrent Tasks: If you need to do multiple things concurrently in async (e.g., fetch data from two services at once), use asyncio.gather or create tasks:
 	task1 = asyncio.create_task(service1.fetch(...))
task2 = asyncio.create_task(service2.fetch(...))
result1 = await task1
result2 = await task2
 	This will run them in parallel (concurrently). Be cautious to properly handle exceptions from tasks (if one fails, gather will raise; you might need return_exceptions=True or try/except around awaits). Always await tasks or they may get lost (avoid creating tasks without awaiting unless you add proper callbacks or background task management, e.g. FastAPI’s BackgroundTasks for truly background work outside response path).
•	Thread Safety: If using threads (like default FastAPI for sync routes uses a threadpool, or if you manually offload work), make sure data accessed is thread-safe. Avoid shared mutable state without locks. For example, don’t have a global list that both the main thread and a worker thread modify without synchronization. In async code, use asyncio.Lock if needed to protect shared state across tasks.
•	Async Libraries: Use high-quality async libraries. For example, for databases: asyncpg for Postgres, or SQLAlchemy’s async ORM. For web requests: httpx or aiohttp. For scheduling: asyncio.sleep for delays. Ensure library versions are updated; early versions of some async ORMs had issues, prefer stable releases.
•	FastAPI-specific: Use dependencies (Depends) to manage things like DB session per request. For example, you can have:
 	async def get_db():
    async with AsyncSession(engine) as session:
        yield session
@app.get("/items/")
async def read_items(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Item).where(...))
    ...
 	This pattern ensures each request gets a session and it’s closed after. It’s non-blocking and uses shared engine.
•	No Mixing Async with Blocking Frameworks: If you use something like Django (sync) with Celery (also sync), keep that separate from async code. Or if mixing, be very careful. Generally, keep async code purely async.
•	Testing Concurrency: Write tests to ensure your async functions behave (use pytest.mark.asyncio to test async functions). If you had a bug where an async function accidentally blocked, you might catch it in load testing or with specific timeout tests.
•	Performance and Concurrency: Remember, Python async is single-threaded by default (except for the threads you explicitly spawn). It’s good for I/O bound tasks with high concurrency. It does not make CPU-bound tasks faster – those need multiprocessing or to be in an external service. Keep this in mind when designing your solution.
•	Summary: Never block the event loop[32]. Use await for I/O, thread executors for CPU/blocking calls, reuse resources, and leverage FastAPI’s features for clean async code. This will ensure your app scales with many concurrent tasks without threads getting stuck.
Exception Handling
•	Fail Fast and Loudly: When something goes wrong, it’s usually better to raise an exception early than to let bad data propagate. Validate inputs at boundaries (e.g., at function entry or request parsing) and raise exceptions (or return errors) if data is invalid. This "fail fast" approach prevents undefined behavior later. Do not continue executing a function if you know preconditions aren’t met – raise an exception immediately.
•	Use Exception Chaining: When catching an exception and re-throwing a different one, use raise NewException(...) from e. This preserves the original traceback and exception context, which is invaluable for debugging[35]. For example:
 	try:
    data = json.loads(input_str)
except ValueError as e:
    raise DataParsingError("Invalid JSON format") from e
 	This way, the stack trace will show both the DataParsingError and the original ValueError. It’s much easier to diagnose. Only in rare cases (where the original exception is truly not relevant) should you use raise ... from None to suppress the context.
•	Catch Specific Exceptions: Avoid blanket except Exception: or except BaseException: catches. Catch the specific exception(s) you expect and know how to handle. If you catch Exception broadly, you might inadvertently hide bugs (like programming errors). For instance, if you only expect a KeyError, just catch that. It’s fine to use a broad catch at a high level (like an outermost loop or a framework integration point) to log and handle any unexpected error, but within application logic, be as specific as possible[36]. If using broad catch, definitely do not just pass – at least log it as error.
•	Never Swallow Exceptions: Do not write empty except blocks or except that just log and continue as if nothing happened (unless you truly can continue safely). Silent failures make bugs hide. If you catch an exception and decide it’s not critical, at least log a warning about it. Typically, only catch an exception if you can handle it meaningfully (e.g., try an alternative or provide a default behavior). Otherwise, let it propagate – it might be handled at a higher level or cause the program to crash (which in some cases is better than corrupt state). Example of what not to do:
 	try:
    process(task)
except Exception:
    # bad: silently ignore all errors
    pass  
 	This is dangerous – it ignores all exceptions, even ones we didn’t expect, possibly leaving the system in an unknown state[36].
•	Logging and Rethrowing: Common pattern: catch a low-level exception, log it with context, then raise a higher-level exception for the caller. This is fine. Ensure logging uses logger.exception as noted, to include traceback. For example:
 	except DatabaseError as e:
    logger.exception("Database error while fetching user %s", user_id)
    raise ServiceError("Failed to fetch user data") from e
 	This way, you translate the exception to something the upper layer (service or API layer) understands, but didn’t lose the context. The from e keeps the chain[35].
•	Custom Exceptions: Define custom exception classes for your domain (e.g., UserNotFoundError, PaymentFailedError). This makes it easier to catch specific ones without catching unrelated exceptions. Custom exceptions should usually subclass Python’s Exception (or a more specific built-in like ValueError if it semantically fits). Provide meaningful messages. For example, a ValidationError exception that includes which field was wrong. Use exceptions to separate error handling logic from normal logic.
•	Cleanup in Exceptions: Use finally blocks or context managers to ensure resources are cleaned up on exceptions. For instance, if you open a file or acquire a lock, use with statements so they auto-clean on exceptions. Or manually:
 	lock.acquire()
try:
    ... do stuff ...
finally:
    lock.release()
 	This guarantees the release happens even if an error occurs.
•	Don’t Overuse Exceptions for Flow: Exceptions shouldn’t control normal flow (e.g., don’t use StopIteration or similar as a way to break out of loops unless it’s an iterator protocol). Use them for exceptional conditions. For example, use return None or a special value for an expected missing result, rather than throwing an exception that isn’t truly “exceptional”. This is context-dependent though – in some APIs, not finding a result might be considered exceptional (then raising is fine). Use your judgment, but document it.
•	Raising Built-ins vs Custom: If one of Python’s built-in exceptions semantically fits (ValueError, TypeError, KeyError, etc.), it’s okay to raise those. It’s often clear to use ValueError for invalid arguments, etc. Custom exceptions are for more domain-specific issues that built-ins don’t describe well.
•	Graceful Error Handling at Boundaries: At the top level (like an API endpoint or a CLI main function), catch exceptions and handle them gracefully. For a CLI, that might mean printing a user-friendly error message instead of a raw traceback (unless --debug flag is on). For an API, that means catching application exceptions and converting to an HTTP response (e.g., return 404 if UserNotFoundError, etc.). FastAPI allows exception handlers for custom exceptions[26]. Use those to map exceptions to responses. This provides a better experience while still logging the issue.
•	Example:
 	class ConfigError(Exception):
    pass

def load_config(path: str) -> Config:
    if not os.path.exists(path):
        raise ConfigError(f"Config file not found: {path}")
    ...
    try:
        data = json.load(f)
    except json.JSONDecodeError as e:
        raise ConfigError("Invalid JSON in config") from e
    return Config(**data)
 	Here we raise ConfigError for missing file or bad JSON, chaining the JSON error. The caller can catch ConfigError and handle appropriately (e.g., use defaults or exit program with message). The original JSON error is preserved for debugging.
•	Testing Exceptions: Write tests to ensure that functions indeed raise exceptions for bad inputs. Also test that exceptions are chained or carry correct messages. If you have a high-level handler (like an API global exception handler), test that as well (e.g., sending a bad request yields the proper HTTP error).
•	No Bare except: Avoid except: with no exception specified – it even catches system-exiting exceptions. Always at least except Exception: if you mean “any regular exception”. And as said, prefer specific ones.
•	Exception Messages: The message in an exception should be clear on what went wrong, ideally including relevant data (but not sensitive info). E.g., "User ID 123 not found" rather than "Not found". This message might propagate to logs or error responses, so make it count.
•	Use assertions for invariants: In internal code, assert can be used to check conditions that should never happen. Assertions are not a replacement for normal error handling, but they are useful for catching programmer errors during development. Do not catch AssertionError; let them crash in dev (they get removed if running with -O optimize flag in production, so don’t rely on them for actual runtime logic).
Data Handling and File I/O
•	Use Context Managers: When working with files (or any resource like network sockets), always use context managers (with statements) to ensure proper cleanup[37]. For example, with open('data.txt', 'r') as f: data = f.read(). This ensures the file is closed automatically, even if an error occurs while reading. Similarly, for locks (with lock:), or for temporary change of state, use context managers if available. This prevents resource leaks and is cleaner than try/finally for most cases.
•	Read/Write in Chunks for Large Data: If dealing with large files or streams, do not load everything into memory if you don’t have to. Iterate over the file or read in chunks. For example, to process a huge log file, do:
 	with open('big.log', 'r') as f:
    for line in f:
        process(line)
 	This reads line by line, instead of f.read() which could blow up memory. If you need to process large binary files, consider using memory-mapped files (mmap) or streaming through libraries like csv for CSVs (which can iterate), etc. The principle is to stream data rather than slurp, whenever the data size is significant[37].
•	Avoid Memory Copies: When handling binary data, use bytes wherever possible and avoid unnecessary decoding/encoding. For example, if you need to hash a file, it’s more efficient to open in binary and feed chunks to hash function than reading entire content into a giant bytes object.
•	File Modes and Encoding: Open text files with explicit encoding (open('file.txt', 'r', encoding='utf-8')) to avoid platform-specific defaults. Use binary mode ('rb'/'wb') for non-text data. For writing text files, decide if you need to append or overwrite and use 'a' or 'w' accordingly. In append mode for logs, open with 'a' plus maybe buffering=1 for line-buffered writes if immediate flush is needed.
•	Error Handling in I/O: Anticipate exceptions like FileNotFoundError, PermissionError, IOError when doing file operations. Handle them gracefully or propagate with a clear message. For instance, if a config file is missing, catch FileNotFoundError and inform the user "Config file not found". Do not just let a stack trace bubble up to a user if it can be handled more nicely.
•	Temporary Files: Use the tempfile module for creating temp files or directories, rather than hardcoding paths like /tmp. tempfile.TemporaryFile() or NamedTemporaryFile give you a safe temp file.
•	CSV/JSON: Use Python’s libraries for parsing data formats (csv, json, xml, etc.) rather than writing your own parser, to avoid common bugs. When using csv or json, handle exceptions (malformed data could throw).
•	Large Data and Pandas/Numpy: If you’re using pandas or numpy for data, be mindful of memory. For example, when reading a large CSV with pandas, consider specifying data types to reduce memory usage (pandas can use less memory if you tell it some columns are categorical or smaller int types). This is beyond basic Python, but part of data handling best practice.
•	Don’t Repeat I/O Unnecessarily: If you need the same data multiple times, read it once and cache it rather than reading file multiple times (unless the file changes and you need updated data). I/O is relatively slow. But also avoid premature caching of huge data if not needed.
•	File Paths: Use pathlib.Path for path manipulations; it’s more readable than os.path functions and reduces errors. For example, Path('data') / 'items.txt' concatenates paths nicely. Also, use path.exists() or better, wrap operations in try/except rather than manually checking existence (to avoid TOCTTOU race conditions).
•	Permissions: Consider the file permissions when writing files. For sensitive data, ensure correct file modes (e.g., not world-readable). Use os.chmod if needed after creating files.
•	No Hardcoded Paths: Avoid writing code that assumes a certain working directory or absolute paths that won’t exist on another system. Use config or environment variables for file locations, or use paths relative to the project or user home if appropriate. For example, for a CLI tool, maybe default to ~/appname/config.json for a config path, but make it overrideable.
•	Data Validation: When reading external data (files, user input, etc.), validate it. For example, if you parse a file and expect a number on each line, handle the case where it’s not a number, perhaps by logging an error and skipping or raising a custom exception.
•	Example:
 	from pathlib import Path
data_path = Path("data/input.txt")
if not data_path.exists():
    raise FileNotFoundError(f"Required file not found: {data_path}")
total = 0
with data_path.open("r", encoding="utf-8") as f:
    for line in f:
        line = line.strip()
        if not line:
            continue
        try:
            num = int(line)
        except ValueError:
            logging.warning("Skipping invalid number: %r", line)
            continue
        total += num
print(f"Total = {total}")
 	Here we open with a context manager, read line by line, handle a possible ValueError with a warning (not crashing the whole process on one bad line), and ensure the file existed with a clear error if not.
•	Resource Management: For other resources like database connections or network sockets, similarly ensure they are closed after use. Use context managers or try/finally. e.g., with socket.create_connection(addr) as sock: ... (in Python 3.10+, sockets have context support).
•	Ensure to Close Files: If not using with, always call f.close(). Unclosed file handles can lead to resource leaks. In long-running processes, this is critical. Python’s garbage collector will eventually close files, but it’s not deterministic – better to close promptly.
Security and Secrets Management
•	Never Hardcode Secrets: API keys, passwords, tokens, and secrets must not be hardcoded in source code[38]. This is a serious security risk (if the repo becomes public or an attacker inspects the code, secrets are exposed). Instead, fetch secrets from environment variables, configuration files (that are not committed to VCS), or secret management services. Use placeholders or config for such values. For example, get DB_PASSWORD from an env var rather than writing it in the code. If you need a default for local dev, use a .env file that is gitignored, and load it in dev only.
•	Environment Variables Best Practices: Environment variables are a common way to supply secrets and config. Do not log them (as mentioned, they can contain secrets)[39][21]. Be cautious with error messages that might include env var content. Also, document which env vars your application uses. Consider using a library like python-dotenv for local development to load a .env file, but never commit that file. In production, ensure the deployment mechanism provides the env vars securely (e.g., in Docker, use -e flags or Kubernetes secrets). After using env vars to configure, you don’t need to remove them from os.environ, but just be mindful of not accidentally printing them.
•	Use Secret Managers for Production: For serious applications, prefer integrated secret management (like AWS Secrets Manager, Vault, etc.) instead of plain env vars. Env vars are fine but have limitations (lack of automatic rotation, potential exposure via process listing or child processes). A best practice is to use env vars for non-sensitive config and use secret stores or vaults for actual secrets, injecting them at runtime.
•	Sanitize Logs and Errors: As covered, ensure secrets don’t end up in logs. Also, don’t expose secrets or stack traces to end-users. For instance, if an error occurs in a web app, return a generic message to the user, but log the detailed error internally. Never include sensitive data in exception messages. E.g., if a database connection fails, an exception might include the connection string which has a password – catch that and log a safer message or strip out the password. Many libraries’ error messages won’t include passwords, but be aware.
•	Configuration Design: Structure your configuration such that it can be easily overridden for different environments (dev, staging, prod). Use environment-specific config files or environment variables. Consider using a pattern like 12-factor app: config via environment. Or use a library like Pydantic’s BaseSettings to define a config object that reads from env vars. For example:

 	from pydantic import BaseSettings
class Settings(BaseSettings):
    db_url: str
    api_key: str
    debug: bool = False

    class Config:
        env_file = ".env"
settings = Settings()
 	This will automatically read env vars DB_URL, API_KEY, etc., and we avoid hardcoding these in code. Document what config is needed (maybe in README or sample .env).
•	No Secrets in VCS: Ensure .gitignore is configured to ignore files that may contain secrets (like .env, config files with secrets, etc.). Use tools like git-secrets or pre-commit hooks to scan for accidental commits of secrets. As Arjan codes noted, many breaches happen via secrets in repos[40] – we avoid that.
•	Encryption and Hashing: If you need to store secrets (like user passwords), never store plaintext. Hash passwords with a strong algorithm (bcrypt, argon2, etc., not plain MD5 or SHA1). For any cryptography, prefer well-vetted libraries (PyNaCl, cryptography, etc.) over writing your own. Also, handle encryption keys as secrets themselves.
•	Least Privilege: When integrating with external systems (databases, APIs), use credentials that have only the necessary permissions. E.g., don’t use a DB superuser in your app if a read/write user will do. This limits damage if credentials leak.
•	Secure Sensitive Information: If your app deals with personal data, follow compliance guidelines (GDPR, etc.): e.g., avoid logging personal info, ensure you can remove data if needed, etc. This might be beyond coding style, but important for overall best practice.
•	Dependency Security: Keep dependencies updated to include security patches. Use tools like pip-audit or safety to scan for known vulnerabilities in deps. If a library is unmaintained and has vulns, consider alternatives.
•	No eval or exec on untrusted input: Avoid using Python eval() or exec() on anything derived from user input. This can lead to remote code execution vulnerabilities. If you absolutely need dynamic evaluation, use ast.literal_eval for safe eval of literals, or sandbox it heavily. But usually, it’s not needed.
•	Validate Inputs: For web apps or any user input, validate and sanitize. Use libraries or frameworks’ validation features (e.g., Pydantic models in FastAPI to validate request data). This prevents malformed data from causing issues or exploits (like SQL injection, which in ORMs is usually handled, but if constructing raw queries, be careful to parameterize queries rather than string format them).
•	Secrets in Memory: Be aware that once a secret is in a Python string, it might linger in memory. Python doesn't easily allow wiping memory like in lower-level languages, but just avoid needless copying or printing of secret values. If particularly concerned (very high security context), consider whether Python is the right tool or if you need additional precautions (like using key management systems that handle decryption in hardware or such).
•	Example:
 	import os
from pydantic import BaseSettings, SecretStr

class Settings(BaseSettings):
    db_user: str
    db_password: SecretStr  # using Pydantic's SecretStr to avoid printing
    api_key: SecretStr
    class Config:
        env_prefix = "MYAPP_"  # expecting env like MYAPP_DB_USER
settings = Settings()
# Use settings.db_user and settings.db_password.get_secret_value() to actually retrieve the secret when needed.
db_user = settings.db_user
db_password = settings.db_password.get_secret_value()
 	This design pulls from env vars MYAPP_DB_USER and MYAPP_DB_PASSWORD. We never hardcode those in code. If we do print(settings.db_password), it will show SecretStr('**********') instead of the real value, protecting against accidental log/print[22].
•	Sanitize Outputs: If your program outputs to console or error messages, ensure it’s not leaking something like a stack trace with secrets. For web APIs, you might want to catch unhandled exceptions and return a generic error instead of a full traceback (FastAPI does this by default in debug vs production mode).
•	Regular Audits: Periodically review the code for any potential secrets or security issues. Use code scanning tools (linters like Bandit, which Ruff can include via the S rules[41], or others) to catch hardcoded passwords, weak cryptography usage, etc.
Dependency Injection and Configuration Design
•	Dependency Injection (DI): Write functions and classes that accept their dependencies as parameters, rather than hard-coding global accesses. This makes code more testable and flexible. For example, if a function needs to send an email, don’t have it directly call a global SMTP client; instead, pass an email sender object into the function. This way you can swap out the real sender for a mock in tests, or replace implementations if needed. Use abstract base classes or Protocols to define what the dependency must do, and code against those interfaces[42]. This decouples components.
•	No Singletons/Globals for State: Avoid using module-level global state that many parts of the program modify (like a global CONFIG dict that everyone reads/writes). Instead, encapsulate state in classes or pass it through functions. If you need a truly global config or object (e.g., a db connection pool), at least treat it as read-only after initialization, and consider using a well-controlled singleton or a dependency injection container in a structured way. Excessive singletons can make testing harder and lead to hidden couplings[43].
•	Configuration Structure: Design a configuration class or structure that holds all config options (database URLs, API keys, feature flags, etc.). Load this at program startup (from env or config files) and then pass the config (or relevant parts of it) to the components that need it. This is a form of DI – the config is injected rather than modules pulling from environment all over. It centralizes config management. For example, have a settings.py that loads env vars and constructs a Settings object. Other modules import Settings from there, or (better) you pass the relevant portion to whatever needs it.
•	Functional Core, Imperative Shell: A pattern to consider: keep business logic in pure functions or methods that don’t do I/O or depend on external state, and have the outer layer handle I/O and pass in everything needed. For example, have a function process_order(order, payment_gateway) where payment_gateway is an interface. The outer code reads config, instantiates a StripeGateway or PayPalGateway implementing that interface, then calls process_order. The core doesn’t itself read config or instantiate the gateway; it’s injected. This separation makes the core easier to unit test (you can pass a fake gateway) and understand in isolation.
•	Using DI frameworks: In larger apps, you might use a DI framework or container (there are libraries for Python, or frameworks like FastAPI have DI in terms of their dependency system). Use them if they add clarity. FastAPI’s Depends is a sort of DI system for request handling. In simpler cases, manual DI (just passing parameters) is fine.
•	Immutable config: Mark configuration as immutable once loaded (you can use dataclasses with frozen=True or just avoid modifying config attributes at runtime). This prevents accidental changes and ensures all parts of the program see consistent config. If you need dynamic config changes, handle that carefully in a controlled manner.
•	Testing and DI: With DI, you can easily inject fakes/mocks in tests. E.g., if a function uses a db_client parameter, in tests you can pass a fake object with the same interface that records calls. This is much easier than monkeypatching a global db client inside the function. So design with that in mind.
•	Separation of Concerns: Don’t mix config parsing with business logic. Do config loading in one place (e.g., at app startup). Within your functions, assume config is already provided. E.g., don’t have deep inside a function something like api_key = os.environ["API_KEY"]; instead, read that once in config, and pass api_key into whatever needs it. This makes the function more reusable (maybe in another context with a different key, etc.).
•	Example:
 	class DataStore(Protocol):
    def save_record(self, data: dict) -> None:
        ...

class FileDataStore:
    def __init__(self, filename: str):
        self.filename = filename
    def save_record(self, data: dict) -> None:
        with open(self.filename, "a") as f:
            json.dump(data, f)
            f.write("\n")

class InMemoryDataStore:
    def __init__(self):
        self.records = []
    def save_record(self, data: dict) -> None:
        self.records.append(data)

def process_item(item: dict, store: DataStore) -> None:
    # ... do processing
    store.save_record(item)

# Usage:
store = FileDataStore("/var/app/data.json") if not DEBUG else InMemoryDataStore()
process_item({"id": 1, "value": 42}, store)
 	In this design, process_item does not need to know how/where data is saved. We can inject a different DataStore (maybe a database-backed one) without changing process_item. Testing process_item is easy with InMemoryDataStore or even a dummy that records calls. This follows DI principles (depending on an abstraction DataStore rather than a concrete file or DB).
•	Configuration Example:
 	class AppConfig:
    def __init__(self):
        self.debug = bool(int(os.getenv("DEBUG", "0")))
        self.db_url = os.getenv("DB_URL")
        self.api_key = os.getenv("API_KEY")
        if not self.db_url or not self.api_key:
            raise RuntimeError("Missing critical configuration!")
config = AppConfig()

# Pass config or parts of it to where needed:
db = Database(config.db_url)
client = APIClient(api_key=config.api_key)
 	Here we load everything at once in AppConfig. If something is missing, we fail early (fail fast). Then we inject those values into our Database and API client objects. The rest of the codebase doesn’t call os.getenv at all – it just gets what it needs from config or is passed the ready-to-use objects (like db or client). This makes it clear and testable (for tests, you could instantiate AppConfig with test env vars or monkeypatch AppConfig to not actually require env).
•	Global Config Access: Some projects use a global config object accessible everywhere. That is convenient but can be abused. It’s okay to have a module config.py that holds global settings as long as it’s only set once. But be careful: if you import config at module import time and config isn’t loaded yet, you get default values. To avoid order issues, you might initialize config at program start and then modules use it. A safer approach is passing config into functions explicitly, which is more verbose but clearer. Choose based on project size and complexity.
Testing and Quality Assurance
•	Pytest: Use pytest as the testing framework (it’s the most popular and feature-rich). Write tests for all new code. Aim for good coverage (ideally 90%+, but coverage isn’t everything – focus on critical paths). Organize tests in a separate tests/ directory or alongside modules in a tests sub-package, as long as they are easy to find and run. Use meaningful test function names (e.g., test_login_success).
•	Test Structure: Follow AAA (Arrange-Act-Assert) in tests:
•	Arrange: set up the conditions (input data, objects, perhaps using fixtures).
•	Act: execute the function or method being tested.
•	Assert: check the outcome (return value, side effects, exceptions).
Keep tests focused: test one thing at a time (one behavior). It’s better to have many small tests than one giant test that verifies everything.
•	Fixtures for Reusability: Utilize pytest fixtures for common setup/teardown tasks (like establishing a test database, or preparing test data). For example, a tmp_path fixture is built-in for file system work. You can also parametrize tests to run the same logic with different inputs.
•	Mocks and Monkeypatch: Use unittest.mock or pytest-mock to replace external calls with fakes during tests. For instance, if a function calls an external API, in tests you can monkeypatch that call to return a predefined response (to not actually call the external service). But use mocking sparingly – if you designed your code with dependency injection, you might not need a lot of mocks because you can inject a fake. When using mocks, assert that they were called with expected arguments, etc., to ensure the code under test is interacting correctly. Pytest’s monkeypatch fixture can set environment variables or replace functions easily.
•	Test for Exceptions: Ensure that your code raises the expected exceptions for bad inputs or error conditions. Pytest makes this easy with with pytest.raises(ExpectedError): .... Also, test that no unexpected exceptions occur for normal cases (if they do, the test will fail anyway).
•	Isolate Tests: Tests should not rely on or alter global state in a way that affects other tests. Use fixtures that provide fresh state. For example, if using a database, use a transaction per test that rolls back, or use a fresh in-memory database for each test. If writing to the filesystem, use a temp directory (pytest’s tmp_path). This allows tests to run in any order and not conflict.
•	Fast Tests: Keep unit tests fast (ideally each test under a few milliseconds). Integration tests (like testing the whole app or database integration) might be slower, but try to mark them separately (with pytest markers) so you can run quick unit tests during development and slower ones in CI or on demand. If a test is slow (maybe it calls an external API or does heavy computation), consider if you can mock the slow part or if it belongs in a separate test suite.
•	Coverage: Use coverage tools. For example, run pytest --cov=your_package --cov-report=term:skip-covered to see coverage. Aim to cover critical modules thoroughly. If some auto-generated code or trivial getters are not covered, that’s fine, but business logic should be. Coverage helps find untested parts, but don’t chase 100% just for the number – ensure meaningful tests.
•	Continuous Integration: Integrate tests in CI (GitHub Actions or others) so that every commit/PR runs the test suite. Include linting and type checking in CI as well. Fail the build if tests or linters fail.
•	Regression Tests: When bugs are found, write a test that reproduces the bug (a failing test), then fix the bug so the test passes. This prevents the bug from creeping back later.
•	Test Naming and Clarity: Name test functions clearly to indicate what's being tested. Use given/when/then comments if it helps readability. For example:
 	def test_divide_by_zero_raises_error():
    # Given
    calculator = Calculator()
    # When / Then
    with pytest.raises(ZeroDivisionError):
        calculator.divide(1, 0)
 	This is clear on the intention.
•	Integration and E2E Tests: Besides unit tests, consider integration tests that test multiple components together (e.g., spin up a test database and test that the data access layer works against it). Also end-to-end tests if applicable (for a web app, you might use TestClient from FastAPI to simulate HTTP requests to your routes[44]). These tests ensure the system works as a whole. They can be fewer than unit tests but are important for catching misconfigurations or issues in wiring components together.
•	Use Mocks Appropriately: For example, if your code sends emails via an SMTP server, in tests don’t actually send emails. Use a mock SMTP server or monkeypatch the send function to just record that it was “sent”. Similarly, for external APIs, use responses library or requests-mock to simulate HTTP responses.
•	Randomness and Time: If your code uses random numbers or current time, make tests deterministic by seeding the random or mocking the time (e.g., monkeypatch datetime.now to return a fixed value). This prevents flaky tests that sometimes fail. For random, random.seed(42) in tests or use the faker library with a seed.
•	Parallel Testing: If using pytest-xdist for parallel test execution, ensure tests don’t conflict over shared resources. Use temporary resources or mark tests that can’t run in parallel.
•	Sample Test Execution:

 	$ pytest
=================== test session starts ====================
platform linux -- Python 3.12, pytest-7.x
collected 25 items

tests/test_api.py ........                             [ 32%]
tests/test_core.py .......                             [ 60%]
tests/test_utils.py ............                       [100%]

=================== 25 passed in 0.50s ====================
 	This output shows all tests passed. If any fail, the CI should flag it.
•	Lint and Type Check in Tests: Apply the same linting/type rules to test code. Tests are code too and benefit from clarity. Use type hints in tests where helpful (though many omit them for brevity).
•	Review Test Coverage for Important Paths: For critical functionalities (e.g., security features, payment processing), ensure multiple tests cover the normal case, edge cases, and failure cases.
Code Readability and Maintainability
•	Naming Conventions: Follow PEP 8 naming: Functions and variables in lowercase_with_underscores[45], classes in CapWords (PascalCase)[46], constants (module-level) in UPPER_SNAKE_CASE. Choose descriptive names that convey intent, but not overly verbose. For loop indices or very short-lived variables, short names like i, n are fine. But for anything used in a broader scope, prefer a meaningful name (row_index instead of i if it’s used a lot). Avoid ambiguous names (data, info are not very clear). Include units or domain in name if relevant (e.g., timeout_sec if a number is in seconds). Be consistent (if you have a function get_user, don’t sometimes call similar one fetchOrder with different verb style – use consistent verb tense and style).
•	Function Length and Complexity: Keep functions and methods reasonably short and focused. There’s no hard rule, but if a function exceeds ~50 lines or tries to do too many things, consider refactoring. Long functions with many branches are hard to follow. Cyclomatic complexity (number of independent paths) should be kept low, ideally under 10 per function[47]. If you have deeply nested if/else or lots of conditions, refactor using early returns, helper functions, or simpler logic. Often, breaking a big function into smaller ones (each handling one piece of logic) improves readability and testability[48]. As a sign: if you find it hard to name a function because it’s doing many things, it likely should be split.
•	Comments and Docstrings: Use docstrings for modules, public classes, and functions to explain their purpose and usage. The first line should be a concise summary. For complex functions, include a longer description and maybe examples in the docstring. This helps users of the code and future maintainers. For internal helper functions, a short comment may suffice if the purpose isn’t obvious from the name. Write comments to explain why something is done, if it’s not obvious. Do not write comments that just restate the code[49] (e.g., x = x + 1 # increment x is pointless[49]). Instead, comments should add context or rationale (e.g., # Using bubble sort here because the data size is small and it avoids extra memory – explaining why a presumably non-optimal choice was made, etc.). If code is tricky or non-intuitive, write a comment or simplify the code.
•	Keep Comments Updated: A wrong/outdated comment is worse than none[50]. If you refactor code, update or remove any comments that no longer apply. Treat comments as code that needs maintenance. A good practice is to run a linter for commented-out code or outdated comments (some tools can detect obviously mismatched comments).
•	Blank Lines and Spacing: Use blank lines to separate logical sections of code within a function. For example, a block of setup, a block of computation, a block of result handling – separate them by a blank line. This chunking helps readability. Follow PEP8: two blank lines before top-level definitions (functions, classes)[51], one blank line between methods in a class, etc. Our formatter will handle some of this (Black enforces two lines between top-level defs by default). Indent consistently (never mix tabs and spaces; use 4 spaces).
•	Avoid Long Expressions: Break down complex expressions into intermediate variables with meaningful names. For example, instead of:
 	if (user.age < 18 and user.country in EU_COUNTRIES) or (user.age < 21 and user.country == "USA"):
    ...
 	You could do:
 	underage_in_eu = user.age < 18 and user.country in EU_COUNTRIES
underage_in_us = user.age < 21 and user.country == "USA"
if underage_in_eu or underage_in_us:
    ...
 	This makes it clear what the conditions represent. It’s easier to read and perhaps reuse parts.
•	DRY (Don’t Repeat Yourself): If you find duplicate or very similar code, consider refactoring into a common function or utility. But also avoid premature abstraction – if two pieces of code are similar but not quite, sometimes forcing them into one function can make it more complex. Use judgment, but generally reduce copy-paste coding, as it makes maintenance harder (bug fixes might need to be applied in multiple places).
•	Module Organization: If a module is long, use sections separated by comments or simply logical grouping. For example:
 	# --- Utility Functions ---
def foo(): ...
def bar(): ...

# --- Core Classes ---
class MainEngine: ...
 	This can help navigate. But don’t overdo – ideally the module should maybe be split if it needs sections.
•	Avoid Deep Nesting: Too many nested if/for/while levels hurt readability. Refactor using early returns/continues or helper functions. For example, instead of:
 	def process(items):
    for item in items:
        if item.is_valid:
            # 10 indented lines here
            ...
 	do:
 	def process(items):
    for item in items:
        if not item.is_valid:
            continue
        ... # 10 lines at less indent
 	Or factor out inner logic into a function. Aim for not more than 3 indent levels if possible.
•	Avoid Magic Numbers/Strings: If you have a special number or string (like a status code, or a specific file path, etc.), give it a name. E.g., MAX_RETRIES = 3 at top of module, then use MAX_RETRIES in code. This makes it clear what the number represents and easier to change. Same for strings that have meaning (like "ADMIN_ROLE", better define as constant ADMIN_ROLE = "admin").
•	Use Enumerations or Literals for Choices: If a variable can only have a few specific values, define an Enum or use Literal in type hints. This makes the code self-documenting and helps avoid invalid values. For instance:
 	class State(Enum):
    PENDING = "pending"
    RUNNING = "running"
    DONE = "done"
state: State = State.PENDING
 	This is clearer than state being just a string that should be one of "pending"/"running"/"done".
•	Refactoring: Regularly refactor code that has grown unwieldy. Don’t be afraid to split a function or rename a variable during a review if it improves clarity – small refactoring can pay off in reducing technical debt.
•	Documentation: Maintain project documentation (even if just a markdown file) for tricky areas or overall architecture. But also inline documentation (docstrings and comments) should ensure that someone reading the code can understand it without constantly referring to external docs.
•	Example of readable code:
 	def calculate_discount(user: User, price: float) -> float:
    """Calculate discount for a user based on their membership level."""
    base_discount = 0.0
    if user.membership == Membership.GOLD:
        base_discount = 0.15
    elif user.membership == Membership.SILVER:
        base_discount = 0.10

    # Loyal customers (over 5 years) get an extra 5%
    loyalty_bonus = 0.05 if user.loyalty_years > 5 else 0.0

    discount_rate = base_discount + loyalty_bonus
    # cap the discount rate at 0.20 (20%)
    discount_rate = min(discount_rate, 0.20)
    return price * (1 - discount_rate)
 	This function is concise and clear: it uses a docstring to explain purpose, meaningful variable names (base_discount, loyalty_bonus), a comment to explain a rule, and breaks down the logic into readable parts. It avoids magic numbers by explaining the 5% as loyalty bonus and caps the value with an explanation.
•	Review and Iterate: Always review your code (or have it reviewed by peers) with readability in mind. If someone says “I don’t understand this part”, that’s a cue to improve naming or add a comment. Remember that code is read more often than it’s written – optimize for readability.
Review Checklist
Before considering a Python task done, go through this checklist:
•	[ ] Python Version: Code is running on Python ≥ 3.10 (and using modern syntax/features appropriately). If upgrading from older Python, any deprecated syntax is removed.
•	[ ] Type Hints: All functions and methods have type annotations for parameters and return types. Complex data structures use typing annotations (Dict[], List[], TypedDict, etc.). No # type: ignore comments are present (or only in exceptional cases with justification). Code passes mypy --strict and pyright with 0 errors[2][5].
•	[ ] Linting & Formatting: Code is formatted with Black (120 line length, double quotes normalized[7]). Ruff (or flake8/pylint) reports no warnings or errors – i.e., no unused imports/vars, proper import order[11], no style violations. Imports are grouped and sorted correctly. Line length is kept within 120 chars (no linter complaints about E501). The codebase is consistent in style (no mixed quote styles, etc.).
•	[ ] Logging: Logging is used instead of prints for tracing execution. Log statements are at appropriate levels and include relevant context. No sensitive information is logged (API keys, passwords, personal data)[21]. Logger configuration is present (or in the framework settings) so that logs will actually be captured. Exceptions are logged with tracebacks where they are caught and handled[23].
•	[ ] Security: No secrets (credentials, tokens) are hardcoded in code or config files in VCS[38]. All secrets/configurations are loaded from secure sources (env vars, vault, etc.). Environment variables usage follows best practices (not misused or exposed). Inputs are validated to prevent injection or misuse. Cryptography (if any) is done using standard libraries and strong algorithms. The principle of least privilege is followed for external integrations.
•	[ ] Async Non-blocking: In async code, no blocking calls are made (check that any I/O or sleep calls in async def are awaited, and long CPU tasks offloaded)[32]. FastAPI or async web routes use async def only when appropriate; sync routes are used for sync operations to avoid event loop blocking[33]. Shared resources (db engines, clients) are reused and not recreated per request unnecessarily.
•	[ ] Exception Handling: Exceptions are used for error cases and not suppressed. There are no broad except Exception: pass blocks[52]. All caught exceptions are either rethrown (possibly wrapped) or handled in a way that doesn’t just fail silently. Exception messages are clear. raise ... from ... is used when rethrowing to maintain context[35]. Higher-level layers have catch-alls to prevent crashes (e.g., an API returns an error response instead of crashing the server). No critical exception is left uncaught at the top level (unless we want the program to crash for some reason).
•	[ ] Resource Management: Files, network connections, and other resources are properly closed after use (via with context managers or finally blocks). No file descriptors or connections are left hanging. Memory usage is appropriate (no obviously huge objects kept alive unintentionally).
•	[ ] Dependency Injection: External interactions (DB, APIs, etc.) are abstracted behind interfaces or passed in so that the core logic is decoupled. This is verified by seeing that functions take in things like session or client parameters rather than doing global lookups. Config is loaded in one place and not scattered. There is a clear separation between configuration and use of that config throughout.
•	[ ] Tests: There are unit tests covering new code or changes. Tests cover happy path and edge cases (including error conditions). Run pytest and ensure all tests pass. Coverage is at acceptable level (target ~90%+, but at least all critical paths are tested). No tests are flaky or depend on external systems (or if they do, they are properly marked/skipped unless those systems are available). If applicable, linters and type checkers are run on tests as well and pass.
•	[ ] Performance: There are no obvious performance killers (like an O(n^2) loop on large data without need, or repeatedly calling a slow function in a loop that could be called once, etc.). Use of caches or efficient algorithms is considered where relevant. If any part of code is performance-sensitive, maybe there's a benchmark or at least a reasoning that it's efficient enough for expected input sizes.
•	[ ] Cyclomatic Complexity: No single function is overly complex (e.g., none has dozens of branches or extremely deeply nested logic). If there is a complex function, consider if it can be simplified or broken down. Perhaps run a complexity tool (like Radon) and ensure no function has a crazy high complexity; aim for under 10-15. If above, justify and add comments or refactor.
•	[ ] Code Clarity: Read through the code (or have someone else review) and ensure it’s understandable. All identifiers (variables, functions, classes) clearly indicate their purpose. There are no misleading names. Comments are present where needed and correct[50]. Remove any commented-out code that is not needed (don’t leave large chunks of dead code; use version control to retrieve old code if needed).
•	[ ] Compliance with PEP 8: The code generally follows PEP 8 guidelines (our tools enforce a lot of it). Naming conventions are followed (checked above)[45][46], spacing and indent as well. No trailing whitespaces, etc. (this is usually handled by formatter).
•	[ ] Documentation: Ensure public APIs (if this is a library or module) have docstrings explaining usage. If this is a script or app, ensure there's usage info (in README or help message). Update any relevant docs or README sections if behavior changed.
By following this comprehensive skill, the Python code produced or reviewed will be robust, maintainable, and aligned with best practices. Always keep learning and updating these guidelines as Python evolves (e.g., new typing features, new linters) to stay current with the state of the art.
________________________________________
[1] [2] [6] Mypy Documentation
https://mypy.readthedocs.io/_/downloads/en/latest/pdf/
[3] Pyright: Static Type Checker for Python | Hacker News
https://news.ycombinator.com/item?id=34222407
[4] [5] [28] [31] [42] [43] Mypy Best Practices and Coding Standards | Cursor Rules Guide | cursorrules
https://cursorrules.org/article/mypy-cursor-mdc-file
[7] The Black code style - Black 26.1.0 documentation
https://black.readthedocs.io/en/stable/the_black_code_style/current_style.html
[8] [9] [10] [12] [41] Ruff Tutorial: A Complete Guide for Python Developers | by MA Raza, Ph.D. | Medium
https://medium.com/@amjadraza24/ruff-tutorial-a-complete-guide-for-python-developers-1aa62272596d
[11] [14] [15] [16] [29] [30] [45] [46] [49] [50] [51] PEP 8 – Style Guide for Python Code | peps.python.org
https://peps.python.org/pep-0008/
[13] Please respect the established Python coding style, as exemplified ...
https://github.com/psf/black/issues/1252
[17] [18] [19] [24] [25] Logging Best Practices — structlog 25.5.0 documentation
https://www.structlog.org/en/stable/logging-best-practices.html
[20] [21] [39] Storing secrets in env vars considered harmful
https://blog.arcjet.com/storing-secrets-in-env-vars-considered-harmful/
[22] [38] [40] Avoiding Secret Leaks in Python Projects | ArjanCodes
https://arjancodes.com/blog/how-to-prevent-secrets-leakage-in-your-python-projects/
[23] [26] [27] [35] [36] [44] [52] Exception Handling Best Practices in Python: A FastAPI Perspective | by Hyunil Kim | 딜리버스 | Medium
https://medium.com/delivus/exception-handling-best-practices-in-python-a-fastapi-perspective-98ede2256870
[32] Top 7 FastAPI asyncio Best Practices for Non‑Blocking Web APIs
https://www.techbuddies.io/2026/01/05/top-7-fastapi-asyncio-best-practices-for-non-blocking-web-apis/
[33] [34] Concurrency and async / await - FastAPI
https://fastapi.tiangolo.com/async/
[37] Python File Handling & Context Managers — Working with Files
https://medium.com/@ghoshsiddharth25/python-file-handling-context-managers-working-with-files-cf4c67e153ab
[47] [48] Cyclomatic Complexity Illustrated · seeinglogic blog
https://seeinglogic.com/posts/cyclomatic-complexity-illustrated/
