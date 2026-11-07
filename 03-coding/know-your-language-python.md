# Know your language: Python internals

[← Coding](README.md) · [Index](../README.md)

"Know your language" is one of Jane Street's four hiring principles, which means they will probe the language you code in, not just the solution. Hudson River Trading's systems loop asked an SRE to explain Python `yield`, generators and decorators alongside shell questions ([Glassdoor](https://www.glassdoor.ca/Interview/Hudson-River-Trading-Interview-E470937-RVW98611866.htm)); Meta's security loop asked how to find a memory leak in a Python script ([block 06](../06-linux/README.md)). If you write the coding round in Python, be ready for any of these as a follow-up to your own code. Each answer below is also a security point, because that is how you turn a language question back toward the job.

---

## Execution model

<details>
<summary><b>What is the GIL, and when do threads help?</b></summary>

The Global Interpreter Lock lets only one thread execute Python bytecode at a time within a process, so CPU-bound work does not speed up with threads. Threads still help for I/O-bound work (network, disk, waiting on a subprocess) because the GIL is released while blocking. For CPU-bound parallelism use `multiprocessing` (separate processes, separate GILs) or push the work into a C extension or NumPy. `asyncio` is single-threaded cooperative concurrency for I/O: great for thousands of network connections, useless for crunching. A free-threaded build of CPython exists as an option in recent versions, but the default interpreter you will be asked about still has the GIL. Security angle: log parsing at scale is I/O-bound reading and CPU-bound regex; stream it with a generator and, if it is slow, profile before reaching for processes.
</details>

<details>
<summary><b>What does `yield` do? Generators versus lists.</b> · [HRT]</summary>

A function with `yield` returns a generator: an iterator that produces one value at a time and keeps its local state between calls, instead of building the whole result in memory. `for line in open(path)` is already lazy; `def failed_logins(lines): for l in lines: if 'fail' in l: yield parse(l)` is a pipeline stage that costs constant memory however big the file. Generator expressions `(f(x) for x in it)` do the same inline. The trade-off: one pass only, no `len()`, no indexing. This is the honest answer to "what breaks at ten million lines" in the coding round: don't load the list, stream it. `itertools` (`islice`, `groupby`, `chain`) are the tools for composing generators.
</details>

<details>
<summary><b>What is a decorator, and what are they for?</b> · [HRT]</summary>

A decorator is a callable that takes a function and returns a new function, applied with `@name`. `@functools.wraps(fn)` inside it preserves the original's name and docstring. Uses you can name from security work: timing and metrics, retry with backoff, caching (`functools.lru_cache`), and authorisation checks (`@require_role("admin")`) that run before the body. Point to make: a decorator that enforces a permission check is only a control if every entry point is decorated; a missed decorator is a missed check, which is why you prefer checks in a middleware or a single chokepoint for anything that matters.
</details>

<details>
<summary><b>Context managers and `with`.</b></summary>

`with` guarantees setup and teardown around a block via `__enter__` and `__exit__` (or `@contextlib.contextmanager` with a `yield`), even if the block raises. Files, locks, database transactions, temporary privilege elevation. Security angle: anything that acquires a sensitive resource (a credential from a vault, a lock, a temp file with secrets) belongs in a context manager so the cleanup cannot be skipped by an exception path.
</details>

---

## Gotchas they like to ask

<details>
<summary><b>Mutable default arguments.</b></summary>

`def add(item, bucket=[])` evaluates the default once at definition time, so every call shares the same list and state leaks between calls. Use `bucket=None` then `if bucket is None: bucket = []`. It is the most common Python bug in interview code; if you write it in the coding round, they will notice.
</details>

<details>
<summary><b>`is` versus `==`.</b></summary>

`==` compares values; `is` compares identity (same object). `is` is for `None`, `True`, `False` and sentinels. Small integers and some strings are cached so `is` can look like it works on them and then fail in production; never use `is` for value comparison.
</details>

<details>
<summary><b>How does a dict work, and why is that a security question?</b></summary>

A hash table: average O(1) insert and lookup, keys must be hashable (immutable). Collisions degrade to a chain scan, and an attacker who can craft many colliding keys (for example as JSON field names) can turn O(1) into O(n²) and stall a service, the hash-flooding denial of service. CPython randomises string hashing per process since 3.3 to make that hard (`PYTHONHASHSEED`), which is also why dict and set iteration order of strings used to vary between runs. Dicts preserve insertion order since 3.7.
</details>

<details>
<summary><b>Exceptions: what not to do.</b></summary>

Bare `except:` and `except Exception: pass` swallow errors, including the ones that mean your security check failed. Catch the specific exception, log it, and re-raise or fail closed. In a detection script, a parsing error that is silently skipped is a log line you never examined. Use `finally` or a context manager for cleanup, not a catch-all.
</details>

<details>
<summary><b>How do you find a memory leak in a Python script?</b> · [Meta]</summary>

`tracemalloc` snapshots at two points in time and `compare_to` shows which lines allocated the growth; `objgraph` shows what is holding references. The usual culprits: an unbounded cache or dict used as a cache, a global list that only grows, closures or logging handlers that keep references, C extensions, and generators you turned into lists. Reference cycles with `__del__` can defeat the cycle collector. Answer with the method, then the usual suspects.
</details>

---

## The security-specific Python knowledge

<details>
<summary><b>`random` versus `secrets`; `subprocess` safely; `pickle`, `yaml.load`, `eval`.</b></summary>

- `random` is a Mersenne Twister, predictable from a few outputs; `secrets` (and `os.urandom`) is the CSPRNG. Tokens, passwords, nonces: `secrets.token_urlsafe()`, never `random`.
- `subprocess.run(["ls", path])` passes arguments as a list, so a filename with `; rm -rf` is just a filename. `shell=True` with string formatting is command injection. If you must use the shell, `shlex.quote`.
- `pickle.loads` on untrusted data executes code; it is not a parser, it is a deserialiser with arbitrary object construction. Same for `yaml.load` without `SafeLoader` (use `yaml.safe_load`), and for `eval`/`exec` on anything that touched user input. The interview phrase: deserialisation of untrusted data is remote code execution with extra steps.
- Compare secrets with `hmac.compare_digest`, not `==`, which short-circuits on the first differing byte and leaks timing.
- Paths: resolve and check containment (`Path(base, user_input).resolve()` and confirm it is still under `base`) before opening, or a `../` walks out of your directory.
</details>

<details>
<summary><b>Type hints, dataclasses, and why they help in security code.</b></summary>

Type hints (`def parse(line: str) -> LoginEvent | None`) plus a checker (mypy, pyright) catch a class of bugs before runtime and make intent legible to a reviewer; `@dataclass(frozen=True)` gives you immutable records for parsed events. Say that this is the Python end of the same instinct that makes Jane Street like OCaml's types: make illegal states hard to represent ([block 18](../18-ocaml-literacy/README.md)).
</details>

---

## Rapid-fire

| Question | One-line answer |
| --- | --- |
| List comprehension vs generator expression? | `[...]` builds the list in memory; `(...)` yields lazily. Use the generator for big inputs. |
| What does `__name__ == "__main__"` do? | Runs the block only when the file is executed, not imported, so your tool can be a library too. |
| Shallow vs deep copy? | `copy.copy` copies the container, shares nested objects; `copy.deepcopy` copies recursively. |
| `*args`, `**kwargs`? | Variable positional and keyword arguments; also how decorators forward arbitrary signatures. |
| How are strings stored? | Immutable Unicode; `bytes` is the raw type. Encode at the boundary, decode on the way in, and know which one a log line is. |
| `sorted(key=...)` vs `cmp`? | Key functions only; use `operator.itemgetter`/`attrgetter`; stable sort, so chain sorts for multi-key order. |
| What is `__slots__`? | Fixes an object's attributes to save memory and block accidental new ones; useful for millions of parsed events. |
| `logging` vs `print`? | Levels, handlers, formatting, and no secrets in logs: never log the token you just parsed. |

[← Coding](README.md) · [Index](../README.md)
