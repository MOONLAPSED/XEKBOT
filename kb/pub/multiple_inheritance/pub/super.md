#Python #Class [[super|super()]]
### Python’s Method Resolution Order (MRO) as a Partially Ordered Set (Poset)

1. **Linearization**:
    
    - In Python, MRO is determined by **C3 linearization** (also called C3 superclass linearization), which creates a linear order of classes in a multiple inheritance tree while preserving the hierarchy. This is much like a **partially ordered set**, where Python resolves methods by following a predictable, deterministic path through the hierarchy.
    - The MRO in Python respects class precedence, preserving the relationship between the superclass and subclass methods.
2. **Poset Structure**:
    
    - If you think of each class as a node, inheritance as directed edges, and the MRO as a linear path through the directed acyclic graph (DAG) of classes, Python’s MRO creates a directed path. This ensures that each class’s methods are called only once and in the correct sequence, even in complex hierarchies.

### `super()` as a Traversal Mechanism in the MRO Graph

When you use `super()`, you’re not just calling the “parent” class; instead, you're invoking the **next class in the MRO**, following the hierarchy Python calculated. This makes `super()` incredibly flexible and avoids hardcoding which superclass to call. It operates in a **context-aware way**, adapting based on the MRO, which is why it’s sometimes described as a “contextual `super()`.”