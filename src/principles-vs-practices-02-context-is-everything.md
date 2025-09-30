# Context is Everything

The practices in this guide are presented as patterns that have worked in specific contexts, not as universal rules. Your team's context includes many factors that determine which approaches will work best for your situation.

## Key Context Factors

### Project Size and Complexity
The scale and intricacy of your software system significantly impacts which practices are appropriate:

- **Small projects** (1-5 developers, simple requirements) can benefit from lightweight processes and minimal ceremony
- **Medium projects** (5-20 developers, moderate complexity) need more structure but should avoid over-engineering
- **Large projects** (20+ developers, high complexity) require robust processes, clear architecture, and comprehensive testing

### Team Experience and Expertise
The skill level and background of your development team:

- **Junior teams** need more guidance, code reviews, and structured processes
- **Mixed teams** benefit from mentorship, clear standards, and gradual responsibility
- **Senior teams** can operate with more autonomy and lighter processes

### Business Domain and Requirements
The specific industry and functional needs of your application:

- **Internal tools** can prioritize speed and flexibility over polish and documentation
- **Customer-facing products** need higher quality, better UX, and more thorough testing
- **Regulated industries** (finance, healthcare) require additional compliance, security, and auditability

### Performance and Reliability Needs
The operational requirements and service level expectations:

- **Non-critical systems** can tolerate occasional downtime and slower performance
- **Business-critical systems** need high availability, monitoring, and performance optimization
- **Life-critical systems** require formal methods, extensive testing, and rigorous processes

### Time and Resource Constraints
The budget, timeline, and available personnel:

- **Startup environments** prioritize speed to market and rapid iteration
- **Enterprise environments** balance speed with stability and compliance
- **Resource-constrained projects** must make careful trade-offs between scope and quality

## Context Examples

### Startup vs. Enterprise

**Startup Context:**
- Small team, limited resources
- Rapid iteration and pivoting
- Uncertain requirements
- Focus on speed and learning

**Enterprise Context:**
- Large team, established processes
- Stable requirements and long-term planning
- Regulatory compliance needs
- Focus on stability and scalability

What works for a startup building an MVP won't work for a financial institution handling transactions.

### Solo Developer vs. Distributed Team

**Solo Developer Context:**
- Complete autonomy and control
- Direct communication with stakeholders
- Simple coordination needs
- Flexible decision-making

**Distributed Team Context:**
- Multiple stakeholders and decision-makers
- Complex communication requirements
- Need for clear processes and documentation
- Coordination overhead and potential conflicts

What works for a solo developer won't work for a distributed team of 50 engineers.

### Different Project Types

**Research Project:**
- Exploratory nature
- Changing requirements
- Focus on discovery and learning
- Tolerance for experimentation and failure

**Production System:**
- Well-defined requirements
- Focus on reliability and performance
- Need for maintainability and scalability
- Low tolerance for failures

**Internal Tool:**
- Limited user base
- Direct feedback loop
- Flexible requirements
- Focus on functionality over polish

**Customer Product:**
- Large user base
- Indirect feedback mechanisms
- Stable requirements
- Focus on UX, performance, and reliability

## The Context-Practice Matrix

Different contexts call for different practices. Here's how context affects practice selection:

| Context Factor | Lightweight Practice | Heavyweight Practice |
|----------------|---------------------|----------------------|
| **Team Size** | Small teams (1-5) | Large teams (20+) |
| **Project Criticality** | Internal tools | Customer-facing systems |
| **Requirements Stability** | Research/exploratory | Well-established domains |
| **Time Pressure** | Startup/rapid iteration | Enterprise/long-term |
| **Team Experience** | Senior engineers | Junior/mixed teams |

## Assessing Your Context

To determine which practices are right for your team, ask these questions:

### Project Assessment
- How many developers are working on this project?
- How complex are the requirements?
- How stable are the requirements likely to be?
- What's the expected lifespan of this project?

### Team Assessment
- What's the experience level of your team members?
- How well do team members know each other?
- What's the team's track record with similar projects?
- How distributed is your team (geographically and temporally)?

### Business Assessment
- How critical is this system to the business?
- Who are the users of this system?
- What are the consequences of failures or bugs?
- What are the compliance and regulatory requirements?

### Operational Assessment
- What are the performance requirements?
- What are the availability requirements?
- What are the security requirements?
- What are the maintenance and support expectations?

## Evolving Context

Context is not static—it evolves over time. As your project grows and changes, your practices should evolve too:

### Growth Triggers
- **Team growth**: Adding more developers often requires more structure
- **User growth**: More users often require better performance and reliability
- **Feature growth**: More features often require better architecture and organization
- **Business growth**: Business success often brings more scrutiny and compliance needs

### Evolution Strategies
- **Gradual evolution**: Add practices incrementally as needed
- **Periodic review**: Regularly assess if your current practices are still appropriate
- **Context-aware adaptation**: Be willing to change practices when context changes
- **Learning from experience**: Use successes and failures to refine your approach

## Next

Continue to [The Engineer's Dilemma: Balancing Contradictory Principles](./principles-vs-practices-03-balancing-principles.md) to learn how to navigate the contradictory principles that exist in software engineering.