Python Module CLI Best Practices

This document is a practical reference for designing, building, packaging, shipping, and operating command-line interfaces (CLIs) implemented as Python modules/packages. It aims to be broadly applicable across project sizes—from small utilities to complex multi-command tools—while remaining compatible with modern Python packaging practices.

Goals and guiding principles
Design goals for a Python CLI

Predictable UX: clear commands, consistent flags, stable output contracts, and well-defined exit codes.

Scriptable by default: work well in pipes, automation, and CI (stdin/stdout/stderr, deterministic output).

Importable + reusable: the CLI should be a thin layer over importable application logic (so other Python code can call it).

Packageable and installable: install cleanly via pip or pipx, with correct entry points and metadata.

Observable and debuggable: structured logging and a controllable verbosity model.

Secure by default: safe handling of inputs, files, secrets, and dependencies.

A strong general CLI reference for UX conventions is the Command Line Interface Guidelines (e.g., stdout/stderr separation, exit codes, subcommands, consistency).

Architecture: keep CLI code thin, keep logic importable
Recommended layering

Core library/module: pure functions/classes that implement your domain logic.

CLI layer: argument parsing, config assembly, calling core functions, formatting output, setting exit codes.

Entry points: wrappers (console_scripts / [project.scripts]) that call the CLI layer.

This layering makes the tool easier to test (core logic tested without subprocesses), easier to reuse as a library, and easier to maintain as complexity grows.

Standard “main pattern”

Use a main(argv=None) -> int and keep if __name__ == "__main__" as a thin wrapper:

# src/mytool/cli.py
from __future__ import annotations

import sys

def main(argv: list[str] | None = None) -> int:
    argv = sys.argv[1:] if argv is None else argv
    # parse args, call core logic, print output
    return 0

if __name__ == "__main__":
    raise SystemExit(main())


Rationale:

Returning an int makes exit code handling explicit.

The wrapper provides a single top-level exit path, which improves consistency and testability.

(Using sys.exit() / raising SystemExit is the standard approach for scripts. )

Provide python -m support via __main__.py

If you want users to run your tool as a module (python -m mytool), include __main__.py. Python executes __main__.py when invoking a package with -m.

Example:

# src/mytool/__main__.py
from __future__ import annotations
from .cli import main

raise SystemExit(main())


This is a high-value, low-cost feature:

Works even if entry point scripts aren’t available in some environments.

Helps debugging and local development.

Project layout: use a src/ layout for CLIs that ship as packages

For packaged CLIs (distributed via PyPI or internal indexes), prefer the src/ layout:

mytool/
  pyproject.toml
  README.md
  src/
    mytool/
      __init__.py
      __main__.py
      cli.py
      core.py
  tests/


Why:

The src layout helps ensure tests and local runs exercise the installed package (or an editable install) rather than accidentally importing from the working directory.

This reduces “works on my machine” import path surprises, especially in CI.

Packaging and distribution (modern Python)
Use pyproject.toml as the source of truth

Modern packaging centers on pyproject.toml for build configuration and project metadata.

Key concepts:

[build-system] declares how to build your package and its build dependencies.

PEP 517 defines a standard interface between build frontends (like pip) and build backends (like setuptools/flit/hatchling).

PEP 621 standardizes core project metadata in [project].

Define CLI commands via entry points (don’t ship raw scripts)

For cross-platform CLIs, define commands as entry points (e.g., [project.scripts]), not as “drop a script file into PATH.” Entry points are a standardized mechanism in Python packaging.

