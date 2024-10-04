# Use match argument of pytest.raises() to check exception message

When writing tests and checking if an exception is raised, sometimes I like to check the exception message to narrow down on the specific cause.

Before migrating to pytest, I was writing tests using Django's [TestCase](https://github.com/django/django/blob/c334c1a8ff4579cdb1dd77cce8da747070ac9fc4/django/test/testcases.py#L1362). Under the hood, TestCase is a subclass of python's [unittest.TestCase](https://docs.python.org/3/library/unittest.html#unittest.TestCase).

When I want to check that an exception is raised, I would call the `TestCase.assertRaises()` method. If I want to check that the exception message matches a particular string or substring, I would need an additional assert statement.

```
from django.test import TestCase

class MyTest(TestCase):
    def test_divde_by_zero(self):
        self.assertRaises(Exception) as context:
            y = 1 / 0
        self.assertEqual(context.exception.message, "ZeroDivisionError: division by zero")
```

After migrating to pytest, I write tests as functions now. Instead of using `TestCase.assertRaises()` method, I now use [`pytest.raises()`](https://docs.pytest.org/en/7.1.x/reference/reference.html#pytest.raises). `match` takes in a string or a regular expression object.

I like how the `match` argument makes it simple to check the exception message all in one line, and the flexibility it provides by supporting regular expressions. There's no need for an additional assert statement.

```
def test_calc():
    with pytest.raises(Exception, match='ZeroDivision Error: division by zero'):
        y = 1 / 0
```