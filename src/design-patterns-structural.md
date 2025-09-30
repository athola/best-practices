# Structural Design Patterns

Structural design patterns describe how to assemble objects and classes into larger structures while keeping these structures flexible and efficient. They help ensure that if one part of a system changes, the entire structure doesn't need to change with it.

## Adapter

**Intent**: Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

**Key Insight**: When you need to integrate existing components with incompatible interfaces, the Adapter pattern acts as a bridge between them. This is particularly useful when working with legacy code, third-party libraries, or when you want to create a reusable class that cooperates with unrelated or unforeseen classes.

**Consequences**:
- **Pros**: Allows collaboration between otherwise incompatible classes, promotes reusability of existing functionality, and decouples the client from the adapted class.
- **Cons**: Can increase complexity, may introduce performance overhead, and can make the system harder to understand if overused.

## Bridge

**Intent**: Decouple an abstraction from its implementation so that the two can vary independently.

**Key Insight**: When you want to avoid a permanent binding between an abstraction and its implementation, the Bridge pattern separates them into separate class hierarchies. This is useful when you need to extend both the abstraction and implementation independently, or when you want to share an implementation among multiple abstractions.

**Consequences**:
- **Pros**: Decouples interface and implementation, improves extensibility, hides implementation details from clients, and supports dynamic implementation switching.
- **Cons**: Increases complexity due to additional indirection, can make debugging more difficult, and may be overkill for simple scenarios.

## Composite

**Intent**: Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.

**Key Insight**: When you need to work with tree structures of objects where both individual objects and compositions should be treated the same way, the Composite pattern provides a unified interface. This is particularly useful for graphics applications, file systems, or any hierarchical structure where you want to treat leaves and branches uniformly.

**Consequences**:
- **Pros**: Simplifies client code by treating individual and composite objects uniformly, makes it easy to add new kinds of components, and promotes flexibility in building complex structures.
- **Cons**: Can make the design overly general, may be difficult to restrict components, and can introduce safety issues if not implemented carefully.

## Decorator

**Intent**: Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

**Key Insight**: When you need to add responsibilities to individual objects dynamically and transparently, without affecting other objects, the Decorator pattern allows you to wrap objects with new functionality. This is particularly useful when you want to add features to objects without changing their core functionality or when subclassing would be impractical due to the large number of independent extensions.

**Consequences**:
- **Pros**: More flexible than static inheritance, avoids feature-laden classes high up in the hierarchy, and allows for dynamic addition/removal of responsibilities.
- **Cons**: Can result in many small objects that look alike to the programmer, can complicate the process of instantiating objects, and may make debugging more difficult.

## Facade

**Intent**: Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

**Key Insight**: When you need to simplify a complex subsystem or provide a simple entry point to a complex system, the Facade pattern provides a simplified interface. This is particularly useful for legacy systems, complex libraries, or when you want to reduce dependencies between subsystems and clients.

**Consequences**:
- **Pros**: Shields clients from subsystem components, promotes weak coupling between subsystems and clients, and doesn't prevent applications from using subsystem classes if needed.
- **Cons**: Can become a "god object" that accumulates too much responsibility, may hide useful functionality, and can create an additional layer of indirection.

## Flyweight

**Intent**: Use sharing to support large numbers of fine-grained objects efficiently.

**Key Insight**: When you need to support large numbers of objects that have similar state, the Flyweight pattern reduces memory usage by sharing common state among objects. This is particularly useful for text editors, graphics applications, or any system where you need to create many similar objects that would otherwise consume excessive memory.

**Consequences**:
- **Pros**: Reduces memory usage, improves performance for large numbers of objects, and centralizes state management.
- **Cons**: Can increase runtime costs associated with transferring, finding, or computing extrinsic state, may make the system more complex, and can introduce threading issues if not handled carefully.

## Proxy

**Intent**: Provide a surrogate or placeholder for another object to control access to it.

**Key Insight**: When you need to control access to an object, add additional functionality when accessing an object, or defer the creation of expensive objects, the Proxy pattern acts as an intermediary. This is particularly useful for remote objects, expensive-to-create objects, or when you need to add security, logging, or other cross-cutting concerns.

**Consequences**:
- **Pros**: Provides controlled access to the real subject, allows for lazy initialization, supports access control, and can add additional functionality transparently.
- **Cons**: Can introduce additional latency, may increase complexity, and can make the system harder to debug due to the indirection.