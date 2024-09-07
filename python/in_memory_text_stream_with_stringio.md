# In-memory text stream with StringIO

```python
from io import StringIO

csv_data = """
name,age
Alice,49
Bob,2
Charlie,14
""".strip()

csv_file = StringIO(csv_data)

for line in f:
    print(line)
```