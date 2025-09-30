# Context-Dependent Organization

Project structure isn't about following rigid rules—it's about creating an organization that makes sense for your specific context. The "right" structure depends on your team size, project complexity, and development workflow.

## Context-Driven Design Approach

"The best project structures emerge from understanding your specific context rather than following universal rules. Context includes team size, project complexity, domain characteristics, development methodology, and business requirements."

### Context Factors

- **Team Context**: Size, experience, distribution, collaboration patterns
- **Project Context**: Complexity, lifespan, criticality, evolution rate
- **Domain Context**: Business domain, regulatory requirements, user base
- **Technical Context**: Technology stack, integration requirements, performance needs
- **Business Context**: Time to market, budget constraints, strategic goals

"There is no 'best' project structure—only structures that are more or less appropriate for your specific context. The key is to understand the trade-offs and choose accordingly."

## When to Separate Concerns

Organizing by concern (what code does) works well when:

### Context: Medium to large projects with multiple developers

```bash
project/
├── src/                    # Source code
├── tests/                  # Test files
├── docs/                   # Documentation
├── config/                 # Configuration files
├── scripts/                # Build and utility scripts
└── tools/                  # Development tools
```

**Why this works:** Clear separation helps teams navigate complex codebases and reduces merge conflicts.

**Analysis**: This structure follows the principle of separation of concerns, which is identified as fundamental to managing complexity. Each directory has a clear, single responsibility, making the system easier to understand and maintain.

**When this might not work:** Small projects or solo developers might find this structure creates unnecessary overhead. A simple `src/` and `tests/` might be sufficient.

**Guidance**: "Start with the simplest structure that works, and add complexity only when the pain of not having it becomes greater than the pain of having it."

## When to Organize by Feature

Organizing by feature works better in different contexts:

### Context: Projects with clear domain boundaries or microservices

```bash
project/
├── user-service/           # User management feature
├── payment-service/        # Payment processing feature
├── notification-service/   # Notification feature
└── shared/                 # Common utilities
```

**Why this works:** Teams can own features independently, and deployment boundaries are clear.

**Analysis**: This structure aligns with principle of high cohesion and loose coupling. Each service is highly cohesive (contains all functionality related to its feature) and loosely coupled (minimizes dependencies on other services).

**When this might not work:** Projects with heavy shared logic might lead to code duplication or complex dependency management.

**Warning**: "Be careful of the 'shared utilities' trap—shared code often becomes a bottleneck for change and a source of coupling. Consider whether shared functionality is truly shared or just commonly used."

## Choosing the Right Approach

### Decision Framework

**When to Choose Separation of Concerns:**
- **Team size**: Medium to large teams (4+ developers)
- **Project complexity**: Medium to high complexity with multiple concerns
- **Lifespan**: Long-term projects that will evolve over time
- **Collaboration**: Multiple developers working on different aspects simultaneously

**When to Choose Feature-Based Organization:**
- **Domain clarity**: Clear business domain boundaries
- **Team autonomy**: Teams that own specific features end-to-end
- **Deployment needs**: Independent deployment of features
- **Scalability**: Need to scale different features independently

**When to Choose a Hybrid Approach:**
- **Mixed requirements**: Some parts need separation, others need feature organization
- **Evolutionary growth**: Project is evolving from one approach to another
- **Multiple stakeholders**: Different teams have different organizational needs

### Context Assessment Questions

Ask these questions to determine the right approach:

1. **How many developers are working on the project?**
   - 1-3 developers: Simple structure may be sufficient
   - 4-10 developers: Need more structure and clear boundaries
   - 10+ developers: Need sophisticated structure and formal processes

2. **How complex is the problem domain?**
   - Simple domain: Feature-based organization may work well
   - Complex domain: May need separation of concerns and clear layers
   - Multiple domains: May need multi-project structure

3. **How long will the project live?**
   - Short-term: Simple structure, focus on speed
   - Medium-term: Balanced structure, some organization
   - Long-term: Sophisticated structure, investment in organization

4. **How frequently will requirements change?**
   - Stable requirements: Can invest in more structure
   - Evolving requirements: Need flexible structure
   - Rapidly changing: Need very flexible, adaptable structure

## Key Takeaways

- **Context determines structure**: There's no universal "best" way to organize projects
- **Start simple**: Begin with the simplest structure that works for your current context
- **Evolve as needed**: Add structure only when the pain of not having it exceeds the pain of having it
- **Assess your context**: Consider team size, project complexity, domain, and business needs
- **Be prepared to change**: Your organizational needs may change as the project evolves

## Next

Continue to [Structural Design Patterns](./project-structure-03-structural-patterns.md) to explore specific architectural patterns and design trade-offs.