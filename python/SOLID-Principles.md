# SOLID Principles
SOLID is an acronym that groups five core principles that apply to object-oriented design. These principles are the following:

1. Single-responsibility principle (SRP)
2. Open–closed principle (OCP)
3. Liskov substitution principle (LSP)
4. Interface segregation principle (ISP)
5. Dependency inversion principle (DIP)

## 1. Single-responsibility principle
The single-responsibility principle states that:

    A class should have only one reason to change

Having a single responsibility doesn’t necessarily mean having a single method.
Responsibility isn’t directly tied to the number of methods but to the core task that your class is responsible for, 
depending on your idea of what the class represents in your code. 
```
# bad
from pathlib import Path
from zipfile import ZipFile

class FileManager:
    def __init__(self, filename):
        self.path = Path(filename)

    def read(self, encoding="utf-8"):
        return self.path.read_text(encoding)

    def write(self, data, encoding="utf-8"):
        self.path.write_text(data, encoding)

    def compress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="w") as archive:
            archive.write(self.path)

    def decompress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="r") as archive:
            archive.extractall()
```

```
# good
from pathlib import Path
from zipfile import ZipFile

class FileManager:
    def __init__(self, filename):
        self.path = Path(filename)

    def read(self, encoding="utf-8"):
        return self.path.read_text(encoding)

    def write(self, data, encoding="utf-8"):
        self.path.write_text(data, encoding)

class ZipFileManager:
    def __init__(self, filename):
        self.path = Path(filename)

    def compress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="w") as archive:
            archive.write(self.path)

    def decompress(self):
        with ZipFile(self.path.with_suffix(".zip"), mode="r") as archive:
            archive.extractall()
```

## 2. Open–closed principle (OCP)
The Open–closed principle states that 

    Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.

```
from math import pi

class Shape:
    def __init__(self, shape_type, **kwargs):
        self.shape_type = shape_type
        if self.shape_type == "rectangle":
            self.width = kwargs["width"]
            self.height = kwargs["height"]
        elif self.shape_type == "circle":
            self.radius = kwargs["radius"]

    def calculate_area(self):
        if self.shape_type == "rectangle":
            return self.width * self.height
        elif self.shape_type == "circle":
            return pi * self.radius**2
```
To add a new shape_type we need to modify the class which violates the open-closed principle. To fix this:
```
from abc import ABC, abstractmethod
from math import pi

class Shape(ABC):
    def __init__(self, shape_type):
        self.shape_type = shape_type

    @abstractmethod
    def calculate_area(self):
        pass

class Circle(Shape):
    def __init__(self, radius):
        super().__init__("circle")
        self.radius = radius

    def calculate_area(self):
        return pi * self.radius**2

class Rectangle(Shape):
    def __init__(self, width, height):
        super().__init__("rectangle")
        self.width = width
        self.height = height

    def calculate_area(self):
        return self.width * self.height

...
```

## 3. Liskov substitution principle
The states Liskov substitution principle that

    Subtypes must be substitutable for their base types.

in other words

    Child class objects should be able to replace parent class objects without breaking the integrity of the application.

Inheritance starts as a cool idea. 
But over time, it’s easy to abuse inheritance. And when there are too many layers of inheritance, 
it’s natural to lose track of what specific classes are supposed to be doing. 
Eventually, some class gets added that can’t do something its parent classes need to do, and the system breaks.
```
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def notify(self, message, email):
        pass

class Email(Notification):
    def notify(self, message, email):
        print(f'Send {message} to {email}')

class SMS(Notification):
    def notify(self, message, phone):
        print(f'Send {message} to {phone}')

if __name__ == '__main__':
    notification = SMS()
    notification.notify('Hello', 'john@test.com')
```
The class SMS class violates the LSP principle because of the <i>phone</i> parameter.
This is fixed by
```
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def notify(self, message):
        pass

class Email(Notification):
    def __init__(self, email):
        self.email = email

    def notify(self, message):
        print(f'Send "{message}" to {self.email}')

class SMS(Notification):
    def __init__(self, phone):
        self.phone = phone

    def notify(self, message):
        print(f'Send "{message}" to {self.phone}')

class Contact:
    def __init__(self, name, email, phone):
        self.name = name
        self.email = email
        self.phone = phone

class NotificationManager:
    def __init__(self, notification):
        self.notification = notification

    def send(self, message):
        self.notification.notify(message)

if __name__ == '__main__':
    contact = Contact('John Doe', 'john@test.com', '(408)-888-9999')

    sms_notification = SMS(contact.phone)
    email_notification = Email(contact.email)

    notification_manager = NotificationManager(sms_notification)
    notification_manager.send('Hello John')

    notification_manager.notification = email_notification
    notification_manager.send('Hi John')
```

