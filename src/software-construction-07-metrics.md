# Complexity Metrics and Heuristics

Complexity metrics and heuristics provide quantitative and qualitative measures to assess code quality, identify potential problems, and guide refactoring efforts. These tools help developers maintain code quality, manage technical debt, and make informed decisions about software construction and maintenance.

## Understanding Code Complexity Metrics

### Static Complexity Metrics
Static metrics analyze code structure without execution:

**Cyclomatic Complexity**
- **Definition**: Measures the number of linearly independent paths through code
- **Calculation**: M = E - N + 2P, where E = edges, N = nodes, P = connected components
- **Interpretation**: Higher values indicate more complex control flow
- **Thresholds**: 1-10 (simple), 11-20 (moderate), 21-50 (complex), 50+ (very complex)

**Lines of Code (LOC)**
- **Definition**: Counts the number of lines in source code
- **Types**: Physical LOC (all lines), Logical LOC (executable statements), Comment LOC
- **Interpretation**: Larger modules tend to be more complex and harder to maintain
- **Guidelines**: Methods < 50 lines, classes < 500 lines, files < 2000 lines

**Halstead Complexity Measures**
- **Definition**: Based on operators and operands in the code
- **Components**: 
  - η1 = number of unique operators
  - η2 = number of unique operands
  - N1 = total operator occurrences
  - N2 = total operand occurrences
- **Metrics**: Program length, vocabulary, volume, difficulty, effort
- **Interpretation**: Higher values indicate more complex code

**Maintainability Index (MI)**
- **Definition**: Composite measure of code maintainability
- **Factors**: Cyclomatic complexity, lines of code, Halstead volume
- **Scale**: 0-100, higher values indicate better maintainability
- **Interpretation**: 85-100 (good), 70-84 (moderate), 0-69 (poor)

### Dynamic Complexity Metrics
Dynamic metrics analyze code behavior during execution:

**Execution Complexity**
- **Definition**: Measures complexity based on runtime behavior
- **Metrics**: Execution time, memory usage, CPU cycles, I/O operations
- **Interpretation**: Higher values indicate performance bottlenecks
- **Use case**: Performance optimization, resource management

**Path Complexity**
- **Definition**: Measures the number of actual execution paths
- **Calculation**: Based on test coverage and execution traces
- **Interpretation**: Higher values indicate more complex runtime behavior
- **Use case**: Testing strategy, risk assessment

**Data Flow Complexity**
- **Definition**: Measures complexity based on data flow patterns
- **Metrics**: Variable definitions, uses, data dependencies
- **Interpretation**: Higher values indicate more complex data relationships
- **Use case**: Refactoring, optimization, testing

### Architectural Metrics
Architectural metrics assess system-level complexity:

