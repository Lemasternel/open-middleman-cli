<!--
Sync Impact Report
- Version change: 1.0.0 → 1.1.0 (MINOR: two principles added, quality gates expanded)
- Modified principles: none renamed
- Added principles:
  - VIII. Spec-Driven Development (specs are the source of truth)
  - IX. Test-Driven Development (NON-NEGOTIABLE)
- Modified sections:
  - Development Workflow & Quality Gates: added test-first, one-scenario-per-test and
    spec-conformance gates; Constitution Check now covers Principles I–IX
  - Governance: specs ranked as source of truth for behavior, below this constitution
- Removed sections: none
- Templates requiring updates: none modified by this command (templates read the
  constitution at runtime)
- Resolved TODOs from 1.0.0:
  - Cargo.toml now declares `edition = "2021"` and `rust-version = "1.77.2"`, and
    `rust-toolchain.toml` pins 1.77.2 (Principle II satisfied).
- Follow-up TODOs: none
-->

# open-middleman-cli Constitution

## Core Principles

### I. Windows 8 Embedded First (NON-NEGOTIABLE)

open-middleman-cli is a command-line debugging proxy for HTTP and HTTPS traffic. Windows 8
Embedded is the primary target platform; current operating systems (recent Windows, Linux,
macOS) are secondary targets.

- Every feature MUST work on Windows 8 Embedded before it is considered done.
- Code MUST NOT depend on OS APIs, system libraries, or runtime components that are
  unavailable on Windows 8 Embedded (e.g. Windows 10+ only APIs, newer Schannel features,
  UWP/WinRT). Where a newer-OS capability is useful, it MUST be optional and gated behind a
  runtime or compile-time check with a Windows 8 Embedded fallback.
- Release artifacts MUST be self-contained executables that need no installer, no extra
  runtime, and no administrator rights for basic operation.
- Secondary platforms MUST NOT regress; platform-specific code MUST be isolated behind
  `cfg` attributes or a platform module.

**Rationale**: The main user base runs embedded Windows 8 devices that cannot be upgraded;
supporting newer systems matters but MUST never compromise the primary target.

### II. Pinned Toolchain & Dependency Discipline

- The Rust toolchain MUST be pinned to 1.77.2 (the last release that supports Windows 7/8),
  and the crate MUST declare `rust-version = "1.77.2"` and use edition 2021.
- Every dependency (direct and transitive) MUST compile with Rust 1.77.2 and MUST run on
  Windows 8 Embedded. `Cargo.lock` MUST be committed, and dependency versions MUST be
  pinned so `cargo update` cannot silently raise the MSRV.
- New dependencies MUST be justified in the feature plan (purpose, MSRV, platform
  support, license). Pure-Rust crates are preferred over crates needing system libraries
  or C toolchains that complicate the Windows 8 build.

**Rationale**: A single incompatible transitive dependency breaks the primary target, so
toolchain and dependency choices are governance decisions, not incidental ones.

### III. No Panics in Production Code

- `unwrap()` and `expect()` are PROHIBITED in production code. Errors MUST be propagated
  with `Result`/`?` and handled or reported meaningfully.
- Other panic paths (`panic!`, `unreachable!`, `todo!`, unchecked indexing or slicing on
  untrusted data) MUST NOT be reachable from network input.
- Test code (`#[cfg(test)]`, `tests/`) MAY use `unwrap()`/`expect()`.
- A failure in one proxied connection MUST NOT terminate the process or affect other
  connections.

**Rationale**: A proxy handles arbitrary, often malformed traffic; a panic drops every
in-flight session the user is debugging.

### IV. Faithful, Transparent Proxying

- The proxy MUST forward requests and responses unchanged (method, headers, header order
  where feasible, body bytes) unless the user explicitly asks for a modification.
- Captured data shown to the user MUST reflect what actually went over the wire; any
  decoding (chunked, gzip, etc.) for display MUST be labeled as such.
- HTTP/1.1 and HTTPS via `CONNECT` are the baseline protocols; any other protocol
  support MUST be declared in a feature spec before it is built.

**Rationale**: A debugging tool that alters traffic without saying so produces misleading
diagnoses.

### V. Observability Through `log`

