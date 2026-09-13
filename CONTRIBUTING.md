# Contributing

Thank you for contributing to OpenEDA IP-XACT.

## Development principles

- Follow the approved domain and implementation architecture.
- Do not introduce Qt, AWorldMagic, Workbench, or Kactus implementation dependencies into the core.
- Keep XML serialization separate from the canonical semantic domain.
- Add tests for behavioral changes.
- Run formatting, linting, and tests before submitting changes.

## Required checks

    cargo fmt --all --check
    cargo clippy --workspace --all-targets -- -D warnings
    cargo test --workspace
