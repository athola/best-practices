# Error Handling

## Structured Error Types

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum BlogError {
    #[error("Database error: {0}")]
    Database(#[from] surrealdb::Error),
    
    #[error("Post not found: {slug}")]
    PostNotFound { slug: String },
    
    #[error("Validation error: {field} - {message}")]
    Validation { field: String, message: String },
    
    #[error("Authentication error: {0}")]
    Authentication(String),
    
    #[error("Authorization error: {0}")]
    Authorization(String),
    
    #[error("Internal server error: {0}")]
    Internal(String),
}

impl BlogError {
    /// Convert error to HTTP status code
    pub fn status_code(&self) -> StatusCode {
        match self {
            BlogError::PostNotFound { .. } => StatusCode::NOT_FOUND,
            BlogError::Validation { .. } => StatusCode::BAD_REQUEST,
            BlogError::Authentication(_) => StatusCode::UNAUTHORIZED,
            BlogError::Authorization(_) => StatusCode::FORBIDDEN,
            _ => StatusCode::INTERNAL_SERVER_ERROR,
        }
    }
    
    /// Convert error to user-friendly message
    pub fn user_message(&self) -> String {
        match self {
            BlogError::PostNotFound { slug } => format!("Post '{}' not found", slug),
            BlogError::Validation { field, message } => format!("{}: {}", field, message),
            BlogError::Authentication(_) => "Authentication required".to_string(),
            BlogError::Authorization(_) => "Access denied".to_string(),
            _ => "An internal error occurred".to_string(),
        }
    }
}
```

## Result Type Aliases

```rust
/// Type alias for Result with BlogError
pub type BlogResult<T> = Result<T, BlogError>;

/// Type alias for database operations
pub type DbResult<T> = Result<T, surrealdb::Error>;

/// Type alias for API responses
pub type ApiResponse<T> = Result<Json<T>, BlogError>;
```

### Error Handling Strategies

**Early Returns**
Use the `?` operator for early returns:

```rust
pub fn create_post(title: &str, content: &str) -> BlogResult<Post> {
    let validated_title = validate_title(title)?;
    let validated_content = validate_content(content)?;
    
    let post = Post::new(validated_title, validated_content);
    save_post(&post)?;
    
    Ok(post)
}
```

**Error Context**
Add context to errors using `map_err` or `context`:

```rust
use anyhow::Context;

pub fn process_file(path: &Path) -> Result<(), BlogError> {
    let content = std::fs::read_to_string(path)
        .context("Failed to read file")?;
    
    let parsed = parse_content(&content)
        .context("Failed to parse content")?;
    
    save_to_database(&parsed)
        .context("Failed to save to database")?;
    
    Ok(())
}
```

**Error Conversion**
Convert between error types using `From` trait:

```rust
impl From<serde_json::Error> for BlogError {
    fn from(err: serde_json::Error) -> Self {
        BlogError::Validation {
            field: "json".to_string(),
            message: err.to_string(),
        }
    }
}

impl From<std::io::Error> for BlogError {
    fn from(err: std::io::Error) -> Self {
        BlogError::Internal(format!("IO error: {}", err))
    }
}
```

### Error Recovery

**Fallback Values**
Provide fallback values when errors occur:

```rust
pub fn get_config() -> Config {
    get_config_from_file()
        .unwrap_or_else(|_| get_default_config())
}

pub fn get_user_preference(user_id: &str, key: &str) -> String {
    get_user_preference_from_db(user_id, key)
        .unwrap_or_else(|_| get_default_preference(key))
}
```

**Retry Logic**
Implement retry logic for transient errors:

```rust
pub async fn fetch_with_retry<F, Fut, T, E>(
    mut operation: F,
    max_retries: usize,
) -> Result<T, E>
where
    F: FnMut() -> Fut,
    Fut: Future<Output = Result<T, E>>,
    E: std::fmt::Debug,
{
    let mut retries = 0;
    
    loop {
        match operation().await {
            Ok(result) => return Ok(result),
            Err(e) if retries < max_retries => {
                retries += 1;
                tokio::time::sleep(Duration::from_millis(100 * retries as u64)).await;
                continue;
            }
            Err(e) => return Err(e),
        }
    }
}
```

**Circuit Breaker**
Implement circuit breaker pattern for external services:

```rust
pub struct CircuitBreaker {
    failure_count: u32,
    last_failure_time: Instant,
    state: CircuitState,
}

#[derive(Debug, Clone, Copy)]
enum CircuitState {
    Closed,    // Normal operation
    Open,      // Failing fast
    HalfOpen,  // Testing recovery
}

impl CircuitBreaker {
    pub async fn call<F, Fut, T, E>(&mut self, operation: F) -> Result<T, E>
    where
        F: FnOnce() -> Fut,
        Fut: Future<Output = Result<T, E>>,
    {
        match self.state {
            CircuitState::Open => {
                if self.last_failure_time.elapsed() > Duration::from_secs(60) {
                    self.state = CircuitState::HalfOpen;
                } else {
                    return Err("Circuit breaker is open".into());
                }
            }
            CircuitState::Closed | CircuitState::HalfOpen => {}
        }
        
        match operation().await {
            Ok(result) => {
                self.failure_count = 0;
                self.state = CircuitState::Closed;
                Ok(result)
            }
            Err(e) => {
                self.failure_count += 1;
                self.last_failure_time = Instant::now();
                
                if self.failure_count >= 5 {
                    self.state = CircuitState::Open;
                }
                
                Err(e)
            }
        }
    }
}
```

### Error Logging

**Structured Logging**
Use structured logging for better error tracking:

```rust
use tracing::{error, warn, info, debug};

pub fn process_request(request: Request) -> Result<Response, BlogError> {
    debug!("Processing request: {:?}", request);
    
    match handle_request(request) {
        Ok(response) => {
            info!("Request processed successfully");
            Ok(response)
        }
        Err(e) => {
            error!(
                error = %e,
                error_type = %std::mem::discriminant(&e),
                "Failed to process request"
            );
            Err(e)
        }
    }
}
```

**Error Metrics**
Track error rates and types:

```rust
use metrics::{counter, histogram};

pub fn process_with_metrics<T>(operation: impl FnOnce() -> Result<T, BlogError>) -> Result<T, BlogError> {
    let start = std::time::Instant::now();
    
    match operation() {
        Ok(result) => {
            let duration = start.elapsed();
            histogram!("operation_duration_seconds").record(duration.as_secs_f64());
            counter!("operation_success_total").increment(1);
            Ok(result)
        }
        Err(e) => {
            let duration = start.elapsed();
            histogram!("operation_duration_seconds").record(duration.as_secs_f64());
            counter!("operation_error_total").increment(1);
            counter!("operation_error_total", "error_type" => format!("{:?}", std::mem::discriminant(&e))).increment(1);
            Err(e)
        }
    }
}
```

### Error Testing

**Error Cases in Tests**
Test error handling explicitly:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_create_post_validation_error() {
        let result = create_post("", "content");
        assert!(matches!(result, Err(BlogError::Validation { .. })));
    }

    #[test]
    fn test_get_post_not_found() {
        let result = get_post("nonexistent-post");
        assert!(matches!(result, Err(BlogError::PostNotFound { .. })));
    }

    #[test]
    fn test_error_status_codes() {
        let not_found_error = BlogError::PostNotFound { slug: "test".to_string() };
        assert_eq!(not_found_error.status_code(), StatusCode::NOT_FOUND);
        
        let validation_error = BlogError::Validation {
            field: "title".to_string(),
            message: "Title is required".to_string(),
        };
        assert_eq!(validation_error.status_code(), StatusCode::BAD_REQUEST);
    }

    #[test]
    fn test_error_user_messages() {
        let not_found_error = BlogError::PostNotFound { slug: "test-post".to_string() };
        assert_eq!(not_found_error.user_message(), "Post 'test-post' not found");
        
        let validation_error = BlogError::Validation {
            field: "title".to_string(),
            message: "Title is required".to_string(),
        };
        assert_eq!(validation_error.user_message(), "title: Title is required");
    }
}
```

**Property-Based Testing for Errors**
Use property-based testing to verify error handling:

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_validate_title_handles_all_strings(title in "\\PC*") {
        let result = validate_title(&title);
        
        // Either it succeeds or fails with a validation error
        match result {
            Ok(_) => {
                // Valid title should not be empty
                assert!(!title.is_empty());
                assert!(title.len() <= 100);
            }
            Err(BlogError::Validation { .. }) => {
                // Invalid title should be empty or too long
                assert!(title.is_empty() || title.len() > 100);
            }
            Err(_) => {
                // Should not get other error types
                panic!("Unexpected error type");
            }
        }
    }
}
```

Next: [Async Patterns](./rust-practices-10-async.md)