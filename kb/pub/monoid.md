# Monoid

## Core Concepts
- **Category**: Abstract Algebra
- **Related Structures**: Semigroups, Groups
- **Key Property**: Composability with Identity

## Foundational Definition
A monoid is a set equipped with:
1. An associative binary operation (•)
2. An identity element (e)

Formally, for all a, b, c in the set:
- (a • b) • c = a • (b • c) [associativity]
- e • a = a • e = a [identity]

## Essential Properties

### 1. Associativity
```python
# For any a, b, c in our set:
(a • b) • c == a • (b • c)

# Example with list concatenation
([1] + [2]) + [3] == [1] + ([2] + [3])  # Both yield [1, 2, 3]
```

### 2. Identity Element
```python
# For any a in our set:
e • a == a • e == a

# Example with string concatenation
"" + "hello" == "hello" + "" == "hello"
```

## Common Examples

### 1. Number Systems
```python
# Addition monoid over integers
identity = 0
operation = lambda a, b: a + b

# Multiplication monoid over integers
identity = 1
operation = lambda a, b: a * b
```

### 2. String Operations
```python
# String concatenation monoid
identity = ""
operation = lambda a, b: a + b

assert operation("hello", identity) == "hello"  # Identity
assert operation("hello", operation("world", "!")) == \
       operation(operation("hello", "world"), "!")  # Associativity
```

### 3. List Operations
```python
from typing import TypeVar, List

T = TypeVar('T')

class ListMonoid(Generic[T]):
    @staticmethod
    def empty() -> List[T]:
        return []  # Identity
    
    @staticmethod
    def combine(a: List[T], b: List[T]) -> List[T]:
        return a + b  # Operation
```

## Application to Callbacks

### Function Composition as a Monoid
```python
from typing import Callable, TypeVar

A = TypeVar('A')
B = TypeVar('B')
C = TypeVar('C')

class FunctionMonoid:
    @staticmethod
    def identity(x: A) -> A:
        return x
    
    @staticmethod
    def compose(f: Callable[[B], C], 
                g: Callable[[A], B]) -> Callable[[A], C]:
        return lambda x: f(g(x))

# Example usage showing associativity
f = lambda x: x + 1
g = lambda x: x * 2
h = lambda x: x ** 2

# These are equivalent due to associativity:
FunctionMonoid.compose(FunctionMonoid.compose(f, g), h)(2) == \
FunctionMonoid.compose(f, FunctionMonoid.compose(g, h))(2)
```

### Connection to Callback Pattern
The callback pattern forms a monoid when:
1. The binary operation is callback chaining
2. The identity is a no-op callback

```python
from typing import Callable, Optional
from dataclasses import dataclass

@dataclass
class CallbackMonoid:
    callback: Callable[[str], Optional[str]]
    
    @staticmethod
    def identity() -> 'CallbackMonoid':
        return CallbackMonoid(lambda x: x)
    
    def combine(self, other: 'CallbackMonoid') -> 'CallbackMonoid':
        def combined(x: str) -> Optional[str]:
            result = self.callback(x)
            return other.callback(result) if result is not None else None
        return CallbackMonoid(combined)

# Example usage
logger = CallbackMonoid(lambda x: f"[LOG] {x}")
validator = CallbackMonoid(lambda x: x if len(x) < 10 else None)
processor = logger.combine(validator)

assert processor.callback("short") == "[LOG] short"
assert processor.callback("very long message") is None
```

## Properties in Programming

### 1. Parallelization
Monoids allow for parallel processing because:
```python
combine(a, combine(b, c)) == combine(combine(a, b), c)
```

### 2. Accumulation
```python
from functools import reduce
from typing import TypeVar, List, Callable

T = TypeVar('T')

def fold_monoid(items: List[T], 
                identity: T,
                combine: Callable[[T, T], T]) -> T:
    return reduce(combine, items, identity)
```

### 3. Error Handling
```python
from typing import Optional

class OptionalMonoid:
    @staticmethod
    def identity() -> Optional[str]:
        return ""
    
    @staticmethod
    def combine(a: Optional[str], b: Optional[str]) -> Optional[str]:
        if a is None or b is None:
            return None
        return a + b
```

## Relationship to Other Patterns
1. **Callback Pattern** [[callback pattern]]
   - Callbacks can be composed like monoid operations
   - The identity is a pass-through callback
   - Composition is associative

2. **Builder Pattern**
   - Method chaining forms a monoid
   - Empty builder is the identity
   - Building operations are associative

3. **Reducer Pattern**
   - Reduction operations form monoids
   - Initial state is the identity
   - Reductions can be parallelized due to associativity

## Common Use Cases
1. **Data Processing**
   - Parallel computations
   - Stream processing
   - Map-reduce operations

2. **State Management**
   - Event sourcing
   - Immutable state updates
   - Transaction composition

3. **Error Handling**
   - Optional value chaining
   - Error propagation
   - Validation composition

## References
- [[category theory]]
- [[abstract algebra]]
- [[functional programming]]
- [[design patterns]]

## Meta Information
- **Concept Type**: Mathematical Structure
- **Programming Paradigm**: Functional
- **Abstraction Level**: High
- **Implementation Complexity**: Medium
- **Prerequisites**: Basic Algebra, Functions