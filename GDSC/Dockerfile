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
