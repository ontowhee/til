# pyscopg2.errors.InFailedSqlTransaction

Reference: https://www.psycopg.org/docs/errors.html?#:~:text=InFailedSqlTransaction

Today my co-worker asked me to help troubleshoot an error they were encountering in one of the existing tests:

> psycopg2.errors.InFailedSqlTransaction: current transaction is aborted, commands ignored until end of transaction block

My first thought went to `@transaction.atomic`. Which blocks of code were wrapped in this decorator, and which were not? The one that wasn't wrapped in a transaction is likely failing.

I ran the test on my local machine and inspected the traceback.


```bash
======================================================================
ERROR: test_order_template (order.tests.test_views.OrderTestCase.test_order_template)
----------------------------------------------------------------------
Traceback (most recent call last):
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 108, in _execute
    return self.cursor.execute(sql, params)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
psycopg2.errors.InFailedSqlTransaction: current transaction is aborted, commands ignored until end of transaction block

...

    File "/myapp/context_processors.py", line 61, in locations_processor
    model = MyModel.objects.select_related('related_field').first()
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
...

    File "/myapp/order/views.py", line 552, in get_order
    return render(request, 'order/order_template.html',
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
...

django.db.utils.InternalError: current transaction is aborted, commands ignored until end of transaction block

```

<p>
<details>
  <summary>Full traceback</summary>
  <pre>
======================================================================
ERROR: test_order_template (order.tests.test_views.OrderTestCase.test_order_template)
----------------------------------------------------------------------
Traceback (most recent call last):
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 108, in _execute
    return self.cursor.execute(sql, params)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
