# Project Structure

## Workspace Organization

Workspaces help manage multi-crate projects by ensuring consistent versions and reducing build times. Here's what worked well in a production blog project:

Filename: Cargo.toml
```toml
[workspace]
members = [
    "app",
    "frontend", 
    "markdown",
    "server",
]
resolver = "2"

[workspace.dependencies]
leptos = { version = "0.2", features = ["csr", "nightly"] }
leptos_axum = { version = "0.2", features = ["csr", "nightly"] }
leptos_meta = { version = "0.2", features = ["csr", "nightly"] }
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
```

This workspace setup resolved several persistent issues:

**Problem**: Version conflicts between crates were becoming increasingly difficult to debug, with one incident requiring half a day to trace a serde version mismatch through transitive dependencies.

**Solution**: The workspace configuration eliminated these conflicts entirely by ensuring all crates use the same dependency versions.

**Additional Benefit**: Build time reduction. The CI pipeline duration decreased from eight to five minutes, which significantly improved developer productivity during code review cycles.

Each crate in the workspace can then use the workspace dependencies:

Filename: app/Cargo.toml
```toml
[package]
name = "app"
version = "0.1.0"
edition = "2021"

[dependencies]
leptos = { workspace = true }
tokio = { workspace = true }
serde = { workspace = true }
```

## Crate Organization

Choosing the right organization structure is crucial for maintainability. Here's a feature-based organization that worked well:

Filename: src/lib.rs
```rust
mod components;
mod activity;
mod api;
mod contact;
mod home;
mod post;
mod references;
mod types;

pub use components::*;
pub use activity::*;
pub use api::*;
pub use contact::*;
pub use home::*;
pub use post::*;
pub use references::*;
pub use types::*;
```

The directory structure looks like this:

```bash
src/
├── components/      # Reusable UI components
│   ├── mod.rs
│   ├── header.rs
│   ├── icons.rs
│   └── loader.rs
├── activity.rs     # Activity feed functionality
├── api.rs          # API endpoints and server functions
├── contact.rs      # Contact form handling
├── home.rs         # Home page logic
├── post.rs         # Blog post management
├── references.rs    # Reference management
├── types.rs        # Shared type definitions
└── lib.rs          # Library entry point
```

**Technical Layer vs Feature-Based Organization**

Our initial approach used technical layer organization:

```bash
src/
├── models/         # Data structures
├── handlers/       # API handlers
├── services/       # Business logic
└── utils/          # Utility functions
```

**Problem**: Simple feature modifications required changes across multiple directories, increasing cognitive load.

**Solution**: Switch to feature-based organization to reduce context switching.

**Benefits**: Feature-related code is now co-located, making maintenance and debugging more efficient.

**The types.rs Pattern**

The `types.rs` file emerged from a specific problem:

**Problem**: We encountered subtle bugs when similar data structures diverged across different modules.

**Solution**: Centralize type definitions in `types.rs`:

Filename: src/types.rs
```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Post {
    pub id: String,
    pub title: String,
    pub content: String,
    pub created_at: chrono::DateTime<chrono::Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct User {
    pub id: String,
    pub name: String,
    pub email: String,
}
```

**Result**: Centralizing type definitions eliminated synchronization issues, though we initially had concerns about creating an overly large module.

Next: [Configuration Management](./rust-practices-03-configuration.md)