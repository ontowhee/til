# Sort lists in place using the built-in list.sort() method
Reference: https://docs.python.org/3/howto/sorting.html

I just learned this recently that python lists have a built-in `list.sort()`.

```python
>>> arr = [5, 3, 6, 2]
>>> arr.sort()
>>> arr
[2, 3, 5, 6]
```

I have been using the built-in `sorted()` function all this time.

```python
>>> arr = [5, 3, 6, 2]
>>> arr_sorted = sorted(arr)
>>> arr
[5, 3, 6, 2]
>>> arr_sorted
[2, 3, 5, 6]
```

`list.sort()` will sort in place, while `sorted()` will return a new list and leave the original unmodified.
