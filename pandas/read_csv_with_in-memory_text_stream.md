Use StringIO to create in-memory text stream.

```python
from io import StringIO
import pandas as pd

csv_data = """
name,age
Alice,49
Bob,2
Charlie,14
""".strip()

csv_file = StringIO(csv_data)
df = pd.read_csv(csv_file, dtype_backend="numpy_nullable")
```