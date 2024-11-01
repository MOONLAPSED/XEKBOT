Here's a Python example that demonstrates the Method Resolution Order (MRO) and `super()` behavior in a multiple inheritance scenario. The code uses comments and docstrings to clarify the behavior of each part.

```python
class Base:
    """Base class with common methods shared across subclasses."""
    def __init__(self):
        print("Base __init__")

    def common_method(self):
        """Base method to be overridden in subclasses."""
        print("Base common_method")


class A(Base):
    """Class A inherits from Base and overrides common_method."""
    def __init__(self):
        super().__init__()
        print("A __init__")

    def common_method(self):
        """Overrides Base common_method in A."""
        super().common_method()  # Calls Base's common_method
        print("A common_method")


class B(Base):
    """Class B inherits from Base and overrides common_method."""
    def __init__(self):
        super().__init__()
        print("B __init__")

    def common_method(self):
        """Overrides Base common_method in B."""
        super().common_method()  # Calls Base's common_method
        print("B common_method")


class C(A, B):
    """Class C inherits from A and B, forming a diamond inheritance pattern.
    Uses super() to demonstrate MRO in a complex hierarchy."""
    def __init__(self):
        super().__init__()
        print("C __init__")

    def common_method(self):
        """Overrides common_method and demonstrates MRO using super()."""
        super().common_method()  # Calls next in MRO (A's common_method)
        print("C common_method")


# Instantiate C and call the methods to see the MRO in action
c_instance = C()
print("\nCalling common_method:\n")
c_instance.common_method()

# Check the Method Resolution Order for class C
print("\nMethod Resolution Order (MRO) for C:")
print(C.__mro__)
```

### Explanation:

1. **Class Structure**: 
    - `Base` is the superclass that defines a common `common_method`.
    - `A` and `B` both inherit from `Base` and override `common_method`, but each calls `super().common_method()` to demonstrate cooperative inheritance.
    - `C` inherits from both `A` and `B`, forming a diamond inheritance pattern (Base is inherited twice).

2. **`__init__` Method**: 
    - Each `__init__` method uses `super()` to ensure initialization of all parent classes in the MRO.

3. **`common_method` Demonstration**:
    - The `common_method` in `C` calls `super().common_method()`, triggering a chain of calls down the MRO.

4. **MRO Output**:
    - The `C.__mro__` output will show the MRO order as:
      ```python
      (<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>, <class '__main__.Base'>, <class 'object'>)
      ```
    - This MRO order is followed by `super()` to decide which `common_method` to call next.

### Expected Output:

```plaintext
Base __init__
B __init__
A __init__
C __init__

Calling common_method:

Base common_method
B common_method
A common_method
C common_method

Method Resolution Order (MRO) for C:
(<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>, <class '__main__.Base'>, <class 'object'>)
```

This example demonstrates how `super()` respects the MRO, which is essential for cooperative inheritance and avoiding repetitive calls in a diamond inheritance pattern.