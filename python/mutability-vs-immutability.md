# Mutability Vs Immutability

## Variables and Objects
In Python, variables don’t have an associated type or size, as they’re labels attached to objects in memory. They point to the memory position where concrete objects live. In other words, a Python variable is a name that refers to or holds a reference to a concrete object. In contrast, Python objects are concrete pieces of information that live in specific memory positions on your computer.
Takeaway:
- Variables hold references to objects.
- Objects live in concrete memory positions.

Every Python object has three core characteristics:
1. Value
2. Identity
3. Type

```
x = 42  # value
x -> Variable

>>> id(x)   # identifier
4343440904

>>> type(x)     # type
int
```

## Mutable and Immutable Objects in Python
An object that allows you to change its values without changing its identity is a <b>mutable object</b>. 
The changes that you can perform on a mutable object’s value are known as mutations.
An object that doesn’t allow changes in its value is an <b>immutable object</b>. 
You have no way to perform mutations on this kind of object. 
You just have the option of creating a new object of the same type but with a different value, which you can do through a new assignment.

Single-item data types, such as integers, floats, complex numbers, and Booleans, are always immutable. 
So, you have no way to change the value of these types. You just have the option of creating a new object with a new value and throwing away the old one.

When it comes to collection or container types, such as strings, lists, tuples, sets, and dictionaries, things are a bit different. 
Like all other objects, a collection has a unique identity. However, collections internally keep references to all their individual values. 
This opens the opportunity for mutability. Strings and tuples are immutable, while lists, dictionaries, and sets are mutable.
In a mutable collection, you can change the value of individual items and, therefore, the identity of those items, however, the
 identity of the collection remains unchanged.


|  Data Type   |   Built-in Class    | Mutable |
|:------------:|:-------------------:|:-------:|
|   Numbers    | int, float, complex |    ❌    |
|   Strings    |         str         |    ❌    |
|    Tuples    |        tuple        |    ❌    |
|   Strings    |        bytes        |    ❌    |
|    Bytes     |        bool         |    ❌    |
| Frozen sets  |      frozenset      |    ❌    |
|    Lists     |        list         |    ✅    |
| Dictionaries |        dict         |    ✅    |
|     Sets     |         set         |    ✅    |
| Byte arrays  |      bytearray      |    ✅    |

### Good to know
1. Immutable objects may need more memory as they need to copy data to make any changes.
2. Mutable objects are not thread-safe.
3. Lists mutate in place and return None unlike strings which returns a new string
4. The keys in a dictionary must be unique as they are hashed. 
   - For an object to be hashable, you must be able to pass it to a hash function and get a unique hash code. To achieve this, your object must be unchangeable. Immutable types, such as numbers, Booleans, and strings, are hashable.
   - Important exception - tuples are only hashable when all their items are also hashable.
5. Sets are an unordered container of hashable objects. 

### Common Gotchas
1. Mutatating Arguments in Functions
```
>>> def squares_of(numbers):
...     for i, number in enumerate(numbers):
...         numbers[i] = number ** 2
...     return numbers
...

>>> sample = [2, 3, 4]
>>> squares_of(sample)
[4, 9, 16]
>>> sample
[4, 9, 16]
```

2. Avoid mutable objects as default argument values as it is dangerous
```
>>> def append_to(item, target=[]):
...     target.append(item)
...     return target

>>> # What you might expect to happen:
>>> append_to(1)
[1]
>>> append_to(2)
[2]
>>> append_to(3)
[3]

>>> # What actually happens:
>>> append_to(1)
[1]
>>> append_to(2)
[1, 2]
>>> append_to(3)
[1, 2, 3]

solution - use None
>>> def append_to(item, target=None):
...     if target is None:
...         target = []
...     target.append(item)
...     return target
```

3. Shallow vs deep copy
- Shallow copy - you create a new list of objects with a different identity. However, the internal components or data items in the new list are just aliases of those in the original list.
An important detail to note is that mutations on a shallow copy don’t affect the original list but mutations on original list will affect shallow copy.
- Deep copy - you create a completely new copy of the original list. <br>

4. The methods that change a mutable object in place return None.
5. In general, putting mutable objects in tuples is a bad idea. It kind of breaks the tuple’s immutability.
It also makes your tuples unhashable, which prevents you from using them as dictionary keys.
6. Concatenating many strings using the plus operator—or its augmented variation (+=)—would imply creating several temporary string objects 
because you can’t perform the mutation in place. All these intermediate strings will be discarded when added to the next string. (solution: use join).

