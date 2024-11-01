#Python #Class [[Inheritance]] [[super|super()]]  [[c3 Inheritance]]
# Understanding Python Inheritance and `super()`

### Overview
Inheritance is a core concept in object-oriented programming (OOP) that allows a class (subclass) to inherit attributes and methods from another class (superclass). This enables reusability and the ability to extend or modify behavior in subclasses. 

Python’s `super()` function is especially useful for managing inheritance because it provides a way to refer to the parent class without explicitly naming it. This helps keep your code maintainable and supports multiple inheritance, where a class inherits from more than one superclass.

In Python 3.12 and newer, `super()` has enhanced usage scenarios, especially when dealing with complex inheritance structures and cooperative inheritance.

### Basics of `super()` [[super]]

The `super()` function is generally used to:
- Call a method from a superclass.
- Avoid hardcoding the parent class’s name, which improves code flexibility.
- Allow for cooperative multiple inheritance by calling the next class in the method resolution order (MRO).

Here’s a simple example:

```python
class Animal:
    def speak(self):
        return "Animal sound"

class Dog(Animal):
    def speak(self):
        return super().speak() + " - Woof!"

d = Dog()
print(d.speak())  # Output: Animal sound - Woof!
```

### Advanced Usage and the `super()` Chain [[super]]

In complex inheritance scenarios, especially with multiple inheritance, `super()` ensures that methods from all parent classes are called in a consistent and predictable order according to the MRO.

#### Example with Multiple Inheritance

```python
class A:
    def a(self):
        print("a")

    def b(self):
        print("A's b method")
        self.a()
        super().b()
        print("A's super:", super())
        
    def d(self):
        print("A's d method")

class C:
    def __init__(self):
        print("C's __init__")

    def b(self):
        print("C's b method")
        super().b()
        print("C's super:", super())

class B(A, C):
    def __init__(self):
        print("B's __init__")
        super().__init__()

    def b(self):
        print("B's b method")
        super().b()
        self.c()
        super(A, self).d()  # Specific call to A's d() method
        super(C, self).d()  # Specific call to C's d() method

    def a(self):
        print("B overrides A's a method")

# Instantiate and call method
b_instance = B()
b_instance.b()
```

#### Explanation of Key Points in the Example

1. **Method Overriding**: Class `B` overrides method `a()` from class `A`. When `b_instance.b()` is called, `B`’s version of `a()` will be executed instead of `A`’s.
  
2. **Method Resolution Order (MRO)**: Python follows a specific order in looking up methods for multiple inheritance, which can be seen with the `__mro__` attribute.

   ```python
   print(B.__mro__)
   ```
   
   Output:
   ```
   (<class '__main__.B'>, <class '__main__.A'>, <class '__main__.C'>, <class 'object'>)
   ```

3. **Using `super()` with Arguments**: The call `super(A, self).d()` in `B.b()` is directed specifically to call `d()` from `A`, bypassing `C` in the MRO.

4. **Breakpoints and Inspection**: Using `print()` statements or debugging tools (`breakpoint()`) can reveal the behavior of `super()` and MRO interactions.

### Example with Cooperative [[Multiple Inheritance]] [[super]]

A classic use case in multiple inheritance is when each class has a common method, and `super()` is used to call each class’s implementation in a chain.

```python
class Parent1:
    def method(self):
        print("Parent1 method")

class Parent2:
    def method(self):
        print("Parent2 method")

class Child(Parent1, Parent2):
    def method(self):
        super(Parent1, self).method()  # Calls Parent1's method
        super(Parent2, self).method()  # Calls Parent2's method

child = Child()
child.method()
```

#### Key Takeaways
- **Ordering in MRO**: `super()` will follow the MRO order defined by the order of inheritance in `Child`.
- **Flexibility**: Using `super()` without hardcoding parent class names allows for easy adjustments in the inheritance hierarchy.

### Understanding Method Wrappers and Types

In the example code, `MethodType` and `MethodWrapperType` are used to inspect methods and wrappers. Here’s a simplified exploration:

```python
from types import MethodType, MethodWrapperType

class Sample:
    def __init__(self):
        print("Sample init")

# Check types
sample_instance = Sample()
print(isinstance(sample_instance.__init__, MethodType))  # False for __init__ (it's not a method type)
print(isinstance(sample_instance.__init__, MethodWrapperType))  # True if it's a built-in wrapper
```

### Summary

Using `super()` and understanding inheritance patterns can greatly enhance code modularity and flexibility in Python, especially with multiple inheritance. In complex designs, understanding MRO, cooperative inheritance, and Python’s `super()` function with context-specific calls (like `super(Class, self)`) is essential for maintaining clear and effective code.