**Coupling Metrics**
- **Definition**: Measures interdependence between modules
- **Types**: 
  - Data coupling (shared data)
  - Control coupling (one module controls another)
  - Common coupling (shared global data)
  - Content coupling (one module accesses another's internal data)
- **Interpretation**: Lower coupling is better for maintainability

**Cohesion Metrics**
- **Definition**: Measures how closely related elements within a module are
- **Types**: 
  - Functional cohesion (elements perform one well-defined task)
  - Sequential cohesion (elements related by sequence)
  - Communicational cohesion (elements operate on same data)
  - Procedural cohesion (elements related by control flow)
- **Interpretation**: Higher cohesion is better for maintainability

**Depth of Inheritance**
- **Definition**: Measures the length of inheritance hierarchies
- **Calculation**: Number of levels in inheritance tree
- **Interpretation**: Deeper hierarchies are harder to understand and modify
- **Guidelines**: Keep inheritance depth < 6 levels

## Heuristics for Code Quality

### Code Smell Heuristics
Identify common code smells that indicate problems:

**Long Method**
- **Description**: Methods that are too long and do too much
- **Indicators**: Method length > 50 lines, multiple responsibilities
- **Problems**: Hard to understand, test, and maintain
- **Refactoring**: Extract Method, Replace Method with Method Object

**Large Class**
- **Description**: Classes that are too large and have too many responsibilities
- **Indicators**: Class size > 500 lines, too many methods or fields
- **Problems**: Hard to understand, violates Single Responsibility Principle
- **Refactoring**: Extract Class, Extract Subclass, Extract Interface

**Duplicated Code**
- **Description**: Code that appears in multiple places
- **Indicators**: Similar code blocks, repeated logic
- **Problems**: Maintenance burden, inconsistency risk
- **Refactoring**: Extract Method, Extract Class, Template Method

**Long Parameter List**
- **Description**: Methods with too many parameters
- **Indicators**: More than 3-4 parameters
- **Problems**: Hard to use, understand, and maintain
- **Refactoring**: Introduce Parameter Object, Preserve Whole Object

**Feature Envy**
- **Description**: Methods that seem more interested in another class
- **Indicators**: Method uses more data from other classes than its own
- **Problems**: Violates encapsulation, creates tight coupling
- **Refactoring**: Move Method, Extract Method

### Design Heuristics
Heuristics for good object-oriented design:

**Single Responsibility Principle (SRP)**
- **Heuristic**: A class should have only one reason to change
- **Indicators**: Class has multiple unrelated responsibilities
- **Problems**: Hard to maintain, test, and understand
- **Refactoring**: Extract Class, Extract Interface

**Open/Closed Principle (OCP)**
- **Heuristic**: Software entities should be open for extension but closed for modification
- **Indicators**: Need to modify existing code to add new functionality
- **Problems**: Risk of breaking existing functionality
- **Refactoring**: Use abstraction, polymorphism, design patterns

**Liskov Substitution Principle (LSP)**
- **Heuristic**: Subtypes must be substitutable for their base types
- **Indicators**: Subclasses behave differently than expected
- **Problems**: Violates polymorphism, creates bugs
- **Refactoring**: Redesign inheritance hierarchy, use composition

**Interface Segregation Principle (ISP)**
- **Heuristic**: Clients should not depend on interfaces they don't use
- **Indicators**: Classes implement methods they don't need
- **Problems**: Unnecessary dependencies, coupling
- **Refactoring**: Split interfaces, role interfaces

**Dependency Inversion Principle (DIP)**
- **Heuristic**: Depend on abstractions, not concretions
- **Indicators**: High-level modules depend on low-level modules
- **Problems**: Tight coupling, hard to test and maintain
- **Refactoring**: Dependency injection, abstract factories

### Architectural Heuristics
Heuristics for system-level design:

**High Cohesion, Low Coupling**
- **Heuristic**: Modules should be highly cohesive and loosely coupled
- **Indicators**: Modules have clear responsibilities and minimal dependencies
- **Problems**: Systems become hard to understand and maintain
- **Refactoring**: Define clear boundaries, use interfaces

**Stable Dependencies Principle**
- **Heuristic**: Depend in the direction of stability
- **Indicators**: Unstable modules depend on stable modules
- **Problems**: Unstable changes propagate through the system
- **Refactoring**: Stabilize dependencies, use abstraction layers

**Stable Abstractions Principle**
- **Heuristic**: Stable modules should be abstract
- **Indicators**: Concrete modules are stable and hard to change
- **Problems**: System becomes rigid and hard to evolve
- **Refactoring**: Make stable modules more abstract

**Acyclic Dependencies Principle**
- **Heuristic**: The dependency graph should have no cycles
- **Indicators**: Circular dependencies between modules
- **Problems**: Modules become tightly coupled, hard to test
- **Refactoring**: Break cycles with abstraction, dependency inversion

## Refactoring Heuristics

### When to Refactor
Heuristics for identifying refactoring opportunities:

**Code Quality Indicators**
- **High complexity**: Cyclomatic complexity > 20
- **Large methods**: Methods > 50 lines
- **Large classes**: Classes > 500 lines
- **Deep nesting**: Nesting > 4 levels
- **Many parameters**: Methods with > 4 parameters

**Maintenance Indicators**
- **Frequent changes**: Code that changes frequently
- **Bug-prone areas**: Code with many bug fixes
- **Hard to understand**: Code that takes time to understand
- **Hard to test**: Code that's difficult to test
- **Duplication**: Repeated code patterns

**Performance Indicators**
- **Slow execution**: Methods that take too long
- **High memory usage**: Code that uses excessive memory
- **Frequent garbage collection**: Code that creates many objects
- **I/O bottlenecks**: Code that does excessive I/O operations

### Refactoring Strategies
Strategies for effective refactoring:

**Incremental Refactoring**
- **Approach**: Make small, incremental changes
- **Benefits**: Lower risk, easier to verify, continuous improvement
- **Process**: Identify problem, make small change, test, commit, repeat
- **Best for**: Large codebases, ongoing maintenance

**Branch by Abstraction**
- **Approach**: Create abstraction layer, implement new behavior behind abstraction
- **Benefits**: Allows gradual migration, reduces risk, enables parallel work
- **Process**: Create abstraction, implement new behavior, switch to new implementation
- **Best for**: Large architectural changes, framework migrations

**Strangler Fig Pattern**
- **Approach**: Gradually replace old system with new system
- **Benefits**: Incremental replacement, continuous delivery, risk reduction
- **Process**: Identify edge, create new implementation, redirect traffic, remove old
- **Best for**: System replacement, legacy modernization

**Refactoring to Patterns**
- **Approach**: Refactor code to use design patterns
- **Benefits**: Better design, improved maintainability, common vocabulary
- **Process**: Identify pattern opportunity, apply pattern, test, document
- **Best for**: Improving design, reducing complexity, enhancing maintainability

### Refactoring Techniques
Specific techniques for common problems:

**Extract Method**
- **When**: Method is too long or has multiple responsibilities
- **How**: Extract part of method into new method
- **Benefits**: Each method has single responsibility, easier to understand
- **Example**: Extract validation logic from a complex business method

**Extract Class**
- **When**: Class has too many responsibilities
- **How**: Extract related functionality into new class
- **Benefits**: Each class has single responsibility, better cohesion
- **Example**: Extract validation logic into a separate validator class

**Replace Conditional with Polymorphism**
- **When**: Complex conditional logic based on type
- **How**: Replace conditionals with polymorphic method calls
- **Benefits**: Eliminates complex conditionals, easier to extend
- **Example**: Replace type-based conditionals with strategy pattern

**Introduce Parameter Object**
- **When**: Method has many parameters
- **How**: Group related parameters into a parameter object
- **Benefits**: Reduces parameter count, improves readability
- **Example**: Create a UserPreferences object instead of multiple parameters

## Tools and Automation

### Static Analysis Tools
Tools that analyze code structure:

**Language-Specific Tools**
- **Java**: PMD, Checkstyle, SpotBugs, SonarQube
- **Python**: Pylint, Flake8, Bandit, SonarQube
- **JavaScript**: ESLint, JSHint, JSCS, SonarQube
- **C#**: StyleCop, FxCop, SonarQube
- **C++**: Cppcheck, Clang-Tidy, SonarQube

**Multi-Language Tools**
- **SonarQube**: Comprehensive code quality platform
- **CodeClimate**: Automated code review tool
- **Codacy**: Automated code review and analysis
- **DeepSource**: Static analysis and code review

**IDE Integration**
- **Visual Studio**: Built-in code analysis, extensions
- **IntelliJ IDEA**: Built-in inspections, plugins
- **VS Code**: Extensions for various languages
- **Eclipse**: Built-in tools, plugins marketplace

### Dynamic Analysis Tools
Tools that analyze code behavior:

**Profiling Tools**
- **Java**: VisualVM, YourKit, JProfiler
- **Python**: cProfile, line_profiler, memory_profiler
- **JavaScript**: Chrome DevTools, Node.js profiler
- **C#**: dotTrace, ANTS Performance Profiler
- **C++**: Valgrind, gprof, perf

**Memory Analysis Tools**
- **Java**: Eclipse MAT, VisualVM, YourKit
- **Python**: objgraph, pympler, memory_profiler
- **JavaScript**: Chrome DevTools Memory tab
- **C#**: dotMemory, ANTS Memory Profiler
- **C++**: Valgrind, AddressSanitizer, LeakSanitizer

**Testing Tools**
- **Unit Testing**: JUnit, pytest, Jest, NUnit
- **Integration Testing**: TestContainers, Selenium, Cypress
- **Performance Testing**: JMeter, Gatling, k6
- **Code Coverage**: JaCoCo, Coverage.py, Istanbul

### Continuous Integration
Integrate complexity analysis into CI/CD:

**Quality Gates**
- **Definition**: Criteria that code must meet to be merged
- **Examples**: Test coverage > 80%, no critical issues, complexity < 20
- **Implementation**: CI pipeline checks, automated reporting
- **Benefits**: Consistent quality, early problem detection

**Automated Reporting**
- **Metrics dashboards**: Visualize complexity trends over time
- **Trend analysis**: Track complexity changes and identify patterns
- **Alerting**: Notify teams when complexity exceeds thresholds
- **Historical data**: Maintain history for analysis and planning

**Integration Strategies**
- **Pre-commit hooks**: Check complexity before commits
- **Pull request checks**: Analyze changes in pull requests
- **Scheduled analysis**: Regular analysis of entire codebase
- **On-demand analysis**: Manual analysis when needed

## Complexity Management Strategies

### Proactive Complexity Management
Prevent complexity from growing uncontrollably:

**Code Reviews**
- **Complexity checks**: Review code for complexity indicators
- **Design reviews**: Review design decisions for complexity impact
- **Refactoring suggestions**: Suggest refactoring opportunities
- **Knowledge sharing**: Share complexity management best practices

**Architecture Governance**
- **Design guidelines**: Establish guidelines for managing complexity
- **Architecture reviews**: Regular reviews of system architecture
- **Technology choices**: Evaluate technologies for complexity impact
- **Standards enforcement**: Enforce coding and design standards

**Training and Education**
- **Complexity awareness**: Train developers on complexity concepts
- **Refactoring skills**: Teach refactoring techniques
- **Design patterns**: Educate on design patterns for complexity management
- **Tool usage**: Train on complexity analysis tools

### Reactive Complexity Management
Address complexity when it becomes problematic:

**Technical Debt Management**
- **Debt identification**: Identify areas with high complexity
- **Debt prioritization**: Prioritize based on risk and impact
- **Debt repayment**: Schedule refactoring and improvement
- **Debt tracking**: Track technical debt over time

**Refactoring Sprints**
- **Dedicated time**: Allocate time specifically for refactoring
- **Targeted improvements**: Focus on high-impact areas
- **Team collaboration**: Collaborative refactoring efforts
- **Measurable results**: Track improvements with metrics

**Legacy System Modernization**
- **Incremental approach**: Gradually modernize legacy systems
- **Strangler fig pattern**: Replace parts of the system incrementally
- **Abstraction layers**: Create abstractions to hide complexity
- **Migration planning**: Plan and execute systematic migration

### Complexity Monitoring
Monitor complexity to prevent problems:

**Metric Tracking**
- **Trend analysis**: Track complexity metrics over time
- **Threshold monitoring**: Alert when metrics exceed thresholds
- **Correlation analysis**: Correlate complexity with other metrics
- **Predictive analysis**: Predict future complexity based on trends

**Quality Dashboards**
- **Visual metrics**: Display complexity metrics visually
- **Team metrics**: Show metrics at team and project level
- **Historical data**: Show historical trends and patterns
- **Actionable insights**: Provide insights for improvement

**Regular Assessments**
- **Code health checks**: Regular assessment of code quality
- **Architecture reviews**: Periodic review of system architecture
- **Team retrospectives**: Discuss complexity in retrospectives
- **Stakeholder reporting**: Report complexity to stakeholders

## Next

Continue to [Construction Best Practices](./software-construction-08-practices.md) to learn about practical guidelines and philosophies for effective software construction.