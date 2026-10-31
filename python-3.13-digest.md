# Python 3.13 Release Notes Digest

A condensed handout for our study group's Python 3.13 unit, distilled from the official **"What's New In Python 3.13"** release notes (Python 3.13 was released on October 7, 2024). This digest reflects what that page says today, not a write-up from memory.

## Highlights

- **PEP 703 — experimental free-threaded CPython**: CPython 3.13 has experimental support for running with the global interpreter lock disabled, via the `python3.13t` executable, official Windows/macOS free-threaded installers, or building from source with `--disable-gil`. The mode is experimental: expect some bugs and a substantial single-threaded performance hit; check state with `sys._is_gil_enabled()`; optional runtime GIL control via `PYTHON_GIL` or `-X gil=1`.
- **PEP 744 — an experimental JIT compiler**: a basic JIT is added when CPython is configured with `--enable-experimental-jit` (values `no`, `yes`, `yes-off`, `interpreter`). It is disabled by default; hot Tier 1 bytecode is translated to a Tier 2 IR ("uops"), optimized, and translated to machine code via copy-and-patch (LLVM build dependency). Performance gains are modest so far.
- **PEP 667 — defined mutation semantics for `locals()`**: optimized scopes (functions, generators, coroutines, comprehensions, generator expressions) now return independent snapshots of currently assigned locals, and `FrameType.f_locals` returns a write-through proxy so debuggers and tracing tools can reliably update locals even during concurrent execution. This also affects `exec()`/`eval()` when no explicit namespace is provided.
- **Better error messages, colored by default**: tracebacks are highlighted in color in the terminal, the new REPL, and doctest output (controlled via `PYTHON_COLORS`, `NO_COLOR`, `FORCE_COLOR`); messages are more helpful when a script shadows a standard library or third-party module, and incorrect keyword arguments are suggested (e.g. `maxsplit` vs `max_split`).
- **New typing features**: **PEP 696** gives `TypeVar`, `ParamSpec`, and `TypeVarTuple` default values; **PEP 702** adds the `warnings.deprecated()` decorator for marking deprecations in the type system and at runtime; **PEP 705** adds `typing.ReadOnly` for `TypedDict` items; **PEP 742** adds `typing.TypeIs` as an alternative to `TypeGuard`.
- **Mobile and web platforms officially supported**: **PEP 730** makes Apple's iOS a tier-3 platform (`arm64-apple-ios`, `arm64-apple-ios-simulator`; `x86_64-apple-ios-simulator` best-effort); **PEP 738** makes Android a tier-3 platform (`aarch64-linux-android`, `x86_64-linux-android`; 32-bit targets best-effort); `wasm32-wasi` is now tier 2 and `wasm32-emscripten` is no longer officially supported.
- **Standard-library additions**: the new `dbm.sqlite3` module is now the default `dbm` backend; `copy.replace()` (with the `__replace__()` protocol); `base64.z85encode()`/`z85decode()`; `math.fma()`; `os.process_cpu_count()` (overridable via `PYTHON_CPU_COUNT`/`-X cpu_count`); the Linux timer file-descriptor API; `argparse` can now deprecate options, positional arguments, and subcommands; and the new `PythonFinalizationError` is raised when operations are blocked during finalization.
- **Release schedule change**: **PEP 602** is updated to extend the full-support ("bugfix") period to two years — Python 3.13 and later get two years of full support followed by three years of security fixes (3.9–3.12 had one and a half years).

## Notable deprecations

### Removed in 3.13 (previously deprecated)

- **PEP 594**: the remaining 19 "dead batteries" — `aifc`, `audioop`, `cgi`, `cgitb`, `chunk`, `crypt`, `imghdr`, `mailcap`, `msilib`, `nis`, `nntplib`, `ossaudiodev`, `pipes`, `sndhdr`, `spwd`, `sunau`, `telnetlib`, `uu`, `xdrlib`.
- The **2to3** tool and `lib2to3` module (deprecated in 3.11); the `tkinter.tix` module (deprecated in 3.6); `locale.resetlocale()`; the `typing.io` and `typing.re` namespaces; chained `classmethod` descriptors; using `Path` objects as context managers; `re.template()` and `re.TEMPLATE`/`re.T`; `configparser.LegacyInterpolation`; deprecated `EntryPoint` subscript access in `importlib.metadata`; deprecated `unittest` functions (`findTestCases`, `makeSuite`, `getTestCaseNames`, `TestProgram.usageExit()`); the `_cafile_`/`_capath_`/`_cadefault_` parameters of `urllib.request.urlopen()`; the `MacOSX` class in `webbrowser`; `turtle.RawTurtle.settiltangle()`; and the keyword-argument `TypedDict` creation syntax.

### Newly deprecated in 3.13

- `array`: the `'u'` format code (`wchar_t`) is deprecated at runtime; use `'w'` (`Py_UCS4`) instead (removal scheduled for 3.16).
- `ctypes`: the undocumented `SetPointerType()` is soft-deprecated (removal in 3.15); `ARRAY()` is soft-deprecated in favor of `type * length` multiplication.
- `traceback`: the `TracebackException.exc_type` attribute is deprecated; use the new `exc_type_str` instead.
- `re`: passing `maxsplit`, `count`, or `flags` positionally to `split()`, `sub()`, and `subn()` becomes keyword-only in a future version.
- `mimetypes`: file-path arguments to `guess_type()` are soft-deprecated; use `guess_file_type()` instead.
- `sqlite3`: positional extra args to `connect()`/`Connection`, keyword args for `create_function()`/`create_aggregate()`, and keyword callables for `set_authorizer()`/`set_progress_handler()`/`set_trace_callback()` flip to keyword-only/positional-only in 3.15.
- `pathlib`: `.PurePath.is_reserved()` is pending removal in 3.15.
- `platform`: `platform.java_ver()` is pending removal in 3.15.
- `wave`: `getmark()`, `setmark()`, and `getmarkers()` on `Wave_read`/`Wave_write` are pending removal in 3.15.
- `http.server`: `CGIHTTPRequestHandler` and the `--cgi` flag are pending removal in 3.15.
- `typing`: the undocumented keyword-argument syntax for `NamedTuple` creation, omitting/`None` `fields` for `NamedTuple`/`TypedDict`, `@typing.no_type_check_decorator`, and `typing.AnyStr` are pending removal (3.16/3.18).
- `decimal`: the undocumented `'N'` format specifier is pending removal in 3.18.
- `glob`: the undocumented `glob0()` and `glob1()` functions are soft-deprecated.

### Deprecation-shaped warnings in the language itself

- `ast` node constructors: omitting a required argument now emits a `DeprecationWarning` (will raise an exception in 3.15); passing an unrecognized keyword also is deprecated (will raise in 3.15).
- `dis`: the `get_instructions()` `show_caches` argument is deprecated and has no effect; jump-target and handler labels are shown logically by default (`-O` / `show_offsets` for offsets).
- `asyncio.create_task()`, `loop.create_task()`, and `TaskGroup.create_task()` accidentally received a `**kwargs` argument in 3.13.3, breaking custom task-factories; the old contract is honored again from 3.13.4 onward.

## Source

Source: "What's New In Python 3.13" — the official Python 3.13 release notes, at https://docs.python.org/3/whatsnew/3.13.html
