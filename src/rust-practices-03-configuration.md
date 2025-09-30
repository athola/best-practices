# Configuration Management

## Build Profiles

```toml
[profile.wasm-release]
inherits = "release"
opt-level = 'z'        # Optimize for size
lto = true             # Link Time Optimization
codegen-units = 1      # Better optimization

[profile.server]
inherits = "release"
opt-level = 2           # Balance speed and size
lto = "thin"           # Thin LTO for faster builds
```

## Feature Flags

```toml
[features]
default = ["ssr"]
ssr = ["leptos/ssr", "leptos_axum/ssr"]
csr = ["leptos/csr", "leptos_axum/csr"]
hydrate = ["csr", "leptos/hydrate"]
```

## Environment Configuration

```bash
.env.example          # Template with documentation
.env                  # Local development
.env.production       # Production overrides
```

### Configuration Best Practices

**Externalize Configuration**
Configuration should be external to the code, not hardcoded. This makes the system more flexible and easier to deploy in different environments.

**Environment-Specific Configuration**
Different environments (development, staging, production) have different needs. Use environment-specific configuration files to manage these differences.

**Secure Configuration**
Sensitive information (API keys, passwords, certificates) should never be stored in configuration files. Use secure secret management instead.

**Configuration Validation**
Validate configuration at startup to catch issues early rather than at runtime when they can cause more damage.

### Configuration Patterns

**Hierarchical Configuration**
```bash
# Base configuration
config/base.toml

# Environment-specific overrides
config/development.toml
config/staging.toml
config/production.toml

# Local overrides (not committed)
config/local.toml
```

**Feature Flags**
```bash
# Feature flag configuration
config/features.toml

[features]
new_user_interface = true
advanced_analytics = false
beta_payment_processor = true
```

**Configuration Schema**
```bash
# Configuration schema validation
config/schema.json

{
  "type": "object",
  "properties": {
    "database": {
      "type": "object",
      "properties": {
        "host": {"type": "string"},
        "port": {"type": "integer", "minimum": 1, "maximum": 65535},
        "name": {"type": "string"}
      },
      "required": ["host", "port", "name"]
    }
  }
}
```

Next: [Testing](./rust-practices-04-testing.md)