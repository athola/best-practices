# Structural Design Patterns

## Architectural Patterns

### Layered Architecture

Layered architecture organizes code into horizontal layers where each layer has a specific responsibility and communicates only with adjacent layers.

**Common Layers:**
- **Presentation Layer**: Handles user interface and interaction
- **Business Logic Layer**: Contains core business rules and processes
- **Data Access Layer**: Manages data storage and retrieval

**Example Structure:**
```
src/
├── presentation/    # UI components, controllers, API endpoints
├── business/        # Business logic, services, domain models
├── data/           # Database access, repositories, external APIs
└── shared/         # Common utilities, types, constants
```

**When to Use:**
- Projects with clear separation of concerns
- Teams that benefit from well-defined boundaries
- Applications where different layers evolve at different rates

**Benefits:**
- Clear separation of concerns
- Easier to test individual layers
- Promotes reusability within layers

**Challenges:**
- Can lead to "pass-through" code
- May create unnecessary abstraction
- Performance overhead from layer boundaries

### Hexagonal Architecture

Hexagonal architecture (ports and adapters) focuses on separating core business logic from external concerns like databases, UI, and external services.

**Core Concepts:**
- **Domain Core**: Pure business logic without external dependencies
- **Ports**: Interfaces defining interactions with the outside world
- **Adapters**: Implementations that connect to external systems

**Example Structure:**
```
src/
├── domain/         # Core business logic and entities
├── ports/          # Interfaces for external interactions
├── adapters/       # Implementations for external systems
│   ├── database/   # Database adapters
│   ├── web/        # Web framework adapters
│   └── external/   # External service adapters
└── infrastructure/ # Configuration and setup
```

**When to Use:**
- Complex business domains
- Projects requiring testability of core logic
- Systems that need to support multiple UIs or data sources

**Benefits:**
- Core business logic is isolated and testable
- Easy to swap external dependencies
- Promotes domain-driven design

**Challenges:**
- Initial complexity is higher
- May be overkill for simple applications
- Requires discipline to maintain boundaries

### Clean Architecture

Clean architecture extends hexagonal architecture with more explicit dependency rules and layer definitions.

**Dependency Rule:** Source code dependencies can only point inward, toward higher-level policies.

**Layer Structure:**
```
src/
├── entities/       # Core business entities and rules
├── use_cases/      # Application-specific business rules
├── interface_adapters/ # Convert data between layers
│   ├── controllers/
│   ├── presenters/
│   └── gateways/
└── frameworks_drivers/ # External frameworks and tools
    ├── web/
    ├── database/
    └── external/
```

**When to Use:**
- Enterprise applications with complex business rules
- Projects requiring long-term maintainability
- Systems where business logic changes independently of technology

**Benefits:**
- Maximum flexibility for technology choices
- Business logic is completely isolated
- Excellent testability

**Challenges:**
- Significant boilerplate code
- Learning curve for teams
- May slow down initial development

## Design Trade-offs

### Predictability vs Flexibility

**Predictability Focus:**
- Strict architectural patterns
- Comprehensive documentation
- Formal change processes
- Extensive testing requirements

**When to Choose:**
- Safety-critical systems
- Regulatory compliance requirements
- Large teams with high turnover
- Projects with stable requirements

**Flexibility Focus:**
- Emergent architecture
- Minimal upfront design
- Rapid prototyping
- Iterative refinement

**When to Choose:**
- Startup environments
- Research and development
- Projects with rapidly changing requirements
- Small, experienced teams

**Balanced Approach:**
- Start with flexibility for unknown areas
- Add predictability as patterns emerge
- Use architectural fitness functions
- Regular architecture reviews

### Simplicity vs Completeness

**Simplicity Benefits:**
- Faster onboarding
- Lower maintenance costs
- Easier debugging
- Reduced cognitive load

**Completeness Benefits:**
- Handles edge cases
- Supports all requirements
- Provides comprehensive solutions
- Reduces future rework

