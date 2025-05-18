# Hetzner, Dokku, Django

I combined deployment steps from the following references:
- https://dev.to/devasservice/how-to-deploy-django-on-a-budget-with-hetzner-and-dokku-1484 - The bulk of the steps come from this post.
- https://railsnotes.xyz/blog/deploying-ruby-on-rails-with-dokku-redis-sidekiq-arm-docker-hetzner - There were a lot of similarities between this post and the one above. I don't remember exactly which pieces I borrowed that were not present in the first. It was interesting to see the price and compute resource comparisons.
- https://mitjamartini.com/blog/2024/09/22/deploying-django-on-dokku/ - The dokku_config.py was really helpful. This is a simple script that allows you to set environment variables for dokku on the remote server really easily. It takes in a .env file and loops through each variable to run the `dokku config:set` command. This blog post also walks through some more advanced stuff that I haven't used.

This article talks about deploying with docker instead of Procfile. I haven't gone through it yet, and don't need it now, but wanted to note down this option.
- https://ruby.mobidev.biz/posts/simplifying-application-deployment-with-dokku/


# On local machine
ssh into your server.
```sh
ssh root@<your-hetzner-server-ip-address>
```

# On the server

Update the packages on the system.
```sh
apt update && apt upgrade -y

# I don't know if this is necessary, but one of the blogs mentioned restarting server
sudo reboot now
```

Install dokku.
```sh
wget -NP . https://dokku.com/install/v0.35.18/bootstrap.sh
sudo DOKKU_TAG=v0.35.18 bash bootstrap.sh
```

Add ssh key that you generated on your local machine. In my case, I didn't have to run this command, because I uploaded my key via the console on Hetzner when creating the server.
```sh
# add our own ssh key so we can git push to our dokku instance
cat ~/.ssh/authorized_keys | dokku ssh-keys:add admin
```

```sh
# add our server IP to dokku domains, so we can visit them in our browser
dokku domains:set-global your.server.ip.address
dokku domains:set-global your.server.ip.address.sslip.io
```

```sh
# create dokku app
dokku apps:create django-app
```

```sh
# install plugin for postgres
dokku plugin:install https://github.com/dokku/dokku-postgres.git

# create postgres database
dokku postgres:create django-app-db

# link django app to the postgres db.
# That is, database url is added to environment variables for dokku app
dokku postgres:link django-app-db django-app

# view the environment variables for dokku app
dokku config:show django-app
```

# On local machine

### Set additional environment variables for remote server
Set more environment variables using dokku_config.py from https://mitjamartini.com/blog/2024/09/22/deploying-django-on-dokku/, which was borrowed from https://github.com/Tobi-De/dokku-envs.
```python
# dokku_config.py
import os
import click

# This is based on https://github.com/Tobi-De/dokku-envs

def run_commands(command, host, app, env_dict):
    for env_name, env_value in env_dict.items():
        os.system(command.format(host_name=host, app_name=app, env_name=env_name, env_value=env_value))


def read_env_file(env):
    f = env.readlines()
    return {line.strip().split("=")[0]: line.strip().split("=")[1] for line in f if "=" in line}


@click.command()
@click.argument(
    "host"
)
@click.argument(
    "app"
)
@click.argument(
    "env",
    type=click.File("r"),
)
def set_dokku_app_envs(host, app, env):
    """
    You should have click installed.
    ```
    pip install click
    ```

    Usage:
    ```
    python dokku_config.py $DOKKU_HOST $APP_NAME .env.prod
    ```


    Set environment variables of a Dokku app to values defined in an env file.

    This runs locally and assumes you can use your Dokku server via SSH, and
    execute commands like `ssh dokku@<server> config:set <app> <env_name>=<env_value>`.
    """
    command = "ssh dokku@{host_name} config:set {app_name} {env_name}={env_value}"
    env_dict = read_env_file(env=env)
    if not env_dict:
        return
    run_commands(command, host, app, env_dict)


if __name__ == "__main__":
    set_dokku_app_envs()

```

Create .env.production to contain additional environment variables. Postgres variables were already created and managed by dokku earlier, and now we need Django secret key.
```sh
# .env.production
DJANGO_SECRET_KEY="secret..."
```

Create a .env file to hold variables for your dokku host. I haven't used APP_DOMAIN yet, since I'm using sslip.io
```sh
# .env.dokku
export ADMIN_USER="dokku"
export DOKKU_HOST=<server-ip-address>
# export APP_DOMAIN=<the-domain-name-of-your-app> # this is your domain name, such as mywebsite.com
export APP_NAME="django-app"
```

Apply the envvars
```sh
source .env.dokku
```

Run the dokku_config.py script.
```sh
python dokku_config.py $DOKKU_HOST $APP_NAME .env.prod
```

### Create django app on local machine
Create django app on your local machine. Make sure your django project directory structure has manage.py and the settings folder at the root:
- / # <-- Root of project
	- simple_django/
		- settings.py
	- manage.py
	- Procfile
	- requirements.txt

Install a few items that will be used for deployment
```sh
pip install python-decouple dj-database-url gunicorn whitenoise psycopg2-binary
```
- python-decouple - helps with reading environment variable
- dj-database-url - helps with using db url for the database settings
- gunicorn - server
- whitenoise - static files
- psycopg2-binary - for postgresql on your remote server

Create .env file and add the database url:
```sh
# .env
DATABASE_URL=sqlite:///db.sqlite3
```
You are configuring the local app to use SQLite, although it will use the credentials created by dokku on your remote server

Update settings.py:
```py
import dj_database_url
import os
from decouple import config
from django.conf.global_settings import DATABASES

...

ALLOWED_HOSTS = ['*']

...

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    "whitenoise.middleware.WhiteNoiseMiddleware",
    ... all others middleware ...
]

...

DATABASES['default'] = dj_database_url.parse(config('DATABASE_URL'), conn_max_age=600)

...

STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / "static"
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'staticfiles'),
)
STATICFILES_STORAGE = 'whitenoise.storage.CompressedStaticFilesStorage'
```
These changes are:
- ALLOWED_HOST - allow anyone to access. We'll change this later to lock it down.
- MIDDLEWARE - add whitenoise
- DATABASES - use dj database url
- STATIC_* - static file settings using whitenoise

### Create files used by dokku to build and run your app
Create a Procfile. This is how dokku runs your app:
```sh
web: gunicorn simple_django.wsgi

release: python manage.py migrate
```

Update requirements.txt:
```sh
pip freeze > requirements.txt
```

### Deployment
Push the project to github!
```sh
echo "# Simple Django" >> README.md
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin [Your GitHub repository URL]
git push -u origin main
```

Add git remote for dokku. This allows you to do `git push dokku ...`:
```
git remote add dokku dokku@<your_server_ip>:<your_dokku_app_name>
```

Deploy:
```sh
# deploy
git push dokku
```

If all went well, you should have an app at the url specified in the output.
