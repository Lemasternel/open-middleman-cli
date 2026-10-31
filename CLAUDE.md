# The open-middleman-cli project

A CLI program to monitor the traffic of HTTP and HTTPS requests: a web debugging proxy.
Support for Windows 8 Embedded (the main target), without neglecting the most recent operating systems.

# Development stack

- Rust 1.77.2 so the binaries can run on Windows 8 Embedded.
- GitHub Spec Kit: https://github.com/github/spec-kit
- Log: https://github.com/rust-lang/log

# Code convention

- Use Result instead of exceptions.