# Django, uWSGI, Nginx and Python3 in a container, using Supervisord.

This Dockerfile takes an existing Django app and deploys it for production using uWSGI and Nginx. I used most of the source code from https://github.com/dockerfiles/django-uwsgi-nginx, but I modified it to use Alpine instead of Ubuntu, the former being much lighter and in my opinion much cleaner.

I added a docker-compose file to ease deployment.


### How to deploy your application
Copy the root of your Django project into the app folder, make sure to include a `requirements.txt` file. 
You need to modify the following files:

- [uwsgi.ini]
Line module=your_app.wsgi:application. `your_app` needs to be the name of your django project, i.e. the folder name where wsgi.py lives.
- [nginx-app.conf]
Lines referring to static and media content. Use their full paths.


Then:

```
$ docker-compose build
$ docker-compose up
```

If that succeds, then `curl -vvv localhost:8113` should return something. 
