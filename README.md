# containerize a python web application





## introduction

- this document is part of [GDSC at the university of bahrain](https://gdsc.community.dev/university-of-bahrain/)
- you don't need to follow along, fell free to refer to this document later. 
![enter image description here](https://media3.giphy.com/media/3oEjHCF6kGlXK0ofsY/giphy.gif?cid=ecf05e471uamk0tok5igc4z2hgvfar8h3sdhr8z2b50k712q&rid=giphy.gif&ct=g)

## agenda

 - you probably heard about docker, Kubernetes, and container.
 - create a web application using Django.
 - explore deployment methods. and development environment challenges.
- docker ( what why how ?)
-	Dockerizing our lovely Django app. 
-	what's next

.

## Django web framework

### what's Django
- it's a free and open-source python web frameworks, that uses MTV - Model Template View ( derived from MVC )
- 	![what is mvc](https://en.wikipedia.org/wiki/File:MVC-Process.svg)
	- you can get up and running quickly. 
	- very good for MVPs ( minimum viable product )
	- scalable.
	- who uses Django ?  
		- pintrest, instagram, coursera
		- me !  checkout [cgine.app](https://cgine.app/)

**NOTE :** im using Linux, if you are using windows, installation process can be a little different, but if you  are using mac they should be similar.

first of all, it's a good practice to create your own virtual environment.
   
   ```
   $ virtualenv venv
   ``` 
	
activate venv

```
$ source venv/bin/activate
```

install django
```
$ pip install django
```

start a new project using django-admin, and a new application
```
$ django-admin startproject GDSC
$ python manage.py startapp main
```

cd to the GDSC file & migrate.
```
$ cd GDSC              
$ python manage.py migrate
 ```

run our python application
```
$ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
October 19, 2021 - 07:51:41
Django version 3.2.8, using settings 'GDSC.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
you can navigate to [http://127.0.0.1:8000/](http://127.0.0.1:8000/) and confirm that your app is running.

now we need to create a  requirements.txt , which is a  file that lists all of the modules needed for the Django project to work. and write this in it ( it should at the same direcotry as manage.py)
```
 Django==3.2.8
 ```

so now we have up and running django project, it's **very** simple, but will we be containerize it.

## explore deployment methods.

so now if we want to deploy our django app, we would get a linux server set up nginx and gunicorn ( HTTP servers), and pray to good that everything work, and to make that app scaleble and able to handle huge traffic, we would set up another server ( load balancing). 

and during development, you would spend hours trying to setup your development environment, from installing dependencies to setting up a databse. postgres & redis ( DB & caching) .

## docker.
coming back to the deployment example, the concept of Hardware server is vanishing thanks to the cloud, 
we now have some sort-of a software server called containers.

> the concept of a server could be removed from the constraints of hardware and instead became, essentially, a piece of software. These software-based servers are called [containers](https://opensource.com/article/18/1/history-low-level-container-runtimes), and they're a [hybrid mix](https://opensource.com/article/18/11/behind-scenes-linux-containers) of the Linux OS they're running on plus a hyper-localized runtime environment (the contents of the container). [open source] (https://opensource.com/resources/what-docker)

containers consist of three main elements:
1- builders : that build the container i.e dockerfile
2- engine : and application that runs the container i.e docker command
3-Orchestration : technology that manages many container i.e Kubernetes.

so using container will help us deploy the application and its configuration using one image, that can be reused anywhere, this is also helpful for developers because all team members will have consistent environments and dependencies.

now we will use docker to deploy and run our django application.
first you will need docker ( obviously )
https://docs.docker.com/get-docker/
if you are using 
-	windows : you can just download the desktop version, ( make sure to add it to PATH variable ) 
	- confirm installtion by typing docker in cmd. 
- mac : same as windows, get the desktop version.
- linux : ...............

** NOTE: ** we will use docker CLI, you can use GUI but it's always better to learn using the  cli first.

now we need to create a builder which is the docker file. 
docker file is  :  a text document that contains all the commands a user could call on the command line to assemble an image.  its similar to setting up an environment on a Linux server.

docker file should be in the same directory as manage . py

the basic syntax of a docker file is 
```
INSTRUCTION  arguments 
```
```
# the offical apline base image
FROM python:3.9.6-alpine

# set up a work directory ( for the app )
WORKDIR /usr/src/GDSC

#set up enviroment variables.
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# copy project
COPY . .

```



what is alpine? and why
its a linux distribution that is small and fast, and we are using an image that is based on it. it uses less disk space and faster to deploy. 
refer to this [talk](https://mherman.org/presentations/dockercon-2018/#1) given by micheal herman about best practices for docker python


now we create the docker compose file ( docker-compose.yml)
what is docker-compose ?
a tool for defining and running multi-container Docker applications and define the services that make up your app

docker-compose.yml should be in the root directory  ( where venv is )
```
version: '3.8'

services:
  web:
    build: ./GDSC
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./GDSC/:/usr/src/GDSC/
    ports:
      - 8000:8000
    

```
to recap.
-	 Dockerfile is a simple text file that contains the commands a user could call to assemble an image
-	Docker Compose is a tool for defining and running multi-container Docker applications (
- what is the relation between them ? 
	- docker compose defines the services that make up you app and uses the docker file. 
- what do you mean by services? 
	- part of your application  ( database, API, etc)  and have to be in a single container.
	-  for example `web` service which we used above, it uses an image that’s built from the `Dockerfile` in the current directory. It then binds the container and the host machine to the exposed port, `8000`.  so currently we have only one service. 
see [ docker compose docs](https://docs.docker.com/compose/)



so your files should look like this 
```
.
├── docker-compose.yml
├── GDSC
│   ├── db.sqlite3
│   ├── Dockerfile
│   ├── GDSC
│   │   ├── asgi.py
│   │   ├── __init__.py
│   │   ├── __pycache__
│   │   │   ├── __init__.cpython-39.pyc
│   │   │   ├── settings.cpython-39.pyc
│   │   │   ├── urls.cpython-39.pyc
│   │   │   └── wsgi.cpython-39.pyc
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── main
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   └── __init__.py
│   │   ├── models.py
│   │   ├── tests.py
│   │   └── views.py
│   ├── manage.py
│   └── requirements.txt
```

and then we build the image and run it.
if you are running docker desktop you already have docker compose installed if not refer to this page https://docs.docker.com/compose/install/

```
$ docker-compose build
$ docker-compose up -d
```

until now we have been working with Django development server, but what if we want to deploy our application using a production grade server ? 

## configuring for production
- we can use the fast,easy,cheap but manual and not scalable way.
- or we can use the more complex, reliable, scalable way.

for the sake of this tutorial we are going to use the first way. 

### adding Gunicorn
[Gunicorn](https://gunicorn.org/), a production-grade WSGI server, that is reliable and fast.
so we will need to add to our requirements.txt file
```
Django==3.2.8
gunicorn==20.1.0
```
so now we will create a new docker-compose file for the production environment, note that we can still use the old one for development. we will call it docker-compose.prod.yml
for more information about why we added .prod refer to  https://docs.docker.com/compose/extends/
```
version: '3.8'

services:
  web:
    build: ./GDSC
    command: gunicorn GDSC.wsgi:application --bind 0.0.0.0:8000
    ports:
      - 8000:8000
    env_file:
      - ./.env.prod
```
we changed the command from running default django server, to running gunicorn server. 

we create a new .dev.prod file that contains environment variables. 
```
SECRET_KEY = foo
DEBUG=1
```

if our development container are still running we need to spin them down before running this one 
```
$ docker-compose down -v
```

and build & run our new docker compose
```
$ sudo docker-compose -f docker-compose.prod.yml up -d --build
```
navigate to http://localhost:8000/
you will see everything working but these is something odd. static files are not served !!
to do that we will need NGINX to act as a [reverse proxy](https://www.nginx.com/resources/glossary/reverse-proxy-server/) for Gunicorn to handle client requests as well as serve up static files.

what is a proxy server ? 
A proxy server is a go‑between or intermediary server that forwards requests for content from multiple clients to different servers across the Internet.

###  adding nginx
we are going to need a new server so where we should add that ? 
to the docker-compose.prod.yml right ? remember the job of docker compose is to define all of the app service. so it should look like this after adding nginx.

```
version: '3.8'

services:
  web:
    build: ./GDSC
    command: gunicorn GDSC.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - ./GDSC/:/usr/src/GDSC/
    ports:
      - 8000:8000
    env_file:
      - ./.env.prod
  nginx:
    build: ./nginx
    ports:
      - 1337:80
    depends_on:
        - web
```
now we need to add nginx configuration files in the root folder ( where docker-compose is )
so its should look like this 

```
.
├── docker-compose.prod.yml
├── docker-compose.yml
├── .env.dev
├── .env.prod
├── GDSC
│   ├── db.sqlite3
│   ├── Dockerfile
│   ├── GDSC
│   │   ├── asgi.py
│   │   ├── __init__.py
│   │   ├── __pycache__
│   │   │   ├── __init__.cpython-39.pyc
│   │   │   ├── settings.cpython-39.pyc
│   │   │   ├── urls.cpython-39.pyc
│   │   │   └── wsgi.cpython-39.pyc
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── main
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── __init__.py
│   │   ├── migrations
│   │   │   └── __init__.py
│   │   ├── models.py
│   │   ├── tests.py
│   │   └── views.py
│   ├── manage.py
│   └── requirements.txt
├── nginx
│   ├── Dockerfile
│   └── nginx.conf
```

in nginx/Dockerfile 
```
```
FROM nginx:1.21-alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d
```
```

and in nginx.conf
```
upstream GDSC {
    server web:8000;
}
server {
    listen 80;
    location / {
        proxy_pass http://GDSC;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
    
}
```

now we replace ports with expose in the docker-compose.prod.yml file
```
version: '3.8'

services:
  web:
    build:
      context: ./GDSC
      dockerfile: Dockerfile.prod
    command: gunicorn GDSC.wsgi:application --bind 0.0.0.0:8000
    volumes:
      - static_volume:/home/GDSC/web/staticfiles
    expose:
      - 8000
    env_file:
      - ./.env.prod
  nginx:
    build: ./nginx
    volumes:
      - static_volume:/home/GDSC/web/staticfiles
    ports:
      - 1337:80
    depends_on:
        - web
volumes:
  static_volume:
```


quick test to insure everything is running
```
$ docker-compose -f docker-compose.prod.yml down -v
$ docker-compose -f docker-compose.prod.yml up -d --build
```

now spin down the container one more time. because we need to add somethings .
```
$ docker-compose -f docker-compose.prod.yml down -v
```
to make sure that static files can be served in development environment add this to settings.py 
```
STATIC_ROOT = BASE_DIR / "staticfiles"
```

now we will create Dockerfile.prod ( in the same directory as Dockerfile) 

```
###########
# BUILDER #
###########

# pull official base image
FROM python:3.9.6-alpine as builder

# set work directory
WORKDIR /usr/src/GDSC

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1





# install dependencies
COPY ./requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/GDSC/wheels -r requirements.txt


#########
# FINAL #
#########

# pull official base image
FROM python:3.9.6-alpine

# create directory for the app user
RUN mkdir -p /home/GDSC

# create the app user
RUN addgroup -S GDSC && adduser -S GDSC -G GDSC

# create the appropriate directories
ENV HOME=/home/GDSC
ENV APP_HOME=/home/GDSC/web
RUN mkdir $APP_HOME
RUN mkdir $APP_HOME/staticfiles
WORKDIR $APP_HOME

# install dependencies
RUN apk update && apk add libpq
COPY --from=builder /usr/src/GDSC/wheels /wheels
COPY --from=builder /usr/src/GDSC/requirements.txt .
RUN pip install --no-cache /wheels/*


# copy project
COPY . $APP_HOME

# chown all the files to the app user
RUN chown -R GDSC:GDSC $APP_HOME

# change to the app user
USER GDSC
```
lastly we need to add static files location to nginx.conf
```
upstream GDSC {
    server web:8000;
}
server {
    listen 80;
    location / {
        proxy_pass http://GDSC;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
    location /static/ {
        alias /home/GDSC/web/staticfiles/;
    }
}
```
## now we have a fully working django app !.
