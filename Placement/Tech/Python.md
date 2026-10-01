# Python for Applied AI — Java/JS Developer Notes

<aside>
🎯

This page is a **Java/JavaScript developer’s bridge into practical Python for backend and Applied AI work**. It is not a beginner course and it is not meant to be memorized line by line. The goal is to internalize the Python concepts that will keep appearing later when you call LLM APIs, parse JSON, handle files, build retrieval pipelines, work with embeddings, and glue AI components together.

</aside>

# 1. The Python mindset that matters

Python rewards code that is explicit, readable, and direct. Compared with Java, you will usually write less ceremony around the same idea: functions can exist without classes, built-in collections are powerful, and many common operations are expressed through compact language features rather than large utility APIs.

This does **not** mean “shorter is always better.” Pythonic code should make the intent obvious. A dense one-liner that hides logic is usually worse than a clear loop. For AI work, this matters because most application code is orchestration: load input, validate it, transform it, call a model or service, parse the result, and handle failures.

## Indentation defines structure

Python uses indentation instead of braces to define blocks. A colon introduces a block and indentation determines where that block ends.

```python
def classify_score(score):
    if score >= 0.8:
        return "high"
    elif score >= 0.5:
        return "medium"
    return "low"
```

In Java, indentation is mostly presentation because braces define scope. In Python, indentation is syntax. Four spaces is the conventional standard.

**AI use case:** prompt-processing and evaluation code often contains nested conditionals for validation, retry rules, or routing. Clear indentation makes these flows easy to inspect.

---

# 2. Python is dynamically typed, but strongly typed

Python does not require Java-style variable declarations. A name refers to an object, and the object's type is checked at runtime.

```python
count = 10
count = "ten"  # legal assignment

result = 40 + "2"  # TypeError at runtime
```

The second line is allowed because variables are not permanently bound to one declared type. The final line fails because Python will not silently combine incompatible values.

This distinction is important:

- **Dynamic typing** means type checks happen at runtime rather than compile time.
- **Strong typing** means incompatible operations are not silently coerced into something else.

## Type hints are still extremely useful

Python lets you annotate expected types even though the runtime does not enforce them automatically.

```python
def build_prompt(question: str, context: list[str]) -> str:
    joined = "\n".join(context)
    return f"Context:\n{joined}\n\nQuestion: {question}"
```

Type hints improve editor support, readability, static analysis, and maintainability. They become particularly valuable in larger AI systems where data moves through many steps: parser → chunker → retriever → model call → evaluator.

<aside>
🧠

Treat type hints as documentation that can be checked by tools. They do not turn Python into Java, but they make larger Python codebases far easier to reason about.

</aside>

---

# 3. REPL: use Python interactively

Python ships with a REPL, which lets you evaluate expressions immediately. It is useful for testing syntax, checking the behavior of a library, or inspecting a small transformation before putting it into application code.

```python
>>> text = "hello world"
>>> text.split()
['hello', 'world']
>>> len(text.split())
2
```

**AI use case:** while building a retrieval or preprocessing pipeline, the REPL is excellent for testing one regex, one JSON transformation, one chunking rule, or one API response shape without rerunning the entire application.

---

# 4. Control flow: familiar ideas, more iterable-oriented syntax

Python's `if`, `while`, `break`, and `continue` will feel familiar. The largest difference from Java is the style of the `for` loop.

## Iterate over values directly

```python
documents = ["a.txt", "b.txt", "c.txt"]

for document in documents:
    print(document)
```

Use `range()` when you genuinely need numbers:

```python
for i in range(5):
    print(i)
```

Use `enumerate()` when you need both the value and its position:

```python
chunks = ["first", "second", "third"]

for index, chunk in enumerate(chunks):
    print(index, chunk)
```

**AI use case:** `enumerate()` is useful when assigning chunk IDs, tracking prompt examples, or attaching positions to retrieved passages.

## `elif` instead of Java-style `else if`

```python
if score > 0.9:
    label = "excellent"
elif score > 0.7:
    label = "good"
else:
    label = "weak"
```

Python also has `match/case`, but it is structural pattern matching rather than a direct replacement for Java's classic `switch`. You do not need it for the current stage.

---

# 5. Numbers: the Java differences worth remembering

