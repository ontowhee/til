# Dokku, SSL, Subdomain

Sources:
- https://mitjamartini.com/blog/2024/09/22/deploying-django-on-dokku/
- https://dokku.com/docs/deployment/application-deployment/#deploying-to-subdomains

```
dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git

dokku domains:add django-app <domain>
dokku letsencrypt:set django-app email <your email address for letsencrypt>
dokku letsencrypt:enable django-app
```

Also, for git, you'll want to add the remote as:
```
git remote add dokku dokku@<domain.tld>:<subdomain.domain.tld>
```

This way, it will push the app onto the subdomain. You'll be able to access it at `subdomain.domain.tld`.