### Mutability in Custom Classes
Custom classes and their instances are mutable. 
Techniques to Control Mutability in Custom Classes: <br>
    1. Defining a `.__slots__` Class Attribute
   - 
        ```
        >>> class Book:
        ...     __slots__ = ("title",)
        ...     def __init__(self, title):
        ...         self.title = title
        ...
        
        >>> harry_potter = Book("Harry Potter")
        >>> harry_potter.title
        'Harry Potter'
        
        >>> harry_potter.author = "J. K. Rowling"
        Traceback (most recent call last):
            ...
        AttributeError: 'Book' object has no attribute 'author' ```
     

Even though Book has a `.__slots__` attribute holding a tuple of allowed instance attributes, 
you can still add/delete new class attributes and methods dynamically to your Book class.

2. Providing Custom `.__setattr__()` and `.__delattr__()` Methods
    - ```
       class Immutable:
        def __init__(self, value):
            super().__setattr__("value", value)
    
        def __setattr__(self, name, attr_value):
            raise AttributeError(f"can't set attribute '{name}'")
    
        def __delattr__(self, name):
            raise AttributeError(f"can't delete attribute '{name}'")
        ```
If you use mutable objects as the value of an immutable class, then you won’t be able to prevent mutations on that value.

3. Using Read-Only Properties and Descriptors

   - ```
        class Point:
            def __init__(self, x, y):
                self._x = x
                self._y = y
        
            @property
            def x(self):
                return self._x
        
            @property
            def y(self):
                return self._y
        
            def __repr__(self):
                return f"{type(self).__name__}(x={self.x}, y={self.y})"
        ```
     you can still access and change the underlying non-public attributes, ._x and ._y:
        ```
        >>> point._x = 555
        >>> point
        Point(x=555, y=42)
        ```
     solution:
        ```
        class Coordinate:
            def __set_name__(self, owner, name):
                self._name = name
        
            def __get__(self, instance, owner):
                return instance.__dict__[f"_{self._name}"]
        
            def __set__(self, instance, value):
                raise AttributeError(f"can't set attribute '{self._name}'")
        
        class Point:
            x = Coordinate()
            y = Coordinate()
        
            def __init__(self, x, y):
                self._x = x
                self._y = y
        
            def __repr__(self):
                return f"{type(self).__name__}(x={self.x}, y={self.y})"
        ```
4. Building Immutable Tuple-Like Objects
    
- ```
            >>> from collections import namedtuple

            >>> Point = namedtuple("Point", "x y")
            >>> point = Point(21, 42)
            >>> point
     ```
  To add methods to custom named tuple classes:
    ```
    import math
    from collections import namedtuple
            
    class Point(namedtuple("Point", "x y")):
        __slots__ = ()
            
        def distance(self, other: "Point") -> float:
            return math.dist((self.x, self.y), (other.x, other.y))
    ```
  - Using typing.NamedTuple instead of collections.namedtuple() can make your code more readable by providing type hints. It also guarantees that your class is immutable by preventing you from adding new attributes and from changing the value of existing ones.
  ```
    import math
    from typing import NamedTuple
    
    class Point(NamedTuple):
        x: float
        y: float
    
        def distance(self, other: "Point") -> float:
            return math.dist((self.x, self.y), (other.x, other.y))
    ```
5. Creating Immutable Data Classes

- ```
    >>> from dataclasses import dataclass
    
    >>> @dataclass
    ... class Color:
    ...     red: int
    ...     green: int
    ...     blue: int
    ...
    
    >>> color = Color(255, 0, 0)
    >>> color
    Color(red=255, green=0, blue=0)
    
    >>> color.green = 128
    >>> color
    ```

 - You can change the attributes of an existing color, which means turning it into a different one.
To solve this issue, you can make your data class immutable by setting the frozen argument of @dataclass to True:

    ```
        >>> @dataclass(frozen=True)
        ... class Color:
        ...     red: int
        ...     green: int
        ...     blue: int
        ...
        
        >>> color = Color(255, 0, 0)
        >>> color
        Color(red=255, green=0, blue=0)
        
        >>> color.green = 128
        Traceback (most recent call last):
            ...
        dataclasses.FrozenInstanceError: cannot assign to field 'green'
    ```
### Sources
- [RealPython](https://realpython.com/python-mutable-vs-immutable-types/)