Python has `int`, `float`, and `complex` built in. For general backend and AI application work, `int` and `float` are the important ones.

## Python integers are arbitrary precision

You do not normally choose between `int`, `long`, and `BigInteger`. Python integers grow as needed, limited mainly by available memory.

```python
huge = 10 ** 100
print(huge)
```

This removes a common Java concern such as accumulator overflow, although memory and performance still matter.

## `/` and `//` are different

```python
print(11 / 4)   # 2.75
print(11 // 4)  # 2
```

`/` performs true division and returns a floating-point result. `//` performs floor division.

This is easy to forget when switching from Java.

## Exponentiation

```python
squared = 7 ** 2
```

Python uses `**` for exponentiation.

---

# 6. Lists: your default mutable sequence

A Python `list` plays a role similar to Java's `ArrayList`, but the syntax and built-in operations are much lighter.

```python
scores = [0.72, 0.91, 0.63]

scores.append(0.88)
last = scores[-1]
first_two = scores[:2]
```

Lists are mutable, indexed from zero, and support negative indexing.

## Slicing

The general form is:

```python
sequence[start:stop:step]
```

Examples:

```python
items = [10, 20, 30, 40, 50]

items[1:4]   # [20, 30, 40]
items[:3]    # [10, 20, 30]
items[::2]   # [10, 30, 50]
items[::-1]  # reversed copy
```

**AI use case:** slicing appears constantly when batching records, selecting prompt examples, taking top-k retrieval results, or examining only part of a token/chunk list.

---

# 7. Tuples: small immutable groupings

A tuple is an immutable sequence. It is useful when several values naturally belong together and should not be modified accidentally.

```python
result = ("doc_17", 0.87)
doc_id, score = result
```

This unpacking style is extremely common in Python.

A one-element tuple needs a trailing comma:

```python
single = (3,)
```

**AI use case:** retrieval code often works naturally with pairs such as `(document, similarity_score)` or `(chunk_id, metadata)`.

---

# 8. Dictionaries: the workhorse of API and AI application code

A `dict` is Python's built-in mapping type. It will feel similar to a Java map, but dictionaries are used far more pervasively because JSON objects map naturally into them.

```python
user = {
    "id": 42,
    "name": "Asha",
    "active": True
}

print(user["name"])
user["role"] = "admin"
```

Keys must be hashable, which usually means they should behave immutably.

## Safe lookup with `get()`

Square-bracket lookup is appropriate when a key is required, but it raises `KeyError` if that key is missing. `dict.get()` is useful when a field is optional because it returns `None` instead of crashing, or a fallback value that you provide.

```python
temperature = payload.get("temperature", 0.7)
city = payload.get("city")  # None if the key is absent
```

This becomes especially useful with nested API or JSON data. A safe lookup can provide an empty dictionary for a missing outer object and then continue inward:

```python
city = payload.get("address", {}).get("city")
```

If `address` is missing, `{}` is used and the second `get()` simply returns `None`. One subtle case remains: if `address` exists but its value is explicitly `None`, calling `.get("city")` on that `None` value would still fail.

## Iterating through dictionaries

```python
for key, value in user.items():
    print(key, value)
```

**AI use case:** model API responses, structured outputs, retrieval metadata, evaluation results, configuration, and JSON files are all commonly represented as dictionaries.

---

# 9. Sets: fast membership and deduplication

A set stores unique hashable values and provides fast membership checks.

```python
seen_ids = {"a1", "a2"}

if "a3" not in seen_ids:
    seen_ids.add("a3")
```

An empty set is created with `set()`, not `{}`, because `{}` creates an empty dictionary.

**AI use case:** sets are useful for deduplicating document IDs, preventing repeated URLs, tracking already-processed chunks, or checking whether a label belongs to an allowed set.

---

# 10. Comprehensions: concise collection transformations

Python comprehensions are one of the most useful differences from Java. They provide a compact way to transform or filter collections.

## List comprehension

```python
scores = [0.31, 0.82, 0.91, 0.55]

high_scores = [score for score in scores if score >= 0.8]
```

Equivalent explicit loop:

```python
high_scores = []

for score in scores:
    if score >= 0.8:
        high_scores.append(score)
```

## Dictionary comprehension

