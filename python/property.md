# Property
Python’s <i>property()</i> is the Pythonic way to avoid formal getter and setter methods in your code. 
This function allows you to turn class attributes into properties or managed attributes.

```
class WriteCoordinateError(Exception):
    pass

class Point:
    def __init__(self, x, y):
        self._x = x
        self._y = y

    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, value):
        raise WriteCoordinateError("x coordinate is read-only")

    @property
    def y(self):
        return self._y

    @y.setter
    def y(self, value):
        raise WriteCoordinateError("y coordinate is read-only")
```
> Note: This will make attributes read only, however, you can still set ```Point._x``` but not ```Point.x```
## When to use
1. When you want to make your attributes read-only or write-only
2. When you have additional logic like data validation each time you set a value
or data logging when you set/get a value
```
class Point:
    def __init__(self, x):
        self.x = x

    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, value):
        try:
            self._x = float(value)
            print("Validated!")
        except ValueError:
            raise ValueError('"x" must be a number') from None
```

```
import logging

logging.basicConfig(
    format="%(asctime)s: %(message)s",
    level=logging.INFO,
    datefmt="%H:%M:%S"
)

class Circle:
    def __init__(self, radius):
        self._msg = '"radius" was %s. Current value: %s'
        self.radius = radius

    @property
    def radius(self):
        """The radius property."""
        logging.info(self._msg % ("accessed", str(self._radius)))
        return self._radius

    @radius.setter
    def radius(self, value):
        try:
            self._radius = float(value)
            logging.info(self._msg % ("mutated", str(self._radius)))
        except ValueError:
            logging.info('validation error while mutating "radius"')
```

3. When you want to delete an attribute. <br>
In rare cases,
```
class TreeNode:
    def __init__(self, data):
        self._data = data
        self._children = []

    @property
    def children(self):
        return self._children

    @children.setter
    def children(self, value):
        if isinstance(value, list):
            self._children = value
        else:
            del self.children
            self._children.append(value)

    @children.deleter
    def children(self):
        self._children.clear()

    def __repr__(self):
        return f'{self.__class__.__name__}("{self._data}")'
```
```
>>> from tree import TreeNode

>>> root = TreeNode("root")
>>> child1 = TreeNode("child 1")
>>> child2 = TreeNode("child 2")

>>> root.children = [child1, child2]

>>> root.children
[TreeNode("child 1"), TreeNode("child 2")]

>>> del root.children
>>> root.children
[]
```

4. When you want to cache expensive results
```
from functools import cached_property
from time import sleep

class Circle:
    def __init__(self, radius):
        self.radius = radius

    @cached_property
    def diameter(self):
        sleep(0.5)  # Simulate a costly computation
        return self.radius * 2
```

```
>>> circle = Circle(42.0)
>>> circle.diameter  # With delay
84.0
>>> circle.diameter  # Without delay
84.0

>>> circle.radius = 100
>>> circle.diameter  # Wrong diameter
84.0

>>> # Allow direct assignment
>>> circle.diameter = 200
>>> circle.diameter  # Cached value
200
```
The problem here is unlike property(), cached_property() doesn’t block attribute mutations unless you provide a proper setter method.
If you want to create a cached property that doesn’t allow modification, then you can use property() and functools.cache().
```
from functools import cache

class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    @cache
    def diameter(self):
        sleep(0.5) # Simulate a costly computation
        return self.radius * 2
```
```
>>> circle = Circle(42.0)

>>> circle.diameter  # With delay
84.0
>>> circle.diameter  # Without delay
84.0

>>> circle.radius = 100
>>> circle.diameter
84.0

>>> circle.diameter = 200
Traceback (most recent call last):
    ...
AttributeError: can't set attribute
```
5. When you want backward compatibility
```
class Person():

    def __init__(self, firstname, lastname):
        self.first = firstname
        self.last = lastname
        self.fullname = self.first + ' '+ self.last

    def email(self):
        return '{}.{}@email.com'.format(self.first, self.last)
```
If you set lastname after initialization, the fullname attribute is untouched.
```
>>> person = Person('uttejh', 'reddy')
>>> person.last = 'jeereddy'
>>> person.last
jeereddy

>>> person.fullname    # old/unexpected value
uttejh reddy

>>> person.email()     # dynamic - expected value
uttejh.jeereddy@email.com
```
To solve this we can turn attribute to a method like
```
def fullname(self):
        return self.first + ' '+ self.last
```
but this would break code which uses ```Person.fullname``` instead of ```Person.fullname()```.
To solve this, we just need to convert it into a property by
```
@property    
def fullname(self):
    return self.first + ' '+ self.last
```

### Sources
- [RealPython](https://realpython.com/python-property/#putting-pythons-property-into-action)
- [machinelearningplus](https://www.machinelearningplus.com/python/python-property/)
