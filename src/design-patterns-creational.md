# Creational Design Patterns

Creational design patterns abstract the instantiation process and help make a system independent of how its objects are created, composed, and represented. They provide flexibility in what gets created, who creates it, how it gets created, and when.

## Abstract Factory

**Intent**: Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

**Key Insight**: When you need to create objects that work together as a family (e.g., GUI components for different operating systems), the Abstract Factory pattern ensures compatibility by enforcing that all created objects belong to the same family. This is particularly useful when the system needs to be independent of how its products are created, composed, and represented.

**Consequences**:
- **Pros**: Promotes consistency among products, isolates concrete classes, makes exchanging product families easy, and supports new product families without changing client code.
- **Cons**: Adding new products requires extending the factory interface and all concrete implementations, which can be complex and may violate the open/closed principle.

## Builder

**Intent**: Separate the construction of a complex object from its representation, allowing the same construction process to create different representations.

**Key Insight**: When constructing complex objects with many optional parameters or configuration steps, the Builder pattern provides a clear, readable way to construct objects step by step. Instead of having a constructor with many parameters (telescoping constructor anti-pattern), you use a builder that accumulates configuration and then creates the final object.

**Consequences**:
- **Pros**: Provides fine-grained control over the construction process, supports different representations, encapsulates complex construction logic, and improves code readability.
- **Cons**: Increases the number of classes in the system, may introduce complexity for simple objects, and requires careful design to ensure thread safety.

## Factory Method

**Intent**: Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

**Key Insight**: When a class cannot anticipate the class of objects it must create, or when a class wants its subclasses to specify the objects it creates, the Factory Method pattern provides a way to delegate the instantiation logic to subclasses. This is useful when you want to localize the knowledge of which helper subclass is the appropriate one to create.

**Consequences**:
- **Pros**: Provides hooks for subclasses, eliminates the need to bind application-specific classes into your code, and supports parallel class hierarchies.
- **Cons**: Clients might have to subclass the Creator class just to create a particular Product object, which can be overkill if the Creator class is already heavily subclassed.

## Prototype

**Intent**: Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

**Key Insight**: When the cost of creating an object is more expensive than copying an existing one, or when you want to create objects without knowing their exact classes, the Prototype pattern allows you to create new objects by cloning existing ones. This is particularly useful for systems that allow users to create and configure complex objects dynamically.

**Consequences**:
- **Pros**: Allows adding and removing products at runtime, specifies new objects by varying values, reduces subclassing, and can dynamically configure an application with classes.
- **Cons**: Can be challenging to implement the clone operation correctly, especially for objects that contain circular references or don't support copying well.

## Singleton

**Intent**: Ensure a class has only one instance and provide a global point of access to it.

**Key Insight**: When exactly one object is needed to coordinate actions across the system (e.g., a database connection pool, thread pool, or configuration manager), the Singleton pattern ensures that only one instance exists and provides a well-known access point to it. This prevents multiple instances that could lead to inconsistent states or resource conflicts.

**Consequences**:
- **Pros**: Controlled access to the sole instance, reduced namespace pollution, permits refinement of operations and representation, and allows a variable number of instances.
- **Cons**: Can make unit testing difficult, can introduce global state, and can create bottlenecks in concurrent systems if not implemented carefully.