```python
documents = ["a.txt", "b.txt", "c.txt"]

document_ids = {
    name: index
    for index, name in enumerate(documents)
}
```

## Set comprehension

```python
words = ["AI", "ai", "Python", "python"]

normalized = {word.lower() for word in words}
```

**AI use case:** comprehensions are excellent for preprocessing model inputs, cleaning retrieved records, extracting fields from JSON, normalizing labels, or building metadata maps.

<aside>
⚠️

Use comprehensions when the transformation remains easy to read. If the logic needs several branches, side effects, or nested reasoning, an ordinary loop is usually clearer.

</aside>

---

# 11. Strings and f-strings

Python strings are immutable Unicode sequences. Single and double quotes are both ordinary string delimiters.

```python
name = "Riya"
model = 'gpt-example'
```

## Prefer f-strings for readable formatting

```python
question = "What is caching?"
context_count = 4

message = f"Answer '{question}' using {context_count} context passages."
```

**AI use case:** prompts, logs, filenames, debug messages, and formatted model instructions frequently use f-strings.

## Useful string operations

```python
text = "  System Design Notes  "

clean = text.strip()
words = clean.split()
slug = "-".join(word.lower() for word in words)
```

These small operations are common in ingestion and preprocessing pipelines.

---

# 12. Booleans, truthiness, and Pythonic conditions

Python uses `and`, `or`, and `not` instead of Java's `&&`, `||`, and `!`.

```python
if authenticated and not blocked:
    process_request()
```

Python also gives every object a truthy or falsy interpretation. Empty strings, empty collections, zero, and `None` are falsy.

```python
chunks = []

if chunks:
    print("There is data to process")
else:
    print("Nothing to process")
```

This is more idiomatic than writing `len(chunks) != 0`.

---

# 13. `None`: Python's null-like singleton

`None` represents the absence of a value. Functions that do not explicitly return something return `None`.

When checking specifically for `None`, use identity:

```python
if result is None:
    print("No result")

if result is not None:
    print(result)
```

Do not rely on general falsiness when you specifically mean “missing,” because values such as `0`, `""`, and `[]` are also falsy.

**AI use case:** optional API fields, missing retrieval results, absent configuration, or an unsuccessful parsing step may legitimately return `None`.

---

# 14. Useful containers from `collections`

Python's standard library includes several specialized containers that are worth recognizing.

## `deque`

A double-ended queue supports efficient insertion and removal from both ends.

```python
from collections import deque

queue = deque(["job1", "job2"])
queue.append("job3")
next_job = queue.popleft()
```

Useful for BFS, work queues, sliding-window logic, and buffering.

## `Counter`

```python
from collections import Counter

labels = ["positive", "negative", "positive", "neutral"]
counts = Counter(labels)
```

Useful for label distributions, token/category counts, and quick frequency analysis.

## `defaultdict`

```python
from collections import defaultdict

groups = defaultdict(list)

groups["python"].append("doc1")
groups["python"].append("doc2")
```

Useful when grouping documents, passages, users, or evaluation results without repeatedly checking whether a key exists.

You should know these exist, but do not turn this block into a tour of every class in `collections`.

---

# 15. Virtual environments and packages: keep each project self-contained

As soon as a Python project starts using external libraries, dependency management matters. Installing every package into the system Python works at first, but different projects may eventually need different versions of the same library. A **virtual environment** gives one project its own Python environment and its own installed packages.

## Create and activate a virtual environment

From the project directory:

```bash
python3 -m venv .venv
```

Here, `-m` means “run this Python module as a program.” In this case, Python runs its built-in `venv` module and creates the environment in a folder named `.venv`.

Creating the environment is not enough; the shell must also be told to use it. On bash/zsh:

```bash
source .venv/bin/activate
```

On Fish:

```
source .venv/bin/activate.fish
```

After activation, commands such as `python3` and `pip` should resolve inside `.venv`. A quick verification is:

```bash
which python3
```

The path should point somewhere inside the project's `.venv/bin/` directory.

## Install packages into that environment

```bash
python3 -m pip install requests
```

Using `python3 -m pip` ties `pip` to the exact Python interpreter being used. That avoids accidentally installing a package into a different Python installation when several versions or environments exist.

You can inspect an installed package with:

