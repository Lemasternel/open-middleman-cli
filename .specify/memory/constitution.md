<!--
Sync Impact Report
- Version change: 1.0.0 -> 1.1.0 (MINOR).
- Added principle: VII. Mandatory Logging. Every significant operation and
  every failure path of the CLI MUST be logged with a severity level and
  context; log records never go to stdout, so they cannot corrupt the user
  contract of Principle VI. The principle does not name a logging library.
- Removed from Additional Constraints (stack, not constitution):
  - the logging library and its link (`log` crate) and the requirement to pin
    it in `Cargo.lock`;
  - the argument-parsing library (`clap` derive);
  - the "Rust toolchain: 1.77.2" bullet, already covered by Principle IV.
- Kept on purpose: Windows 8 Embedded target, `x86_64-pc-windows-msvc`,
  `rust-version = "1.77.2"`, edition 2021, the pinned toolchain and the legacy
  console requirement in Principle IV, because they are compatibility
  requirements of the main target, not stack choices. Also kept: the
  dependency on `open-middleman-lib` pinned in `Cargo.lock` (it defines the
  project, see Principle I), GitHub Spec Kit (process) and the clippy lints
  that enforce Principle V.
- Rationale: logging is a core requirement of a debugging proxy; the libraries
  chosen to implement logging and argument parsing are implementation
  decisions that belong to the plan, not to the constitution. Same change as
  open-middleman-lib constitution 2.3.0.
- Backwards compatibility: no code exists yet that emits logs, so nothing
  breaks; future features MUST include logging in their specs and plans.
- Follow-up files: none required. Specs and plans written from now on MUST
  describe the logging of each feature.
-->
# open-middleman-cli Constitution

## Core Principles

### I. Thin Bridge to the Library
The project delivers a single Rust binary crate: a command-line bridge
between the user and the functions of `open-middleman-lib`. All domain logic
(proxying, capture, parsing, filtering, storage) MUST live in the library
and MUST NOT be reimplemented in this project. The CLI is responsible only
for argument parsing, input/output, output formatting, and exit codes.
When a needed behavior is missing from the library, it is requested or added
in the library first and then consumed here. The CLI MUST depend on the
library as a regular dependency and MUST NOT copy or fork its code.

### II. Specifications Are the Source of Truth
Specifications define expected behavior; the code MUST conform to them.
When code and spec diverge, a report referencing the code location and the
spec section MUST be produced by the author of the change (or the reviewer who
detects it) and attached to the pull request. The divergence is resolved by
fixing the code or by amending the spec through the amendment process (see
Governance).

### III. Test-First (NON-NEGOTIABLE)
Tests for a behavior are written before its implementation and MUST be
observed failing before implementation code is written; the failing test
MUST be committed before the implementation commit. Each test function
covers exactly one scenario. Every command and option is covered at least
for its success path, its error path, and its boundary conditions, including
the exit code and the text written to stdout and stderr.

### IV. Target Compatibility
The CLI MUST build and run on Windows 8 Embedded, x86_64
(`x86_64-pc-windows-msvc`). It declares `rust-version = "1.77.2"` in
`Cargo.toml`, uses `edition = "2021"` (the 2024 edition requires a newer
compiler), and the development toolchain is pinned to 1.77.2 via
`rust-toolchain.toml`. Every dependency (including the argument-parsing
framework and the library) MUST build on 1.77.2 and MUST NOT require Windows
APIs unavailable on Windows 8. The console output MUST remain usable on the
legacy Windows console (no reliance on ANSI/VT sequences without detection
and a plain-text fallback).

### V. Production Code Safety
`unwrap()`, `expect()` and `panic!()` are prohibited in production code.
"Production code" is everything outside `#[cfg(test)]` modules and the
`tests/` directory. Failures are reported through `Result` with an explicit
error type; `Option` is used only for legitimate absence of a value. The
`main` function translates errors into a message on stderr and a non-zero
exit code. Tests may use `unwrap()` and `expect()`.
Enforced in CI via `clippy::unwrap_used`, `clippy::expect_used` and
`clippy::panic`.

### VI. Stable User Contract
Commands, options, output formats and exit codes are the public interface of
this project. Results go to stdout and diagnostics go to stderr. Exit code 0
means success and non-zero means failure, with each documented failure class
mapped to a distinct code. Breaking changes to this contract require a
specification update and, when incompatible, a MAJOR version bump of the CLI.

### VII. Mandatory Logging
Logging is REQUIRED, not optional. The CLI MUST emit log records for every
significant operation (for example: parsing and validating arguments, loading
configuration, starting and stopping the proxy through the library, opening
and writing output files) and for every failure path, before the error is
reported to the user and the process exits. Each record MUST carry a severity
level (error, warning, info, debug, trace) and enough context to identify the
operation it describes. As the consuming application, the CLI decides where
log records are written, including the records emitted by `open-middleman-lib`;
log records MUST NOT be written to stdout, so they never mix with command
results. The choice of logging library is an implementation decision recorded
in the plan, not in this constitution. Every spec and plan MUST state what its
feature logs, and code review rejects changes that add operations or failure
paths without logging.

## Additional Constraints

- Library dependency: `open-middleman-lib`, pinned to a specific release or
  revision and recorded in `Cargo.lock`.
- Spec process: GitHub Spec Kit (https://github.com/github/spec-kit).

## Development Workflow

- Any behavior change starts with a spec update.
- Commit messages follow Conventional Commits 1.0.0
  (https://www.conventionalcommits.org, source:
  https://github.com/conventional-commits/conventionalcommits.org):
  `<type>[optional scope][!]: <description>`, with optional body and footers.
  Only the abbreviated type names are allowed: `feat`, `fix`, `docs`,
  `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore` and `revert`
  (never long forms such as `feature` or `bugfix`). Breaking changes are
  marked with `!` or a `BREAKING CHANGE:` footer.
- Branch names follow Conventional Branch
  (https://conventional-branch.github.io, source:
  https://github.com/conventional-branch/conventional-branch):
  `<type>/<description>`, using the same abbreviated naming. Allowed types
  are `feat/`, `fix/`, `hotfix/`, `release/` and `chore/`; the long forms
  `feature/` and `bugfix/` are not accepted. Names use lowercase letters,
  digits and hyphens (dots only in release versions), with no consecutive,
  leading or trailing hyphens or dots. The default branch (`master`/`main`)
  and `develop`, if used, are exempt.
- Code review verifies compliance with all Core Principles, in particular
  that no domain logic has leaked from the library into the CLI and that the
  logging required by Principle VII is present.
- CI builds and tests the CLI on the 1.77.2 toolchain.
- CI runs clippy with `unwrap_used`, `expect_used` and `panic` denied.

## Governance

- This constitution supersedes other project practices.
- Amendments require a documented rationale, maintainer approval (the
  repository owner), and a backwards-compatibility assessment, and are made
  through a pull request that updates this file.
- Constitution versioning: MAJOR for removing or redefining a principle in an
  incompatible way, MINOR for adding or materially expanding a principle or
  rule, PATCH for wording and clarifications.
- Violations must be justified in the plan's Complexity Tracking section.

**Version**: 1.1.0 | **Ratified**: 2026-11-21 | **Last Amended**: 2026-11-24
