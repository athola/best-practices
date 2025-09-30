# Performance Optimization

## Build Profiles

```toml
[profile.dev]
opt-level = 1           # Some optimization in dev
debug = true            # Debug info

[profile.ci]
inherits = "dev"
opt-level = 2           # More optimization for CI
debug = false           # No debug info for CI

[profile.release]
opt-level = 3           # Maximum optimization
lto = true             # Link Time Optimization
codegen-units = 1      # Best optimization
panic = "abort"        # Smaller binaries
```

## Runtime Performance

We encountered a performance issue where post creation times increased linearly with the number of posts being created. Investigation revealed that each post was triggering a separate database call. The solution involved batching these operations:

```rust
pub async fn batch_create_posts(
    db: &SurrealDB,
    posts: Vec<PostData>,
) -> Result<Vec<Post>> {
    let mut results = Vec::new();
    
    // Instead of one query per post, we batch them all together
    let query = posts.iter().map(|post| {
        format!(
            "CREATE post SET title = '{}', content = '{}', created_at = time::now()",
            post.title, post.content
        )
    }).collect::<Vec<_>>().join(";");
    
    let created_posts = db.query(&query).await?;
    results.extend(created_posts);
    
    Ok(results)
}
```

The performance improvement was substantial. Post creation operations that previously caused user complaints now complete efficiently. This experience reinforced the value of examining database interaction patterns.

## Memory Optimization

We observed excessive memory usage during large dataset processing. Monitoring data showed the application consuming gigabytes of RAM for processing tasks that should have required significantly less memory. Profiling identified frequent memory reallocations as the root cause:

```rust
pub fn process_large_dataset(data: &[DataPoint]) -> ProcessedData {
    // Pre-allocate the exact capacity we need
    let mut result = ProcessedData::with_capacity(data.len());
    
    for point in data {
        // Now we can push without reallocations
        result.push(process_point(point));
    }
    
    result
}
```

This optimization reduced memory usage by over fifty percent and improved processing performance considerably.

### Performance Optimization Strategies

**Profile-Driven Optimization**
Always measure before optimizing. Use tools like `perf`, `valgrind`, or Rust's built-in profiling to identify bottlenecks:

```rust
#[cfg(feature = "profiling")]
use profiling::profiler;

#[cfg(feature = "profiling")]
profiler!("process_data");

pub fn process_data(data: &[DataPoint]) -> ProcessedData {
    // Processing logic
}
```

**Zero-Cost Abstractions**
Leverage Rust's zero-cost abstractions to write high-level code that compiles to efficient machine code:

```rust
// High-level iterator chain
let result: Vec<i32> = data
    .iter()
    .filter(|x| x > 0)
    .map(|x| x * 2)
    .collect();

// Compiles to efficient loop similar to:
let mut result = Vec::with_capacity(data.len());
for x in data.iter() {
    if *x > 0 {
        result.push(x * 2);
    }
}
```

**Memory Management**
Use Rust's ownership system to avoid unnecessary allocations:

```rust
// Avoid: Creating new strings
fn process_string(s: &str) -> String {
    s.to_uppercase()
}

// Better: Return references when possible
fn process_string(s: &str) -> &str {
    // Return slice instead of new string
    &s[..s.len()]
}

// Best: Use Cow for conditional ownership
fn process_string(s: &str) -> Cow<str> {
    if s.chars().all(|c| c.is_uppercase()) {
        Cow::Borrowed(s)
    } else {
        Cow::Owned(s.to_uppercase())
    }
}
```

**Concurrent Processing**
Use Rust's concurrency features to improve performance:

```rust
use rayon::prelude::*;

fn process_data_parallel(data: &[DataPoint]) -> Vec<ProcessedData> {
    data.par_iter()
        .map(|point| process_point(point))
        .collect()
}
```

### Performance Monitoring

**Metrics Collection**
Implement comprehensive metrics collection:

```rust
use metrics::{counter, histogram, gauge};

pub fn process_request(request: Request) -> Response {
    let start = std::time::Instant::now();
    
    counter!("requests_total").increment(1);
    gauge!("active_requests").increment(1);
    
    let response = handle_request(request);
    
    let duration = start.elapsed();
    histogram!("request_duration_seconds").record(duration.as_secs_f64());
    gauge!("active_requests").decrement(1);
    
    response
}
```

**Performance Benchmarks**
Use criterion for comprehensive benchmarking:

```rust
use criterion::{criterion_group, criterion_main, Criterion};

fn bench_process_data(c: &mut Criterion) {
    let data = generate_test_data(1000);
    
    c.bench_function("process_data", |b| {
        b.iter(|| process_data(&data))
    });
}

criterion_group!(benches, bench_process_data);
criterion_main!(benches);
```

Next: [Security](./rust-practices-07-security.md)