pyproject.toml example
[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "mytool"
version = "1.2.3"
requires-python = ">=3.10"
dependencies = [
  # keep runtime deps minimal
]

[project.scripts]
mytool = "mytool.cli:main"


Notes:

Keep the script name stable; treat it like a public API.

Avoid command name collisions with other tools (entry points must be unique within a distribution; conflicts across distributions are resolved by the installer/consumer).

Recommend pipx for installing CLI applications

For end-user installation of Python CLIs, pipx is widely recommended because it installs apps into isolated environments and exposes their executables on PATH.

Best practice:

Document both pipx install mytool and pip install mytool (the latter for “inside a virtualenv” scenarios).

Ensure your tool works cleanly when installed with pipx (especially entry points and dependency metadata).

CLI UX best practices (flags, help, output, behavior)
Commands and subcommands

Use subcommands when:

The tool has multiple distinct workflows (e.g., mytool init, mytool run, mytool status).

Each workflow has different required arguments.

This reduces cognitive load and lets each command have focused help text.

Consistency rules

Use the same flag names across subcommands (e.g., --verbose means the same thing everywhere).

Prefer conventional flags: -h/--help, -v/--verbose, -q/--quiet, --version.

Don’t invent novel flag syntax.

Help text and discoverability

Your CLI should always provide:

--help on the root command and subcommands.

Short, action-oriented help strings for each argument.

Examples in the help epilog or README for common tasks.

If using the standard library, argparse automatically generates help/usage text and errors for invalid arguments.

Output contracts: stdout vs stderr, structured output, and TTY awareness
stdout vs stderr

Primary output goes to stdout.

Diagnostics (warnings, progress logs, human guidance) go to stderr.

This is crucial for piping and automation.

Provide a structured output mode

If your tool outputs “data,” provide at least one machine-readable format (commonly JSON):

--format json or --json

Ensure JSON output is stable across versions (treat as an API).

Best practice: In JSON mode, suppress decorative formatting and keep stdout purely the JSON payload.

Color and formatting

Use color only when output is a TTY.

Honor NO_COLOR to disable ANSI colors when users request it.

Practical rules:

Default: color only if sys.stdout.isatty().

If NO_COLOR is set (non-empty), disable color even if TTY.

Offer --color auto|always|never if color matters.

Exit codes and error handling
Return meaningful exit codes

Rules of thumb:

0 = success

non-zero = failure

Document your exit codes, and keep them stable once published.

A pragmatic exit code scheme:

0: success

1: unexpected error

2: command line usage error (invalid args, missing required flags)

3: configuration error (missing/invalid config file)

4: external dependency failure (network/service)

5: partial failure (batch operation)

Keep exit codes within 0–255 because many shells treat exit codes modulo 256.

Don’t dump stack traces by default

Default behavior should be:

A concise error message to stderr

A non-zero exit code

Optional --debug / --traceback / -vv to show stack traces

If you use Click, it has a defined model for exceptions and exit codes, including usage errors and help/exit behavior.

Choosing an argument parsing approach
General guidance

argparse (stdlib): excellent default for many CLIs, no dependencies, good help generation.

Click: strong for composable multi-command CLIs, robust UX defaults, strong ecosystem.

Typer: Click-based but uses type hints to reduce boilerplate; great for modern Python codebases.

Best practice: pick one primary approach per CLI. Mixing frameworks is possible but increases complexity; only do it deliberately (Typer even documents interop with Click, but treat that as advanced usage).

Argparse pattern for subcommands

Argparse supports subcommands using add_subparsers().

Common effective pattern: bind a function to each subparser via set_defaults() and call it after parse.

Configuration: files, environment variables, and precedence
A sane precedence order

A widely workable precedence model:

CLI flags (highest precedence)

Environment variables

Config file

Defaults (lowest precedence)

Make it predictable and document it.

Where to store config/cache/data

Do not invent OS-specific paths yourself. Use platformdirs to locate user config/cache/data directories consistently across platforms.

Example:

from platformdirs import user_config_dir, user_cache_dir

config_dir = user_config_dir("mytool")
cache_dir = user_cache_dir("mytool")


Best practices:

Namespace your files under an app name to avoid collisions.

Separate:

config (user intent; safe to keep)

cache (rebuildable; safe to delete)

data/state (persistent runtime state; use carefully)

Provide --config PATH to override config location when needed.

Secrets in config

Never store secrets in plaintext config unless explicitly requested and clearly documented.

Prefer environment variables, OS keychains, or external secret managers for sensitive data.

Logging and observability
Use the standard logging module

Python’s logging guide recommends creating a logger via getLogger(__name__) and using appropriate log levels.

CLI logging best practices:

Send logs to stderr (so stdout remains clean for program output).

Provide verbosity controls:

-v increases verbosity (often maps to INFO/DEBUG)

-q reduces output

Avoid configuring global logging deep inside libraries; configure logging at the CLI entrypoint.

Example:

import logging

def configure_logging(verbose: int) -> None:
    level = logging.WARNING
    if verbose == 1:
        level = logging.INFO
    elif verbose >= 2:
        level = logging.DEBUG

    logging.basicConfig(level=level, format="%(levelname)s %(name)s: %(message)s")

Safe file I/O and filesystem behavior
Avoid surprising writes

Don’t write into the current working directory unless explicitly requested.

Use explicit --output paths or a clearly documented default location.

For commands that modify files, consider:

--dry-run to preview changes

--force for destructive actions

--backup or write-to-temp-and-rename for atomicity

Be atomic where possible

When writing output files:

Write to a temp file in the same directory

fsync if durability matters

Rename into place (atomic on most filesystems)

Performance and startup time

CLIs often feel “slow” due to import time and initialization, not actual work.

Best practices:

Keep imports in the CLI entrypoint minimal; delay heavy imports until needed (especially for subcommands).

Avoid doing network calls, config discovery, or expensive initialization just to display --help or --version.

Cache expensive computed data if it’s safe and valid to reuse; store cache under platformdirs cache dir.

Version reporting (--version) and metadata
Use packaging metadata where appropriate

For installed packages, importlib.metadata provides access to distribution metadata.

Common pattern:

from importlib.metadata import version, PackageNotFoundError

def get_version() -> str:
    try:
        return version("mytool")  # distribution name
    except PackageNotFoundError:
        return "0+unknown"


Practical note: distribution names and import package names can differ; keep them aligned when possible to reduce confusion.

Follow PEP 440 for versioning

Use PEP 440-compliant versions so packaging tools and dependency resolvers behave correctly.

Testing strategy for CLIs
Layered tests

Unit tests: test core logic directly (no subprocess).

Parser tests: validate argument parsing and defaulting.

Integration tests: run the CLI end-to-end and assert:

exit code

stdout/stderr

output format

Subprocess vs in-process

For argparse-based CLIs, subprocess tests (subprocess.run) are often simplest and realistic.

For Click/Typer-based CLIs, consider framework test helpers (e.g., Click’s testing tools), but still keep at least some subprocess tests for realism.

Test in a package-like environment

Using a src/ layout helps avoid accidental imports from the working directory and more closely matches real installation behavior.

Development best practices
Local development workflow

Use a virtual environment (venv, uv, etc.).

Install your package in editable mode for development.

Ensure python -m mytool ... works during development (thanks to __main__.py).

Developer ergonomics

Provide make/task runner commands (or equivalent) for:

lint

test

format

typecheck

build

Add shell completion if your framework supports it (Click/Typer have support patterns; treat as an optional enhancement).

Documentation

README should include:

install instructions (including pipx)

quick-start examples

configuration locations and precedence

exit code meanings

output formats

Packaging docs include guidance on creating and packaging command-line tools and installing standalone CLI tools via pipx; align your docs with those expectations.

Production best practices

“Production” for a CLI commonly means:

Running in CI pipelines

Running on servers or containers

Used by other automation or as part of larger systems

Reproducible installs and supply-chain safety

Pin tool versions where repeatability matters (CI/build systems).

Prefer installing released versions (tags) rather than arbitrary git commits unless necessary.

Keep runtime dependencies minimal; extras can be used for optional features.

Operational stability

Avoid interactive prompts unless explicitly requested (provide --yes/--no-input).

Always support non-interactive usage with clear exit codes and structured outputs.

Avoid writing secrets to logs; audit your logging at DEBUG level too.

Upgrades and compatibility

Maintain backward compatibility of:

command names

flag names/behavior

structured output fields

exit code meanings

When breaking changes are unavoidable:

bump major version (PEP 440 compliant)

provide a migration guide

consider deprecation warnings in stderr for at least one release cycle

Practical templates
Minimal argparse skeleton (single command)
# src/mytool/cli.py
from __future__ import annotations
import argparse
import sys

def build_parser() -> argparse.ArgumentParser:
    p = argparse.ArgumentParser(prog="mytool", description="Describe what the tool does.")
    p.add_argument("--version", action="store_true", help="Show version and exit.")
    p.add_argument("-v", "--verbose", action="count", default=0, help="Increase verbosity.")
    return p

def main(argv: list[str] | None = None) -> int:
    argv = sys.argv[1:] if argv is None else argv
    args = build_parser().parse_args(argv)

    if args.version:
        print("mytool 1.2.3")
        return 0

    # run core logic here
    return 0

if __name__ == "__main__":
    raise SystemExit(main())


Argparse’s strength is user-friendly parsing with generated help and built-in error handling.

Minimal pyproject.toml with scripts
[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "mytool"
version = "1.2.3"
requires-python = ">=3.10"
dependencies = []

[project.scripts]
mytool = "mytool.cli:main"


This aligns with modern packaging guidance for pyproject.toml and entry points.

Checklists
CLI design checklist

 Clear command naming and subcommand grouping (if needed)

 Consistent flags across commands

 Helpful --help everywhere; examples documented

 stdout for primary output; stderr for diagnostics

 Structured output option (e.g., JSON) for automation

 TTY-aware formatting; honor NO_COLOR

 Stable, documented exit codes; 0 success, non-zero failure

Packaging checklist

 pyproject.toml contains [build-system] and [project] metadata

 Entry points defined via [project.scripts] / console scripts

 src/ layout used for packaged projects

 python -m mytool supported with __main__.py

 Install guide includes pipx for end-users

Production-readiness checklist

 Non-interactive by default; prompts require explicit opt-in

 Logging is controllable and does not pollute stdout

 Config precedence documented; config stored via platformdirs

 No secrets in logs or default config

 Backwards compatibility policy for flags/output/exit codes

 Versioning follows PEP 440

Recommended references (authoritative)

Python Packaging User Guide: pyproject.toml, src layout, and CLI packaging guides

PEP 517 / 518 / 621 for modern packaging standards

Python docs: argparse, logging, __main__, importlib.metadata

CLI Guidelines for UX conventions

pipx guidance for installing standalone CLI tools

NO_COLOR specification