# Security

## Production Hardening

```dockerfile
FROM rust:1.75-slim AS builder

# Build with security flags
ENV RUSTFLAGS="-C target-cpu=native -C link-arg=-s"

# Production image with minimal attack surface
FROM debian:bookworm-slim

# Create non-root user
RUN useradd --create-home --shell /bin/bash app
USER app

# Copy only necessary binaries
COPY --from=builder /usr/local/bin/app /usr/local/bin/

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

### Security Best Practices

**Least Privilege**
Run applications with the minimum privileges necessary:

```rust
// Drop privileges after initialization
pub fn drop_privileges() -> Result<()> {
    // Switch to non-root user
    unsafe {
        let pwd = libc::getpwnam("app".as_ptr() as *const libc::c_char);
        if pwd.is_null() {
            return Err("User not found".into());
        }
        
        if libc::setgid((*pwd).pw_gid) != 0 {
            return Err("Failed to set group ID".into());
        }
        
        if libc::setuid((*pwd).pw_uid) != 0 {
            return Err("Failed to set user ID".into());
        }
    }
    
    Ok(())
}
```

**Input Validation**
Validate all external inputs:

```rust
use validator::Validate;

#[derive(Debug, Validate)]
pub struct UserInput {
    #[validate(length(min = 1, max = 100))]
    pub username: String,
    
    #[validate(email)]
    pub email: String,
    
    #[validate(range(min = 13, max = 120))]
    pub age: u8,
}

pub fn process_user_input(input: UserInput) -> Result<()> {
    input.validate()?;
    // Process validated input
    Ok(())
}
```

**Secure Randomness**
Use cryptographically secure random number generation:

```rust
use rand::{thread_rng, Rng};
use rand::distributions::Alphanumeric;

pub fn generate_secure_token(length: usize) -> String {
    thread_rng()
        .sample_iter(&Alphanumeric)
        .take(length)
        .map(char::from)
        .collect()
}
```

**Memory Safety**
Leverage Rust's memory safety features:

```rust
// Safe string handling
pub fn process_user_input(input: &str) -> String {
    // Rust prevents buffer overflows by design
    input.chars()
        .filter(|c| c.is_ascii_alphanumeric())
        .collect()
}

// Safe array access
pub fn get_element<T>(array: &[T], index: usize) -> Option<&T> {
    // Rust prevents out-of-bounds access
    array.get(index)
}
```

### Security Scanning

**Dependency Scanning**
Use `cargo-audit` to scan for vulnerabilities:

```bash
# Install cargo-audit
cargo install cargo-audit

# Scan for vulnerabilities
cargo audit

# Generate security report
cargo audit --json > security-report.json
```

**Static Analysis**
Use static analysis tools to find security issues:

```bash
# Use clippy for linting
cargo clippy -- -W clippy::all

# Use rust-analyzer for advanced analysis
# Configure in your IDE or editor
```

**Secret Detection**
Use tools to detect secrets in your codebase:

```bash
# Use gitleaks to detect secrets
gitleaks --no-git --verbose

# Use truffleHog for advanced secret detection
trufflehog filesystem --directory .
```

### Secure Configuration

**Environment Variables**
Use environment variables for sensitive configuration:

```rust
use std::env;

pub fn get_database_url() -> Result<String> {
    env::var("DATABASE_URL")
        .map_err(|_| "DATABASE_URL environment variable not set".into())
}

pub fn get_api_key() -> Result<String> {
    env::var("API_KEY")
        .map_err(|_| "API_KEY environment variable not set".into())
}
```

**Configuration Validation**
Validate configuration at startup:

```rust
use serde::Deserialize;
use validator::Validate;

#[derive(Debug, Deserialize, Validate)]
pub struct Config {
    #[validate(url)]
    pub database_url: String,
    
    #[validate(length(min = 32))]
    pub secret_key: String,
    
    #[validate(range(min = 1, max = 65535))]
    pub port: u16,
}

pub fn load_config() -> Result<Config> {
    let config: Config = envy::from_env()?;
    config.validate()?;
    Ok(config)
}
```

### Secure Coding Practices

**Avoid Unsafe Code**
Minimize the use of unsafe code:

```rust
// Avoid this unless absolutely necessary
unsafe fn dangerous_operation() {
    // Potentially unsafe operations
}

// Prefer safe alternatives
fn safe_operation(data: &[u8]) -> Result<&[u8], Error> {
    // Safe implementation
}
```

**Use Secure Libraries**
Use well-vetted security libraries:

```rust
// Use ring for cryptography
use ring::rand::{SecureRandom, SystemRandom};
use ring::digest::{Context, Digest, SHA256};

pub fn hash_data(data: &[u8]) -> Vec<u8> {
    let mut context = Context::new(&SHA256);
    context.update(data);
    let digest = context.finish();
    digest.as_ref().to_vec()
}

// Use rustls for TLS
use rustls::ClientConfig;
use rustls::RootCertStore;

pub fn create_tls_config() -> ClientConfig {
    let mut root_store = RootCertStore::empty();
    root_store.add_server_trust_anchors(webpki_roots::TLS_SERVER_ROOTS.0.iter().map(|ta| {
        rustls::OwnedTrustAnchor::from_subject_spki_name_constraints(
            ta.subject,
            ta.spki,
            ta.name_constraints,
        )
    }));
    
    ClientConfig::builder()
        .with_safe_defaults()
        .with_root_certificates(root_store)
        .with_no_client_auth()
}
```

Next: [Documentation](./rust-practices-08-documentation.md)