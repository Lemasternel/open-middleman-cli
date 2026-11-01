# I. The open-middleman-cli project

A CLI program to monitor the traffic of HTTP and HTTPS requests: a web debugging proxy.
Support for Windows 8 Embedded (the main target), without neglecting the most recent operating systems.

# II. Development stack

- Rust must be at version 1.77.2 so the binaries can run on Windows 8 Embedded.
- GitHub Spec Kit: https://github.com/github/spec-kit
- Log: https://github.com/rust-lang/log
- Clap: https://github.com/clap-rs/clap

# III. Development methodologies

## III.I Spec-Driven Development

- Specifications are the source of truth, not the code.
- If the code does not follow the specs, a report pointing to the code and the spec must be presented.

## III.II Test-Driven Development

- Test code is always generated first, even if it fails at first.
- It needs to cover as many test scenarios as possible.
- Each test method must cover only one test scenario.

# IV. Code convention

- Prohibited usage of unwrap() and expect() in production code.