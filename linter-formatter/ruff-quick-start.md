# ruff quick start

Reference: https://docs.astral.sh/ruff/

I had the opportunity to introduce ruff to the engineering team recently. These are the examples I used in my demo to give a taste of what auto-linters and auto-formatters can do to increase developer productivity. The buy-in was quick!

### Array

```python
# example_array.py
a = [1, 2, 3, 4, 5]
b = [1, 2, 3, 4, 5,]
```

After running `ruff check`.

```
All checks passed!
```

After running `ruff format`.

```python
# example_array.py
a = [1, 2, 3, 4, 5]
b = [
    1,
    2,
    3,
    4,
    5,
]
```

The first array does not have a comma after the last element, so it is unchanged. The second array has a comma after the last element, so the code is reformatted to place each element on its own line.


### Not-in-test

```python
# example_conditional.py
if not(1 in [2,3,4,5,6]):
    print('hello')
else:
    print('goodbye')
```

After running `ruff check`.

```
example_conditional.py:2:8: E713 [*] Test for membership should be `not in`
  |
1 | # example_conditional.py
2 | if not(1 in [2,3,4,5,6]):
  |        ^^^^^^^^^^^^^^^^ E713
3 |     print('hello')
4 | else:
  |
  = help: Convert to `not in`
  
Found 1 error.
[*] 1 fixable with the `--fix` option.
```

After running `ruff check --fix`.

```python
# example_conditional.py
if 1 not in [2,3,4,5,6]:
    print('hello')
else:
    print('goodbye')
```

### Dict
```python
# example_dict.py
data = {'Email': 'bob@example.com', 'Username': 'bob.username', 'First Name': 'bob',
        'Last Name': 'bob', 'Is Manager': 'True', 'Is Admin': 'False', 'Is Active': 'False',
        'Department': 'Sales', 'Role': 'Sales Development Representative', 'tz': 'US/Eastern'}
```

After running `ruff check`.

```
All checks passed!
```

After running `ruff format`.

```python
# example_dict.py
data = {
    'Email': 'bob@example.com',
    'Username': 'bob.username',
    'First Name': 'bob',
    'Last Name': 'bob',
    'Is Manager': 'True',
    'Is Admin': 'False',
    'Is Active': 'False',
    'Department': 'Sales',
    'Role': 'Sales Development Representative',
    'tz': 'US/Eastern'
}
```

### Function call line breaks

```python
# example_function_call_line_breaks.py
def calculate(first_parameter, second_parameter, third_paramter, fourth_parameter, fifth_parameter):
    return first_parameter * second_parameter - third_paramter / fourth_parameter + fifth_parameter


calculate(first_parameter=11111, second_parameter=22222, third_paramter=33333, fourth_parameter=44444,
          fifth_parameter=55555)
```

The line-length is set to be 120 characters, and this code has a line break after the fourth parameter.

After running `ruff check`.

```
All checks passed!
```

After running `ruff format`.

```python
# example_function_call_line_breaks.py
def calculate(first_parameter, second_parameter, third_paramter, fourth_parameter, fifth_parameter):
    return first_parameter * second_parameter - third_paramter / fourth_parameter + fifth_parameter


calculate(
    first_parameter=1111111,
    second_parameter=2222222,
    third_paramter=3333333,
    fourth_parameter=4444444,
    fifth_parameter=5555555,
)
```

The reformatted code has a line break after every parameter.

### Sort import

```python
# example_isort.py
import sys
import os
import datetime

from itertools import product, pairwise, zip_longest, batched, combinations_with_replacement, starmap, chain, \
    permutations
from functools import lru_cache, partial, cache


# Add a bunch of print statements to avoid the "unused imports" linting error
print(sys)
print(os)
print(datetime)

print(lru_cache)
print(partial)
print(cache)

print(product)
print(pairwise)
print(zip_longest)
print(batched)
print(combinations_with_replacement)
print(starmap)
print(chain)
print(permutations)
```

After running `ruff check`.

```
example_isort.py:1:1: I001 [*] Import block is un-sorted or un-formatted
   |
 1 | / import sys
 2 | | import os
 3 | | import datetime
 4 | | 
 5 | | from itertools import product, pairwise, zip_longest, batched, combinations_with_replacement, starmap, chain, \
 6 | |     permutations
 7 | | from functools import lru_cache, partial, cache
 8 | | 
 9 | | 
10 | | print(sys)
   | |_^ I001
11 |   print(os)
12 |   print(datetime)
   |
   = help: Organize imports

Found 1 error.
[*] 1 fixable with the `--fix` option.
```

After running `ruff check --fix`.

```python
import datetime
import os
import sys
from functools import cache, lru_cache, partial
from itertools import (
    batched,
    chain,
    combinations_with_replacement,
    pairwise,
    permutations,
    product,
    starmap,
    zip_longest,
)

print(sys)
print(os)
print(datetime)

print(lru_cache)
print(partial)
print(cache)

print(product)
print(pairwise)
print(zip_longest)
print(batched)
print(combinations_with_replacement)
print(starmap)
print(chain)
print(permutations)
```

### Long string

```python
# example_long_string.py
long_string = 'really long line .......................................................................................................'
```

After running `ruff check`.

```
example_long_string.py:2:121: E501 Line too long (136 > 120)
  |
1 | # example_long_string.py
2 | long_string = 'really long line .......................................................................................................'
  |                                                                                                                         ^^^^^^^^^^^^^^^^ E501
  |

Found 1 error.
```

In some of the previous examples we saw the suggestion "1 fixable with the \`--fix\` option." Notice that we do not see the suggestion here. Breaking up a long string into a multi-line string is not supported yet. See https://github.com/astral-sh/ruff/issues/9634.

After running `ruff format`.

```
1 file left unchanged
```

### Unused import

```python
# example_unused_import.py
import os

print('hello')
```

After running `ruff check`.

```
example_unused_import.py:1:8: F401 [*] `os` imported but unused
  |
1 | import os
  |        ^^ F401
2 | 
3 | print('hello')
  |
  = help: Remove unused import: `os`

Found 1 error.
```

After running `ruff check --fix`.

```python
# example_unused_import.py

print('hello')
```

### pyproject.toml
This was the configuration I used for the examples.

```
# pyproject.toml
[project]
name = "ruff-demo"
version = "0.0.1"
dependencies = [
    "ruff==0.6.5",
]
requires-python = ">= 3.12"

[tool.ruff]
# Allow lines to be as long as 120.
line-length = 120

[tool.ruff.format]
quote-style = "single"

[tool.ruff.lint]
select = [
    # pycodestyle
    "E",
    # Pyflakes
    "F",
    # pyupgrade
    "UP",
    # flake8-bugbear
    "B",
    # flake8-simplify
    "SIM",
    # isort
    "I",
]
```
