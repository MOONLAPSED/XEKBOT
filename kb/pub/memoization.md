# Memoization

## Core Concepts
- **Category**: Optimization Pattern
- **Related Patterns**: [[callback pattern]], [[monoid]]
- **Key Property**: Idempotent Computation Caching

## Foundational Definition
A technique that stores computed results of pure functions, creating a mapping between inputs and outputs to avoid redundant calculations.

## Mathematical Properties

### 1. Function Properties Required
```python
# Pure function (required for memoization)
def pure_add(a: int, b: int) -> int:
    return a + b  # Same input always yields same output

# Impure function (cannot be safely memoized)
def impure_add(a: int, b: int) -> int:
    return a + b + random.randint(0, 1)  # Output varies for same input
```

### 2. Monoid Formation
Memoization caches form a monoid under composition:
```python
from typing import Dict, TypeVar, Callable, Any
from dataclasses import dataclass

K = TypeVar('K')
V = TypeVar('V')

@dataclass
class MemoizationCache:
    cache: Dict[K, V]
    
    def combine(self, other: 'MemoizationCache') -> 'MemoizationCache':
        return MemoizationCache({**self.cache, **other.cache})
    
    @staticmethod
    def empty() -> 'MemoizationCache':
        return MemoizationCache({})  # Identity element

# Associativity: (a.combine(b)).combine(c) == a.combine(b.combine(c))
```

## Implementation Patterns

### 1. Basic Decorator Pattern
```python
from functools import wraps
from typing import TypeVar, Callable, Dict, Any

T = TypeVar('T')

def memoize(func: Callable[..., T]) -> Callable[..., T]:
    cache: Dict[tuple, T] = {}
    
    @wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> T:
        key = (*args, *sorted(kwargs.items()))
        if key not in cache:
            cache[key] = func(*args, **kwargs)
        return cache[key]
    
    wrapper.cache = cache  # Expose cache for inspection/clearing
    return wrapper

@memoize
def expensive_computation(x: int) -> int:
    return x ** 2
```

### 2. Class-Based Pattern with Lifecycle Management
```python
from typing import Generic, TypeVar, Dict, Optional
from datetime import datetime, timedelta

K = TypeVar('K')
V = TypeVar('V')

class MemoizedValue(Generic[V]):
    def __init__(self, value: V, ttl: Optional[timedelta] = None):
        self.value = value
        self.timestamp = datetime.now()
        self.ttl = ttl

    def is_valid(self) -> bool:
        if self.ttl is None:
            return True
        return datetime.now() - self.timestamp < self.ttl

class MemoizationManager(Generic[K, V]):
    def __init__(self):
        self._cache: Dict[K, MemoizedValue[V]] = {}
    
    def get(self, key: K) -> Optional[V]:
        if key in self._cache and self._cache[key].is_valid():
            return self._cache[key].value
        return None
    
    def set(self, key: K, value: V, ttl: Optional[timedelta] = None) -> None:
        self._cache[key] = MemoizedValue(value, ttl)
    
    def clear_expired(self) -> None:
        self._cache = {
            k: v for k, v in self._cache.items() if v.is_valid()
        }
```

### 3. Callback Integration Pattern
```python
from typing import Callable, TypeVar, Dict, Optional
from dataclasses import dataclass

T = TypeVar('T')
R = TypeVar('R')

@dataclass
class MemoizedCallback(Generic[T, R]):
    func: Callable[[T], R]
    cache: Dict[T, R] = field(default_factory=dict)
    
    def __call__(self, arg: T) -> R:
        if arg not in self.cache:
            self.cache[arg] = self.func(arg)
        return self.cache[arg]
    
    def compose(self, other: 'MemoizedCallback[R, Any]') -> 'MemoizedCallback[T, Any]':
        def composed(x: T) -> Any:
            return other(self(x))
        return MemoizedCallback(composed)

# Example usage with callback chain
transform = MemoizedCallback(lambda x: x * 2)
validate = MemoizedCallback(lambda x: x if x < 100 else None)
process = transform.compose(validate)
```

## Advanced Patterns

### 1. Concurrent Memoization
```python
from threading import Lock
from typing import Dict, TypeVar, Callable
from functools import wraps

T = TypeVar('T')

class ThreadSafeMemoize:
    def __init__(self, func: Callable[..., T]):
        self.func = func
        self.cache: Dict[tuple, T] = {}
        self.lock = Lock()
    
    def __call__(self, *args: Any, **kwargs: Any) -> T:
        key = (*args, *sorted(kwargs.items()))
        
        with self.lock:
            if key not in self.cache:
                self.cache[key] = self.func(*args, **kwargs)
            return self.cache[key]

@ThreadSafeMemoize
def thread_safe_computation(x: int) -> int:
    return x ** 2
```

### 2. Hierarchical Memoization
```python
from typing import Dict, Any, Optional

class HierarchicalCache:
    def __init__(self, parent: Optional['HierarchicalCache'] = None):
        self.cache: Dict[Any, Any] = {}
        self.parent = parent
    
    def get(self, key: Any) -> Optional[Any]:
        if key in self.cache:
            return self.cache[key]
        if self.parent:
            return self.parent.get(key)
        return None
    
    def set(self, key: Any, value: Any) -> None:
        self.cache[key] = value

# Example usage
global_cache = HierarchicalCache()
local_cache = HierarchicalCache(parent=global_cache)
```

## Common Use Cases

### 1. Dynamic Programming
- Fibonacci sequence
- Path finding algorithms
- Optimization problems

### 2. API Response Caching
```python
@dataclass
class APICache:
    ttl: timedelta
    cache_manager: MemoizationManager[str, dict] = field(
        default_factory=MemoizationManager
    )
    
    def get_data(self, endpoint: str) -> dict:
        if cached := self.cache_manager.get(endpoint):
            return cached
        result = self.fetch_data(endpoint)  # External API call
        self.cache_manager.set(endpoint, result, self.ttl)
        return result
```

### 3. Computation Graph Optimization
```python
from typing import Dict, Set, Callable

class ComputationNode:
    def __init__(self, compute: Callable[..., Any]):
        self.compute = compute
        self.cache: Dict[tuple, Any] = {}
        self.dependencies: Set['ComputationNode'] = set()
    
    def add_dependency(self, node: 'ComputationNode') -> None:
        self.dependencies.add(node)
    
    def invalidate(self) -> None:
        self.cache.clear()
        for dep in self.dependencies:
            dep.invalidate()
```

## Common Pitfalls

1. **Memory Leaks**
   - Unbounded cache growth
   - Circular references
   - Resource cleanup

2. **Cache Invalidation**
   - Stale data
   - Inconsistent states
   - Race conditions

3. **Performance Overhead**
   - Cache lookup cost
   - Memory pressure
   - Thread contention

## References
- [[dynamic programming]]
- [[cache patterns]]
- [[functional programming]]
- [[callback pattern]]
- [[monoid]]

## Meta Information
- **Pattern Category**: Optimization
- **Memory Usage**: Variable
- **Thread Safety**: Implementation Dependent
- **Complexity**: Medium
- **Prerequisites**: Pure Functions, Cache Management