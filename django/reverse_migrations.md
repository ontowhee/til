# Revert migrations

Reference: https://docs.djangoproject.com/en/5.1/topics/migrations/#reversing-migrations

To reverse migrations from a file and onwards, specify the previous file in the command:
```
$ python manage.py migrate <app_name> <migration_name>
```

For example, given an app `blog` with the following model,

```bash
class Blog(models.Model):
    name = models.CharField()
    created_date = models.DateField()
    modified_date = models.DateField()
```

and migration files,
```bash
$ ls blog/migrations/
0001_initial.py
0002_post_created_date.py
003_post_modified_date.py
```

To reverse migrations from file `0003`, specify `0002` as the migration_name:
```bash
$ python manage.py migrate blog 0002
Operations to perform:
  Target specific migration: 0002_post_created_date, from blog
Running migrations:
  Rendering model states... DONE
  Unapplying blog.0003_post_modified_date... OK
```

To reverse migrations from _all_ files, specify `zero` as the migration_name:
```bash
$ python manage.py migrate blog zero
Operations to perform:
  Unapply all migrations: blog
Running migrations:
  Rendering model states... DONE
  Unapplying blog.0003_post_modified_date... OK
  Unapplying blog.0002_post_created_date... OK
  Unapplying blog.0001_initial... OK
```
