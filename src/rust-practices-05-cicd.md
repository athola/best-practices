# CI/CD Pipeline

## Workflow Structure

```yaml
name: Rust CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Security Scan
        run: |
          cargo audit
          gitleaks --no-git
  
  test:
    needs: security-scan
    runs-on: ubuntu-latest
    strategy:
      matrix:
        profile: [ci, full]
    steps:
      - uses: actions/checkout@v4
      - name: Setup Rust
        uses: actions-rs/toolchain@v1
        with:
          profile: minimal
          toolchain: stable
          override: true
      - name: Cache Dependencies
        uses: actions/cache@v3
        with:
          path: |
            ~/.cargo/registry
            ~/.cargo/git
            target
          key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
      - name: Run Tests
        run: |
          if [ "${{ matrix.profile }}" == "ci" ]; then
            cargo test --profile ci
          else
            cargo test --release
          fi
```

## Security Scanning

```yaml
security:
  runs-on: ubuntu-latest
  steps:
    - name: Gitleaks Scan
      run: gitleaks --no-git --verbose
    
    - name: Semgrep Scan
      run: semgrep --config=.semgrep.yml
    
    - name: Cargo Audit
      run: cargo audit
```

## Custom Security Rules

```yaml
# .semgrep.yml
rules:
  - id: detect-gcp-api-key
    pattern-regex: 'AIza[0-9A-Za-z\-_]{35}'
    message: "Potential GCP API key detected"
    severity: "ERROR"
    languages: [rust]
    
  - id: detect-sql-injection
    pattern: |
      query($INPUT, ...)
    message: "Potential SQL injection vulnerability"
    severity: "WARNING"
    languages: [rust]
```

### CI/CD Best Practices

**Fast Feedback**
CI/CD pipelines should provide fast feedback to developers. The longer it takes to get feedback, the more expensive it is to fix issues.

**Comprehensive Validation**
CI/CD pipelines should validate all aspects of the codebase, not just functionality. This includes code quality, security, performance, and compliance.

**Automated Deployment**
Deployment should be automated to reduce human error and ensure consistency. Manual deployment processes are risky and unreliable.

**Gradual Rollout**
Deployments should be gradual to minimize risk. Use canary releases, feature flags, and gradual rollout strategies.

### CI/CD Patterns

**Pipeline Stages**
```yaml
# Multi-stage pipeline
stages:
  - validate    # Fast validation checks
  - test        # Comprehensive testing
  - build       # Build and package
  - security    # Security scanning
  - deploy      # Deployment to staging
  - release     # Deployment to production
```

**Conditional Execution**
```yaml
# Run only on relevant changes
jobs:
  test:
    if: contains(github.event.commits[0].message, '[test]')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: cargo test
```

**Matrix Testing**
```yaml
# Test across multiple environments
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    rust: [stable, beta, nightly]
```

### Build Automation

Use a Makefile for common tasks:

```makefile
.PHONY: build test clean deploy

build:
	@echo "Building project..."
	cargo build  # or uv build for Python

test:
	@echo "Running tests..."
	cargo test  # or pytest for Python

clean:
	@echo "Cleaning build artifacts..."
	cargo clean  # or appropriate clean command

deploy:
	@echo "Deploying to production..."
	# Deploy commands
```

**Build Automation Principles**

- **Automate Everything That Can Be Automated**: Manual processes are error-prone and time-consuming. Automate build, test, and deployment processes to improve reliability and efficiency.
- **Make Build Processes Reproducible**: Builds should be reproducible across different environments and over time. Use dependency locking and version pinning to ensure consistency.
- **Provide Clear Feedback**: Build processes should provide clear, actionable feedback when they fail. Developers should be able to quickly understand what went wrong and how to fix it.
- **Optimize for Developer Experience**: Build processes should be fast and easy to use. Slow or complicated build processes discourage developers from running them frequently.

Next: [Performance Optimization](./rust-practices-06-performance.md)