```bash
python3 -m pip show requests
```

## Record the project's dependencies

```bash
python3 -m pip freeze > requirements.txt
```

`pip freeze` prints installed packages with exact versions, while `>` writes that output into `requirements.txt`. Another machine can later recreate those dependencies with:

```bash
python3 -m pip install -r requirements.txt
```

When finished with the environment:

```bash
deactivate
```

`deactivate` takes no argument. It simply returns the shell to the normal system environment.

<aside>
🧠

Mental model: **venv isolates the project → pip installs into that isolated environment → requirements.txt records what the project depends on.**

</aside>

---

# 16. Functions, modules, and exceptions: building small reliable programs

Python functions are defined with `def`. Parameters are written directly in the function signature, and `return` sends a value back to the caller. Unlike Java, a function does not need to live inside a class.

## Functions and default arguments

A parameter can have a default value. That default is used only when the caller omits that argument.

```python
def greet(name, prefix="Hello"):
    return f"{prefix}, {name}"

greet("Shahid")             # "Hello, Shahid"
greet("Shahid", "Welcome")  # "Welcome, Shahid"
```

This is useful for behavior that has a sensible normal case but should still be configurable.

## A Python file can be a module

A `.py` file can expose functions to another file. This gives Python a lightweight way to organize code without creating utility classes just for structure.

```python
# prompts.py
def build_prompt(question: str) -> str:
    return f"Answer clearly: {question}"
```

```python
# main.py
from prompts import build_prompt

print(build_prompt("What is a vector database?"))
```

In a real application, separate modules might hold configuration, API clients, prompt construction, document loading, retrieval, or evaluation logic.

## Exceptions: handle expected failures precisely

Code that may fail goes inside `try`. An `except` block handles a specific failure, `else` runs only when the `try` succeeds, and `finally` runs whether the operation succeeds or fails.

```python
raw = "42"

try:
    number = int(raw)
except ValueError:
    print("Input was not a valid integer")
else:
    print(number)
finally:
    print("Conversion attempt finished")
```

Prefer a specific exception such as `ValueError` instead of a bare `except:`. Catching everything can hide unrelated bugs and make failures harder to diagnose.

<aside>
🧠

Mental model: **try the risky operation → catch the failure you actually expect → use else for success-only work → use finally for work that must happen either way.**

</aside>

---

# 17. Files, JSON, and HTTP: moving data across program boundaries

A large part of practical Python is moving data between your program and the outside world. Files persist data locally, JSON provides a common structured text format, and HTTP moves that data between services.

## File I/O with `with open(...)`

`open()` gives you a file object. The mode tells Python what you intend to do with the file:

- `"r"` reads an existing file;
- `"w"` writes to a file, creating it if necessary and replacing existing contents;
- `"a"` appends new data to the end instead of replacing what is already there.

```python
with open("hello.txt", "w") as f:
    f.write("Hello from Python\n")

with open("hello.txt", "r") as f:
    text = f.read()

print(text)
```

The `with` statement is a context manager. When the block ends, Python closes the file automatically even though variables created inside the block can still exist afterward. In other words, `with` manages the file resource; it does **not** create a new variable scope.

## Python objects and JSON are not the same thing

A Python dictionary is already a normal Python object and needs no JSON import:

```python
data = {"name": "Shahid", "age": 22}
print(data["name"])
```

JSON is a text format used to store or exchange structured data. The `json` module is needed when converting between Python objects and JSON text.

The four names are easiest to remember in pairs:

- `json.dumps(obj)` → Python object to a JSON **string**
- `json.loads(text)` → JSON **string** back to a Python object
- `json.dump(obj, file)` → Python object directly into a JSON **file**
- `json.load(file)` → JSON **file** directly back into a Python object

```python
import json

user = {
    "name": "Shahid",
    "address": {
        "city": "Bhubaneswar"
    }
}

with open("user.json", "w") as f:
    json.dump(user, f)

with open("user.json", "r") as f:
    loaded = json.load(f)

city = loaded.get("address", {}).get("city")
print(city)
```

JSON uses `null` for a missing/null value; once parsed into Python, the corresponding value is `None`.

## HTTP requests: talking to an API

The `requests` library lets Python communicate with web services using familiar HTTP methods such as GET and POST.