**Finding the Balance:**
- Start simple, add complexity only when needed
- Use YAGNI (You Ain't Gonna Need It) principle
- Implement the simplest thing that works
- Refactor toward completeness as requirements solidify

### Performance vs Maintainability

**Performance Focus:**
- Optimized data structures
- Minimal abstraction layers
- Caching strategies
- Parallel processing

**Maintainability Focus:**
- Clear abstractions
- Comprehensive testing
- Documentation
- Modular design

**Trade-off Strategies:**
- Profile before optimizing
- Optimize critical paths only
- Use abstraction layers that can be bypassed
- Document performance-critical sections

## Evolution Strategies

### Emergent Structure

Emergent structure allows architecture to evolve naturally based on actual usage patterns rather than upfront design.

**Principles:**
- Start with the simplest structure
- Let architecture emerge from use
- Refactor when patterns become clear
- Avoid premature abstraction

**Process:**
1. **Initial Structure**: Simple, flat organization
2. **Usage Observation**: Monitor how code is used and changed
3. **Pattern Identification**: Look for recurring changes and dependencies
4. **Gradual Refactoring**: Restructure based on observed patterns

**Benefits:**
- Architecture fits actual needs
- Less upfront design time
- Adapts to changing requirements
- Avoids over-engineering

**Challenges:**
- Requires regular refactoring
- Can lead to technical debt if neglected
- May require architectural reviews
- Needs experienced team members

### Refactoring-Driven Design

Refactoring-driven design uses the need for refactoring as a signal for architectural improvements.

**Triggers for Refactoring:**
- Code duplication across modules
- High coupling between components
- Difficulty in testing
- Changes affecting multiple areas
- Performance bottlenecks

**Refactoring Patterns:**
- **Extract Module**: Move related functionality to new modules
- **Introduce Abstraction**: Add interfaces to reduce coupling
- **Consolidate Responsibilities**: Merge related modules
- **Separate Concerns**: Split modules with multiple responsibilities

**Process:**
1. **Identify Pain Points**: Look for areas that are hard to change
2. **Design Target Structure**: Define the desired organization
3. **Incremental Migration**: Move code gradually
4. **Validate**: Ensure functionality is preserved

### Architectural Fitness Functions

Architectural fitness functions are objective criteria that measure how well the architecture meets its goals.

**Types of Fitness Functions:**
- **Structural Metrics**: Cyclomatic complexity, coupling, cohesion
- **Performance Metrics**: Response time, throughput, resource usage
- **Quality Metrics**: Test coverage, code duplication, maintainability
- **Business Metrics**: Feature delivery time, defect rates

**Implementation:**
```yaml
# Example fitness function configuration
fitness_functions:
  structural:
    max_complexity: 10
    max_coupling: 5
    min_cohesion: 0.7
  performance:
    max_response_time: 500ms
    min_throughput: 1000 req/s
  quality:
    min_test_coverage: 80%
    max_duplication: 5%
```

**Usage:**
- Integrate with CI/CD pipeline
- Run on every commit
- Set thresholds for alerts
- Track trends over time

## Structural Heuristics

### Single Responsibility Principle

Each module should have one, and only one, reason to change.

**Application to Project Structure:**
- Modules should focus on a single feature or concern
- Avoid modules that handle multiple unrelated responsibilities
- Group related functionality together
- Separate different types of concerns

**Examples:**
```
# Good: Single responsibility
src/
├── user_management/  # All user-related functionality
├── order_processing/ # All order-related functionality
└── reporting/        # All reporting functionality

# Bad: Multiple responsibilities
src/
├── misc/             # Mixed unrelated functionality
├── utils/            # Too broad, no clear focus
└── common/           # Vague, contains everything
```

**Benefits:**
- Easier to understand and maintain
- Reduces impact of changes
- Improves testability
- Promotes reusability

### Common Closure Principle

Gather into modules those components that change for the same reasons and at the same time.

**Key Concepts:**
- Components that change together should belong together
- Minimize the number of modules affected by a single change
- Group code by change frequency and reason

**Example:**
```
# Grouped by change frequency
src/
├── core/            # Rarely changes, stable business rules
├── features/        # Changes with business requirements
├── integrations/    # Changes with external systems
└── ui/              # Changes with user interface requirements
```

**Benefits:**
- Reduces coordination overhead
- Minimizes deployment scope
- Improves team autonomy
- Simplifies dependency management

### Stable Dependencies Principle

Depend in the direction of stability. Less stable components should depend on more stable components.

**Stability Factors:**
- **Change Frequency**: How often the component changes
- **Number of Dependencies**: More dependencies = less stable
- **Importance**: Critical components should be more stable
- **Maturity**: Older, well-tested components tend to be more stable

**Stability Hierarchy:**
```
# Most stable (bottom)
src/
├── domain/          # Core business entities, rarely changes
├── infrastructure/   # Technical foundations, stable
├── services/        # Business services, moderately stable
└── controllers/     # UI/API layer, least stable
```

**Implementation:**
- Abstract unstable components behind interfaces
- Use dependency inversion
- Place stable components at the bottom
- Minimize dependencies on volatile components

### Stable Abstractions Principle

Stable components should be abstract. Unstable components should be concrete.

**Key Concepts:**
- Stable components should be defined by abstractions (interfaces)
- Concrete implementations can change without affecting stable abstractions
- Abstractions should be more stable than their implementations

**Example:**
```
# Stable abstractions
src/
├── domain/
│   ├── interfaces/   # Abstract definitions
│   │   ├── repository.rs
│   │   └── service.rs
│   └── entities/    # Core domain objects
└── infrastructure/
    ├── database/    # Concrete implementations
    └── external/    # External service implementations
```

**Benefits:**
- Stable components can evolve without breaking changes
- Concrete implementations can be replaced easily
- Promotes loose coupling
- Improves testability through mocking

## Practical Application

### Choosing the Right Pattern

**Decision Factors:**
1. **Project Size**: Small projects need simpler patterns
2. **Team Size**: Larger teams benefit from more structure
3. **Domain Complexity**: Complex domains need more sophisticated architecture
4. **Expected Lifetime**: Long-term projects benefit from more investment in architecture
5. **Performance Requirements**: High performance may require trade-offs with purity

**Decision Matrix:**
| Factor | Layered | Hexagonal | Clean | Emergent |
|--------|---------|-----------|-------|----------|
| Small Project | ✓ | ✗ | ✗ | ✓✓ |
| Large Team | ✓✓ | ✓ | ✓ | ✗ |
| Complex Domain | ✓ | ✓✓ | ✓✓ | ✗ |
| Rapid Changes | ✗ | ✓ | ✓ | ✓✓ |
| High Performance | ✓ | ✗ | ✗ | ✓ |

### Evolution Path

**Typical Evolution:**
1. **Start Simple**: Flat structure, minimal organization
2. **Add Layers**: As complexity grows, add technical layers
3. **Introduce Abstractions**: When multiple implementations emerge
4. **Adopt Patterns**: Apply formal patterns when benefits outweigh costs
5. **Refine Continuously**: Regular architectural improvements

**Migration Strategies:**
- **Strangler Fig Pattern**: Gradually replace old structure with new
- **Parallel Development**: Build new structure alongside old
- **Incremental Refactoring**: Change one module at a time
- **Architectural Sprints**: Dedicated time for structural improvements

### Measuring Success

**Success Metrics:**
- **Development Velocity**: Time to implement new features
- **Defect Rates**: Number of bugs related to structural issues
- **Code Quality Metrics**: Complexity, coupling, cohesion
- **Team Satisfaction**: Developer surveys and feedback
- **Maintenance Costs**: Time spent on structural issues

**Continuous Improvement:**
- Regular architecture reviews
- Automated fitness functions
- Team retrospectives
- Documentation updates

Next: [Growth Considerations](./project-structure-04-growth.md)