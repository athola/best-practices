# Testing

## Fast Testing

Our migration test suite was becoming prohibitively slow, with execution times that impacted developer productivity. After exploring several optimization approaches, we developed a custom test framework:

```rust
// tests/harness/mod.rs
pub struct MigrationTestFramework {
    db: SurrealDB,
    migrations: Vec<Migration>,
    cached_files: HashMap<String, String>,
}

impl MigrationTestFramework {
    pub fn new() -> Self {
        // Loading migrations was taking forever, so we cache them
        let migrations = Self::load_migrations_cached();
        Self { /* ... */ }
    }
    
    pub async fn run_migration_chain(&self) -> Result<()> {
        // We batch operations instead of running them one by one
        // This made our tests about 10x faster
    }
}
```

## Test Organization

```bash
tests/
├── harness/           # Custom test framework
│   └── mod.rs
├── server_integration_tests.rs    # Integration tests
├── migration_core_tests.rs       # Migration testing
├── schema_evolution_tests.rs     # Schema validation
└── activity_feed_tests.rs       # Feature testing
```

## Performance Benchmarks

We added some basic performance tests to make sure our optimizations were actually working:

```rust
#[cfg(test)]
mod benchmarks {
    use super::*;
    
    #[test]
    fn test_single_execution_time() {
        let start = std::time::Instant::now();
        // Run a single test
        let duration = start.elapsed();
        // We want this to be fast, under 160ms
        assert!(duration.as_millis() < 160);
    }
    
    #[test]
    fn test_full_suite_performance() {
        let start = std::time::Instant::now();
        // Run the whole test suite
        let duration = start.elapsed();
        // Full suite should be under 350ms
        assert!(duration.as_millis() < 350);
    }
}
```

### Testing Strategies

**Unit Testing**
Focus on testing individual functions and modules in isolation. Rust's built-in test framework makes this straightforward:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_post_creation() {
        let post = Post::new("Test Title", "Test Content");
        assert_eq!(post.title, "Test Title");
        assert_eq!(post.content, "Test Content");
    }
}
```

**Integration Testing**
Test how different parts of your system work together:

```rust
#[cfg(test)]
mod integration_tests {
    use super::*;

    #[tokio::test]
    async fn test_full_post_workflow() {
        // Test the complete workflow from creation to retrieval
        let post = create_test_post().await;
        let retrieved = get_post(&post.id).await;
        assert_eq!(post.title, retrieved.title);
    }
}
```

**Property-Based Testing**
Use libraries like `proptest` to test your code with many different inputs:

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_post_title_validation(title in "\\PC*") {
        let result = Post::validate_title(&title);
        // Test that validation works for all possible strings
        assert!(result.is_ok() || result.is_err());
    }
}
```

**Mock Testing**
Use mocking frameworks to test dependencies:

```rust
#[cfg(test)]
mockall::mock! {
    pub Database {}
    impl DatabaseTrait for Database {
        async fn save_post(&self, post: &Post) -> Result<()>;
        async fn get_post(&self, id: &str) -> Result<Post>;
    }
}
```

Next: [CI/CD Pipeline](./rust-practices-05-cicd.md)