A GET request is normally used to retrieve data:

```python
import requests

response = requests.get(
    "https://jsonplaceholder.typicode.com/todos/1",
    timeout=10
)
```

`timeout=10` prevents the program from waiting forever if the server never responds.

The returned `response` is an HTTP response object, not just the JSON body. It contains useful information such as:

```python
response.status_code
response.headers
response.text
```

If the body contains JSON, this convenience method parses it into normal Python dictionaries/lists:

```python
data = response.json()
```

Conceptually, `response.json()` is similar to:

```python
import json
data = json.loads(response.text)
```

The difference is that `response.json()` is provided directly by the `requests` response object.

## Sending JSON with POST

When an API expects JSON, pass the Python object with `json=`:

```python
payload = {
    "message": "nice",
    "length": 4
}

response = requests.post(
    "https://api.example.com/items",
    json=payload,
    timeout=10
)
```

`requests` serializes the object to JSON and sets the appropriate content type for you. A GET request normally should not carry a JSON body; if the goal is to send a structured object to the server, POST is usually the more appropriate method.

## Handle HTTP failures before parsing the body

A response can arrive successfully at the network level and still represent an HTTP error. For example, `404` means not found and `405` means the URL does not allow the HTTP method you used.

A clean pattern is:

```python
import requests

try:
    response = requests.get(
        "https://jsonplaceholder.typicode.com/todos/1",
        timeout=10
    )

    response.raise_for_status()
    data = response.json()
    print(data)

except requests.RequestException as e:
    print("HTTP request failed:", e)
```

`raise_for_status()` does nothing for a successful response, but raises an HTTP-related exception for error status codes. Catching `requests.RequestException` is preferable to a bare `except:` because it handles request failures without hiding unrelated programming mistakes.

One final distinction matters: a successful HTTP status does not guarantee that the body is valid JSON. Calling `response.json()` on HTML or other text can still raise a JSON-decoding error.

The larger mental model is:

```
construct request
→ send it
→ inspect / validate the response
→ parse JSON only when appropriate
→ continue the program
```

---

# 18. Bytes: know what they are, do not over-focus on them yet

Strings represent text. `bytes` represents raw binary data.

You will encounter bytes when dealing with files, HTTP bodies, images, PDFs, audio, or encoded payloads.

```python
text = "hello"
raw = text.encode("utf-8")
restored = raw.decode("utf-8")
```

For your current stage, knowing the distinction is enough. Deeper binary manipulation is unnecessary unless a later task requires it.

---

# 19. What to deliberately postpone

The source article contains several interesting topics that are not important for the current transition into Applied AI.

## CPython internals and alternative runtimes

Knowing that CPython compiles source into bytecode and interprets it is useful background, but Jython, IronPython, PyPy, and runtime implementation details will not improve your ability to build LLM applications right now.

## Operator overloading and dunder methods

Methods such as `__len__` and `__add__` let custom classes integrate with Python's operators and built-ins. Recognize what “dunder method” means, but postpone custom operator behavior until classes become relevant.

## Advanced object-oriented Python

You already understand OOP from Java. Python classes, inheritance, decorators, and protocol-style behavior can be learned when a real application requires them. They are not prerequisites for making API calls, parsing data, or building a basic RAG pipeline.

## NumPy, pandas, TensorFlow, Keras, and PyTorch

These are important parts of the broader Python ecosystem, especially for data science and machine learning. They are **not prerequisites for the current Applied AI / backend path**. Learn them when the work specifically demands numerical computing, data analysis, or model training rather than adding them simply because they are associated with Python.

---

# 20. A complete mini example: turning an API response into usable AI context

This small example combines several of the concepts above.

```python
from collections import Counter

response = {
    "documents": [
        {"id": "d1", "text": "Redis is an in-memory data store.", "score": 0.91},
        {"id": "d2", "text": "Kafka is a distributed event platform.", "score": 0.62},
        {"id": "d3", "text": "Redis supports caching.", "score": 0.88},
    ]
}

documents = response.get("documents", [])

high_quality = [
    doc
    for doc in documents
    if doc.get("score", 0) >= 0.8
]

context = "\n".join(
    f"[{doc['id']}] {doc['text']}"
    for doc in high_quality
)

words = [
    word.lower().strip(".,")
    for doc in high_quality
    for word in doc["text"].split()
]

common = Counter(words).most_common(5)

print(context)
print(common)
```

