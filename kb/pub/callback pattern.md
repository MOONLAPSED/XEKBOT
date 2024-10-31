# Callback Pattern

## Core Concepts
- **Alternative Names**: Observer Pattern, One-to-Many Pattern, Publish-Subscribe Pattern
- **Category**: Behavioral Design Pattern
- **Mathematical Concept**: Monoid (composition of functions)
- **Key Property**: Decoupled Communication

## Foundational Definition
A pattern where an object (subject) maintains a collection of dependent functions or objects (observers) and invokes them when its state changes or events occur.

## Components
1. **Subject** (Publisher/Observable)
   - Maintains collection of callbacks
   - Provides registration interface
   - Notifies observers of state changes
   - Does not know observer implementation details

2. **Observer** (Subscriber/Callback)
   - Implements update mechanism
   - Registers with subject
   - Responds to notifications
   - Can observe multiple subjects

3. **Registration Mechanism**
   - Add/remove observers
   - Store observer references
   - Handle cleanup of dead references

4. **Notification Protocol**
   - Define how updates are sent
   - Specify data passed to observers
   - Determine notification timing

## Implementation Patterns

### Basic Implementation
```python
from typing import Protocol, List
from weakref import WeakSet

class Observer(Protocol):
    def update(self, message: str) -> None: ...

class Subject:
    def __init__(self) -> None:
        # WeakSet prevents memory leaks
        self._observers: WeakSet = WeakSet()
    
    def attach(self, observer: Observer) -> None:
        self._observers.add(observer)
    
    def detach(self, observer: Observer) -> None:
        self._observers.discard(observer)
    
    def notify(self, message: str) -> None:
        for observer in self._observers:
            observer.update(message)
```

### Function-Based Implementation
```python
from typing import Callable, Set
from dataclasses import dataclass, field

@dataclass
class EventEmitter:
    _callbacks: Set[Callable[[str], None]] = field(default_factory=set)
    
    def on(self, callback: Callable[[str], None]) -> None:
        self._callbacks.add(callback)
    
    def off(self, callback: Callable[[str], None]) -> None:
        self._callbacks.discard(callback)
    
    def emit(self, message: str) -> None:
        for callback in self._callbacks:
            callback(message)
```

## Common Use Cases
1. **Event Handling**
   - GUI applications
   - User input processing
   - System event monitoring

2. **State Change Propagation**
   - Model-View-Controller pattern
   - Data binding
   - Cache invalidation

3. **Asynchronous Operations**
   - Progress updates
   - Completion notifications
   - Error handling

## Behavioral Properties
1. **One-to-Many Relationship**
   - Single subject, multiple observers
   - Fan-out notification pattern
   - Dynamic observer registration

2. **Loose Coupling**
   - Subject doesn't know observer details
   - Observers can be added/removed at runtime
   - Independent evolution of components

3. **Composability**
   - Observers can be chained
   - Multiple subjects can share observers
   - Hierarchical observation patterns

## Implementation Considerations

### Memory Management
1. **Reference Handling**
   - Use weak references to prevent memory leaks
   - Implement cleanup mechanisms
   - Handle observer lifetime

2. **Performance**
   - Consider notification overhead
   - Batch updates when possible
   - Implement priority mechanisms

3. **Thread Safety**
   - Synchronize observer collection access
   - Handle concurrent notifications
   - Protect shared state

### Common Pitfalls
1. **Circular References**
   - Observer holding subject reference
   - Mutual observation cycles
   - Memory leak potential

2. **Update Loops**
   - Infinite notification chains
   - Recursive updates
   - State inconsistency

3. **Resource Management**
   - Unregistered observers
   - Dangling references
   - Resource cleanup

## Related Patterns
1. **Mediator Pattern**
   - Centralized communication
   - Many-to-many relationships
   - Event coordination

2. **Command Pattern**
   - Encapsulated operations
   - Queued execution
   - Undo/redo capability

3. **Chain of Responsibility**
   - Sequential processing
   - Request handling
   - Filter chains

## Mathematical Foundation
1. **Monoid Properties**
   - Associativity: (a • b) • c = a • (b • c)
   - Identity element: e • a = a • e = a
   - Closure: a • b is defined for all a, b

2. **Function Composition**
   ```python
   def compose(f, g):
       return lambda x: f(g(x))
   ```

## Testing Strategies
1. **Unit Tests**
   - Observer registration
   - Notification delivery
   - State consistency

2. **Integration Tests**
   - Multiple observer interaction
   - Event propagation
   - Error handling

3. **Performance Tests**
   - Notification scalability
   - Memory usage
   - Update latency

## Code Examples

### Type-Safe Implementation
```python
from typing import Generic, TypeVar, Protocol, Set
from dataclasses import dataclass, field

T = TypeVar('T')

class Observer(Protocol, Generic[T]):
    def on_update(self, data: T) -> None: ...

@dataclass
class TypedSubject(Generic[T]):
    observers: Set[Observer[T]] = field(default_factory=set)
    
    def attach(self, observer: Observer[T]) -> None:
        self.observers.add(observer)
    
    def notify(self, data: T) -> None:
        for observer in self.observers:
            observer.on_update(data)
```

## References
- [[design patterns]]
- [[event handling]]
- [[functional programming]]
- [[memory management]]
- [[concurrency patterns]]

## Meta Information
- **Pattern Category**: Behavioral
- **Difficulty Level**: Intermediate
- **Implementation Complexity**: Medium
- **Usage Frequency**: Very High
- **Language Agnostic**: Yes