- All diagnostic output MUST go through the `log` crate facade
  (https://github.com/rust-lang/log); `println!`/`eprintln!` are reserved for the
  intended CLI output (captured traffic, command results).
- Log verbosity MUST be selectable at runtime (e.g. CLI flag or environment variable).
- Errors MUST be logged with enough context (connection, host, phase) to diagnose them
  without a debugger.
- Captured traffic output MUST be separate from diagnostic logs (e.g. stdout vs. stderr
  or a file), so each can be redirected on its own.

**Rationale**: Embedded targets are often debugged remotely and from text output only.

### VI. Security of Intercepted Traffic

- HTTPS interception MUST be explicit and opt-in; the user MUST know when TLS is being
  terminated by the proxy.
- The root CA private key generated for interception MUST be unique per installation,
  MUST NOT be shipped in the repository or binaries, and MUST be stored with restrictive
  permissions.
- By default the proxy MUST listen on loopback only; listening on other interfaces MUST
  require an explicit flag.
- Captured traffic (which may contain credentials) MUST NOT be persisted or sent anywhere
  unless the user explicitly asks for it.

**Rationale**: An intercepting proxy is a powerful man-in-the-middle; unsafe defaults
would turn a debugging tool into a liability.

### VII. CLI-First Simplicity

- The product is a CLI: all functionality MUST be usable from command-line arguments and
  text output, with exit code 0 on success and non-zero on failure.
- Machine-readable output (e.g. JSON) SHOULD be offered for captured traffic so it can be
  scripted.
- Start simple (YAGNI): no GUI, no service/daemon management, and no abstraction layers
  without a concrete current need documented in the plan.
- Resource usage MUST stay modest (memory, threads, binary size), because embedded
  targets have limited hardware.

**Rationale**: Simple tools are easier to port, audit, and keep compatible with an old
platform.

### VIII. Spec-Driven Development

- Specifications (`spec.md`, `plan.md`, `tasks.md` and related Spec Kit artifacts) are
  the source of truth for behavior, not the code.
- No feature or behavior change MAY be implemented without a spec that describes it; when
  behavior must change, the spec MUST be updated first and the code second.
- When code does not follow its spec, a conformance report MUST be produced that cites
  both the code location (file and line) and the spec requirement it violates. The
  divergence MUST then be resolved by fixing the code or by amending the spec — never by
  leaving the two silently inconsistent.

**Rationale**: Keeping intent in specs makes behavior reviewable, portable across
refactors, and checkable without reading the implementation.

### IX. Test-Driven Development (NON-NEGOTIABLE)

- Test code MUST be written before the production code it exercises, and MUST be run and
  seen to fail (Red) before the implementation is written (Green), followed by Refactor.
- Tests MUST cover as many scenarios as practical for each requirement: success paths,
  error paths, boundary values, and malformed or hostile input.
- Each test function MUST cover exactly one scenario; its name MUST describe that
  scenario. Multiple unrelated assertions about different scenarios in one test are
  PROHIBITED.

**Rationale**: Tests written first prove that each requirement is testable and actually
exercised; one scenario per test makes a failure point directly at the broken behavior.

## Platform & Technology Constraints

- **Language**: Rust 1.77.2, edition 2021, pinned via `rust-toolchain.toml`.
- **Primary target**: Windows 8 Embedded (`x86_64-pc-windows-msvc`; 32-bit
  `i686-pc-windows-msvc` if the device fleet requires it).
- **Secondary targets**: current Windows, Linux and macOS releases.
- **Logging**: `log` crate facade with a backend that is compatible with Rust 1.77.2.
- **TLS**: the TLS stack MUST work on Windows 8 Embedded without relying on OS TLS
  features introduced after Windows 8 (a pure-Rust stack such as `rustls`, at a version
  compatible with Rust 1.77.2, is preferred).
- **Specification workflow**: features follow GitHub Spec Kit
  (https://github.com/github/spec-kit): specify → plan → tasks → implement.

## Development Workflow & Quality Gates

Before a change is merged:

1. `cargo build` and `cargo test` MUST pass with the pinned 1.77.2 toolchain.
2. `cargo clippy` MUST pass with no warnings, and `cargo fmt --check` MUST be clean.
3. Production code MUST contain no `unwrap()`/`expect()` (enforced with clippy's
   `unwrap_used`/`expect_used` lints outside test code).
4. Changes that touch networking, TLS, or platform code MUST be tested on Windows 8
   Embedded (a real device or VM) before release; secondary platforms MUST at least build.
5. Proxy behavior changes MUST include integration tests that run real HTTP/HTTPS
   exchanges through the proxy against a local test server.
6. Task lists MUST order test tasks before the implementation tasks they cover, and
   reviews MUST confirm that each test covers a single scenario (Principle IX).
7. The implemented behavior MUST match its spec; any divergence MUST be reported and
   resolved as described in Principle VIII before merge.
8. Every feature plan MUST include a Constitution Check that confirms compliance with
   Principles I–IX or records a justified exception.

## Governance

- This constitution overrides other project practices. Where `CLAUDE.md` or other
  guidance files conflict with it, the constitution wins, and the conflicting file MUST be
  updated.
- Below this constitution, feature specifications are the authority on behavior; code and
  tests MUST conform to them (Principle VIII).
- Amendments require a documented change to this file, a Sync Impact Report, a version
  bump, and a migration note for any work affected.
- Versioning follows semantic versioning: MAJOR for removing or redefining a principle,
  MINOR for adding a principle or section or materially expanding guidance, PATCH for
  clarifications and wording.
- Every spec, plan, and review MUST check compliance; any added complexity or exception
  MUST be justified in the plan's Complexity Tracking section.
- `CLAUDE.md` provides runtime development guidance for AI-assisted work and MUST stay
  consistent with this constitution.

**Version**: 1.1.0 | **Ratified**: 2026-10-31 | **Last Amended**: 2026-10-31