This is deliberately small, but the shape is realistic:

```
structured response
→ dictionary/list access
→ filtering
→ comprehension
→ string construction
→ lightweight analysis
→ prompt-ready context
```

That same shape will reappear later in retrieval and RAG work.

---

# 21. What you should be able to do after this page

You do **not** need to remember every method name. The target is that Python no longer blocks you when the AI work becomes more interesting.

You should be able to:

- read ordinary Python without mentally translating every line into Java;
- write loops, conditionals, and small functions comfortably;
- use default arguments when a parameter has a sensible fallback;
- choose correctly between a list, tuple, set, and dictionary;
- transform collections with readable comprehensions;
- use slicing, unpacking, `range()`, and `enumerate()`;
- handle `None`, truthiness, and optional dictionary keys correctly;
- format strings with f-strings;
- recognize `Counter`, `defaultdict`, and `deque`;
- organize helper functions into modules and import them;
- handle expected failures with specific `try/except/else/finally` logic;
- read and write files safely with `with open(...)`;
- convert between Python objects, JSON strings, and JSON files with `dump/load/dumps/loads`;
- create, activate, verify, and deactivate a virtual environment;
- install packages with `python -m pip` and record them in `requirements.txt`;
- make GET/POST requests, send JSON with `json=`, and use timeouts;
- inspect HTTP responses, parse JSON, and handle non-2xx failures with `raise_for_status()`;
- understand the shape of an HTTP → JSON → transformation pipeline.

The next useful progression is not more syntax for its own sake. It is to make these pieces interact in real application code: virtual environments and packages, HTTP requests, environment/config data, and then an actual LLM API.

---

# Source and scope

Primary source: [Real Python — Java vs Python: Basic Python for Java Developers](https://realpython.com/java-vs-python/)

The notes intentionally extract the parts of that tutorial that matter most for a developer already comfortable with Java/JavaScript. The AI-oriented examples and sequencing are practical additions so the page remains useful beyond the initial syntax transition.

## Reacquisition checkpoint — 30 Aug 2026

The first hands-on syntax drill confirms the intended diagnosis: **Python is rusty, not conceptually new.**

### Strong / already usable

- list and dictionary comprehensions after a short warm-up
- sets and deduplication
- `enumerate()`
- `Counter`
- slicing
- tuple unpacking
- filtering collections
- basic function/control-flow structure

### Surface-syntax misses to normalize through use

- comprehension brackets
- `len(x)` rather than Java/JS-style length methods
- `.lower()`
- `not` rather than `!`
- `defaultdict(list)`
- `dict.get(key, default)`
- occasional missing colon / punctuation

These are lexical-interference errors from switching languages, not evidence of weak programming concepts. Do **not** restart a general Python course.

### 31 Aug — Functions, modules, exceptions: PASS

The follow-up block moved from syntax recall into program structure. Functions and default arguments were usable; a small module could be written and imported from another file; and exception handling was understood through `try/except/else/finally`, with the important rule that expected failures should be caught using a **specific exception** rather than a bare `except:`.

### 1 Sep — Files and JSON: PASS

The next block confirmed practical file and structured-data handling. File reads/writes with `with open(...)` were understood, including the fact that a `with` block closes the file resource without creating a new variable scope. JSON round-tripping was completed in both forms: `dumps/loads` for strings and `dump/load` for files. Optional and nested dictionary fields were handled safely with `.get()`, including the distinction that Python returns `None`, not JSON's `null`.

### 1 Sep — venv, pip, and first HTTP call: PASS

A fresh virtual environment was created and activated, including the Fish-specific activation script. The active interpreter was verified with `which python3`; `requests` was installed inside the venv; and exact dependencies were recorded with `pip freeze > requirements.txt`. A real public JSON endpoint was called successfully, its response was parsed with `response.json()`, and an invalid endpoint was then used to prove non-200 handling with `raise_for_status()`. The main polish point is to prefer specific `requests` exceptions over a bare `except:`.

Current direction: continue to **Python gate → first LLM API call**, repairing only concrete misses as they appear.