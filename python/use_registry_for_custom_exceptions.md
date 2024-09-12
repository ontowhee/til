# Use registry for custom exceptions

This may not be the best practice for writing custom exceptions, but it is just example code to illustrate the use of a registry for handling different cases of an exception. The example shows how to distinguish different sources of IntegrityError and to subsequently raise custom exceptions.

Let's say you have some database constraint on a table. This could be created with a Django model or with the sql create table statement.

For this example, we'll define the Person model with two CheckConstraints and a UniqueConstraint in the Meta class.

```python
# person/models.py
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=255)
    age = models.IntegerField(default=0)

    class Meta:
        constraints = [
            models.CheckConstraint(
                check=~(name__icontains='admin'),
                name='%(app_label)s_%(class)s_name_does_not_contain_admin',
            ),
            models.CheckConstraint(
                check=age__gte=18,
                name='%(app_label)s_%(class)s_age_gte_18',
            ),
            models.UniqueConsrtaint(
                fields=['name', 'age'],
                name='%(app_label)s_%(class)s_unique_name_and_age',
            )
        ]
```

Any time the Person model is created or modified such that it violates any of the db constraints, the database will raise an IntegrityError. For example, each of the following will raise the error:

```python
# Raise error due to person_person_name_does_not_contain_admin
Person.objects.create(name='Administrator', age=55)

# Raise error due to person_person_age_greater_than_18
Person.objects.create(name='Alice', age=10)

# Raise error on the second create due to person_person_unique_name_and_age
Person.objects.create(name='Bob', age=30)
Person.objects.create(name='Bob', age=30)
```

The error message might look like the following:

```
IntegrityError: new row for relation "person_person" violates check consraint "person_person_name_does_not_contain_admin" DETAIL: Failing row contains (Administrator, 55).
```

Let's say you have a function that is creating instances of Person using the Faker library, and it does not restrict the input values to satisfy the db constraints. It will be bound to raise some IntegrityErrors.

```python
from faker import Faker

def make_random_person():
    fake = Faker()
    return Person.objects.create(name=fake.name(), age=fake.random_int(0, 100))
```

One might add a try-except clause wherever this function is called to catch the IntegrityError and return a custom exception.

```python
try:
    make_random_person()
except IntegrityError as e:
    return Exception('Please check your Person data for invalid values.')
```

The problem with this approach is that it catches all IntegrityErrors and applies the same custom exception. It doesn't distinguish the constraint that was violated for the Person model. In addition, if there is more code in the `try` block that is creating other models, and those other models raise IntegrityErrors, the same custom exception that was written for the Person model is applied to the other models.

One way to distinguish the IntegrityErrors is to check the error message for the constraint name by accessing the `__cause__` attribute of the IntegrityError, and then raise a custom exception tailored for each one.

```python
try:
    make_random_person()
except IntegrityError as e:
    if 'person_person_name_does_not_contain_admin' in str(e.__cause__):
        raise Exception('Please check the value of the name field. It should not contain the substring "admin".')
    elif 'person_person_age_gte_18':
        raise Exception('Please check the value of the age field. It should not be less than 18.')
    elif 'person_person_unique_name_and_age':
        raise Exception('Please check the name and age fields. It should be unique.')
    else:
        # Raise the original exception
        raise
```

This is good so far, because it can handle each constraint violation with their own custom exception.

However, it can be improved by creating custom exception classes that subclass Exception. It can also create a registry class. The registry class provides a lookup that pairs an error message substring with the corresponding custom exception class.

```python
class CustomError(Exception):
    message = None
    def __str__(self):
        return self.message

class PersonNameAdminError(Exception):
    message = 'Please check the value of the name field. It should not contain the substring "admin".'

class PersonAgeError(Exception):
    message = 'Please check the value of the age field. It should not be less than 18.'

class PersonUniqueError(Exception):
    message = 'Please check the name and age fields. It should be unique.'

class PersonErrorRegistry():
    _errors = {
        'person_person_name_does_not_contain_admin': PersonNameAdminError,
        'person_person_age_gte_18': PersonAgeError,
        'person_person_unique_name_and_age': PersonUniqueError,
    }

    def get_error(self, original_exception):
        for error_message_substring, error_class in self._errors:
            if error_message_substring in str(original_exception.__cause__):
                # Raise the custom exception
                raise error_class()

        # Continue with the original exception
        raise
```

Now the try-except block can use the PersonErrorRegistry to lookup and return the appropriate custom exception.

```python
try:
    make_random_person()
except IntegrityError as e:
    raise PersonErrorRegistry().get_error(e)
```