## 4. Interface segregation principle
The Interface segregation principle states that

    Clients should not be forced to depend upon methods that they do not use. Interfaces belong to clients, not to hierarchies.
 In other words, if a class doesn’t use particular methods or attributes, then those methods and attributes should be segregated into more specific classes.
```
# bad
from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

    @abstractmethod
    def fax(self, document):
        pass

    @abstractmethod
    def scan(self, document):
        pass

class OldPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in black and white...")

    def fax(self, document):
        raise NotImplementedError("Fax functionality not supported")

    def scan(self, document):
        raise NotImplementedError("Scan functionality not supported")

class ModernPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in color...")

    def fax(self, document):
        print(f"Faxing {document}...")

    def scan(self, document):
        print(f"Scanning {document}...")
```
```
# good
from abc import ABC, abstractmethod

class Printer(ABC):
    @abstractmethod
    def print(self, document):
        pass

class Fax(ABC):
    @abstractmethod
    def fax(self, document):
        pass

class Scanner(ABC):
    @abstractmethod
    def scan(self, document):
        pass

class OldPrinter(Printer):
    def print(self, document):
        print(f"Printing {document} in black and white...")

class NewPrinter(Printer, Fax, Scanner):
    def print(self, document):
        print(f"Printing {document} in color...")

    def fax(self, document):
        print(f"Faxing {document}...")

    def scan(self, document):
        print(f"Scanning {document}...")
```
If you have a violation of the Liskov Substitution Principle, it could very well be that you also have a violation of the Interface Segregation Principle!
<br>
This principle can lead to the problem of multiple inheritance. To solve this you can use composition over inheritance.
Refer [youTube](https://youtu.be/pTB30aXS77U?t=731)

## 5. Dependency inversion principle
The dependency inversion principle states that

    Abstractions should not depend upon details. Details should depend upon abstractions.
High-level Classes should not depend on Low-level Classes but both of them should depend on abstractions.
```
class FrontEnd:
    def __init__(self, back_end):
        self.back_end = back_end

    def display_data(self):
        data = self.back_end.get_data_from_database()
        print("Display data:", data)

class BackEnd:
    def get_data_from_database(self):
        return "Data from the database"
```

In this example, the FrontEnd class depends on the BackEnd class and its concrete implementation. You can say that both classes are tightly coupled.
This coupling can lead to scalability issues. For example, say that your app is growing fast, and you want the app to be able to read data from a REST API. 
<br>
To fix the issue, you can apply the dependency inversion principle and make your classes depend on abstractions rather than on concrete implementations like BackEnd.
```
from abc import ABC, abstractmethod

class FrontEnd:
    def __init__(self, data_source):
        self.data_source = data_source

    def display_data(self):
        data = self.data_source.get_data()
        print("Display data:", data)

class DataSource(ABC):
    @abstractmethod
    def get_data(self):
        pass

class Database(DataSource):
    def get_data(self):
        return "Data from the database"

class API(DataSource):
    def get_data(self):
        return "Data from the API"
```
```
>>> from app_dip import API, Database, FrontEnd

>>> db_front_end = FrontEnd(Database())
>>> db_front_end.display_data()
Display data: Data from the database

>>> api_front_end = FrontEnd(API())
>>> api_front_end.display_data()
Display data: Data from the API
```
### Sources
- [realpython](https://realpython.com/solid-principles-python/#single-responsibility-principle-srp)
- [pythontutorial](https://www.pythontutorial.net/python-oop/python-liskov-substitution-principle/)
- [openclassrooms](https://openclassrooms.com/en/courses/6900866-write-maintainable-python-code/7010225-l-for-the-liskov-substitution-principle)
- [ArjanCodes - YouTube](https://www.youtube.com/watch?v=pTB30aXS77U)