psycopg2.errors.InFailedSqlTransaction: current transaction is aborted, commands ignored until end of transaction block

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
    File "/myapp/order/tests/test_views.py", line 223, in test_order_template
    response = self.client.get(reverse('order:get_order', kwargs={'pk': pk}))
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 1124, in get
    response = super().get(
                ^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 475, in get
    return self.generic(
            ^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 671, in generic
    return self.request(**r)
            ^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 1087, in request
    self.check_exception(response)
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 802, in check_exception
    raise exc_value
    File "/myapp/venv/lib/python3.12/site-packages/django/core/handlers/exception.py", line 55, in inner
    response = get_response(request)
                ^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/core/handlers/base.py", line 198, in _get_response
    response = wrapped_callback(request, *callback_args, **callback_kwargs)
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/contrib/auth/decorators.py", line 59, in _view_wrapper
    return view_func(request, *args, **kwargs)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/order/views.py", line 552, in get_order
    return render(request, 'order/order_template.html',
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/shortcuts.py", line 25, in render
    content = loader.render_to_string(template_name, context, request, using=using)
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/template/loader.py", line 62, in render_to_string
    return template.render(context, request)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/template/backends/django.py", line 107, in render
    return self.template.render(context)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/template/base.py", line 172, in render
    with context.bind_template(self):
    File "/usr/lib/python3.12/contextlib.py", line 137, in __enter__
    return next(self.gen)
            ^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/template/context.py", line 263, in bind_template
    context = processor(self.request)
                ^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/context_processors.py", line 61, in locations_processor
    model = MyModel.objects.select_related('related_field').first()
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/db/models/query.py", line 1145, in first
    for obj in queryset[:1]:
    File "/myapp/venv/lib/python3.12/site-packages/django/db/models/query.py", line 390, in __iter__
    self._fetch_all()
    File "/myapp/venv/lib/python3.12/site-packages/django/db/models/query.py", line 2000, in _fetch_all
    self._result_cache = list(self._iterable_class(self))
                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/db/models/query.py", line 95, in __iter__
    results = compiler.execute_sql(
                ^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/db/models/sql/compiler.py", line 1624, in execute_sql
    cursor.execute(sql, params)
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 79, in execute
    return self._execute_with_wrappers(
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 92, in _execute_with_wrappers
    return executor(sql, params, many, context)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 100, in _execute
    with self.db.wrap_database_errors:
    File "/myapp/venv/lib/python3.12/site-packages/django/db/utils.py", line 94, in __exit__
    raise dj_exc_value.with_traceback(traceback) from exc_value
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 105, in _execute
    return self.cursor.execute(sql, params)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
django.db.utils.InternalError: current transaction is aborted, commands ignored until end of transaction block

----------------------------------------------------------------------
Ran 1 test in 0.308s

FAILED (errors=1)
  </pre>
</details>
</p>


This line in the traceback,

```bash
    File "/myapp/context_processors.py", line 61, in locations_processor
    model = MyModel.objects.select_related('related_field').first()
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

tells us the error was raised when a query was made in the Django context processor, and this line,

```bash
    File "/myapp/order/views.py", line 552, in get_order
    return render(request, 'order/order_template.html',
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

tells us the context processor was called during the rendering of the template. However, the problem was neither in the context processor itself, nor was it in the template.

A closer look at **InFailedSqlTransaction** tells us that a previous SQL statement had been executed and failed. An explanation can be found in the [transaction management documentation for psycopg3](https://www.psycopg.org/psycopg3/docs/basic/transactions.html#:~:text=InFailedSqlTransaction):

> If a database operation fails with an error message such as **InFailedSqlTransaction**: current transaction is aborted, commands ignored until end of transaction block, it means that **a previous operation failed** and the database session is in a state of error. You need to call [rollback()](https://www.psycopg.org/psycopg3/docs/api/connections.html#psycopg.Connection.rollback) if you want to keep on using the same connection.

This means, while the error was raise on the context processor during the template render, the real failure happened before that.

How did I troubleshoot this? There were a lot of try-except statements in the code base. I started by removing the try-except clause from the block of code that contained call to `render(request, 'order/order_template.html', ...)`. I ran the test again and continued to see the error. I walked backwards to find a preceding function that was executing SQL queries and was also wrapped in a try-except statement. I removed the try-except statement, and reran the test. It was like peeling back a layer of an onion, one at a time until I started to cry with joy. Ok, not that dramatic. It only took me two blocks of code to uncover the failure point.

The new traceback revealed a more meaningful error message pointing at raw SQL in the code base.

```bash
psycopg2.errors.UndefinedTable: relation "product_inventory" does not exist LINE 1: ...sku, origin, price from product_in...
```

<p>
<details>
  <summary>Full traceback</summary>
  <pre>
======================================================================
ERROR: test_order_template (order.tests.test_views.OrderTestCase.test_order_template)
----------------------------------------------------------------------
Traceback (most recent call last):
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 103, in _execute
    return self.cursor.execute(sql)
            ^^^^^^^^^^^^^^^^^^^^^^^^
psycopg2.errors.UndefinedTable: relation "product_inventory" does not exist
LINE 1: ...sku, origin, price from product_in...
                                                                ^

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
    File "/myapp/order/tests/test_views.py", line 319, in test_order_template
    response = self.client.get(reverse('order:get_order', kwargs={'pk': pk}))
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 1124, in get
    response = super().get(
                ^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 475, in get
    return self.generic(
            ^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 671, in generic
    return self.request(**r)
            ^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 1087, in request
    self.check_exception(response)
    File "/myapp/venv/lib/python3.12/site-packages/django/test/client.py", line 802, in check_exception
    raise exc_value
    File "/myapp/venv/lib/python3.12/site-packages/django/core/handlers/exception.py", line 55, in inner
    response = get_response(request)
                ^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/core/handlers/base.py", line 198, in _get_response
    response = wrapped_callback(request, *callback_args, **callback_kwargs)
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/contrib/auth/decorators.py", line 59, in _view_wrapper
    return view_func(request, *args, **kwargs)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/order/views.py", line 460, in get_order
    medic_prod_list = get_product_inventory_list(
                        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/common/product_sorting.py", line 130, in get_product_inventory_list
    cursor.execute("""select product_id, sku, origin, price from product_inventory
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 79, in execute
    return self._execute_with_wrappers(
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 92, in _execute_with_wrappers
    return executor(sql, params, many, context)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 100, in _execute
    with self.db.wrap_database_errors:
    File "/myapp/venv/lib/python3.12/site-packages/django/db/utils.py", line 94, in __exit__
    raise dj_exc_value.with_traceback(traceback) from exc_value
    File "/myapp/venv/lib/python3.12/site-packages/django/db/backends/utils.py", line 103, in _execute
    return self.cursor.execute(sql)
            ^^^^^^^^^^^^^^^^^^^^^^^^
django.db.utils.ProgrammingError: relation "product_inventory" does not exist
LINE 1: ...sku, origin, price from product_in...
                                                                ^

----------------------------------------------------------------------
Ran 1 test in 0.262s

FAILED (errors=1)
  </pre>
</details>
</p>

Perhaps the models and raw SQL had become out of sync. I reported my findings to my co-worker and left them with the remaining investigation work.

I also advised them to check that the migration files on their local machine is in sync with the models in their working branch. The project does not track the migration files in the repository, unfortunately. Having mismatching migration files has bitten me many times.

Could or did my co-worker ask the LLM to figure out this problem? I didn't think to try it myself in the moment, given that I had a pretty good gut instinct on how to troubleshoot this, but now I'm curious...
