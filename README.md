# django-deploy-guides
Guides on how to deploy a Django app on a server like an AWS EC2

I learned Django and developed an app locally, which was fun. Deploying it into production is hell, though. So I started this repository where I plan to collect a few guides as Markdown files, for deploying Django using different methods (e.g. Apache + mod_wsgi, or Nginx + gunicorn, or uwsgi)

I'll use [this app](https://github.com/AlexEngelhardt/hired-gun) as an example for deployment in the guides - it's my project for a freelancer time tracking application.

The [Deployment checklist](https://docs.djangoproject.com/en/2.1/howto/deployment/checklist/) is a crucial help there. The `python3 manage.py check --deploy` command is helpful too.
