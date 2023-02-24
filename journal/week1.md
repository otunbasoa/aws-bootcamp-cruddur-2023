# Week 1 — App Containerization

we started off the live stream with guest tutors Edith pucilla and james spurin.

I learnt about docker hub, this is  where docker images are pushed to or pulled from and its free to use

I learnt how to create a dockerfile and wrote my first dockerfile. see code below;

```
FROM python:3.10-slim-buster

# Inside container
# make a new folder inside container

WORKDIR /backend-flask

# Outside container -> Insiide container
# this contains the libraries you want to install to run the app

COPY requirements.txt requirements.txt

# Inside container
# Install the python libraries used for the app
RUN pip3 install -r requirements.txt
# outsiide container -> Inside container
# . means everything in the current directory
# first period . represents /backend-flask (inside container)
# second period . /backend-flask (inside container)
COPY . .
# set environment variables (Env vars)
# Insiide container and will remain set when the container is run
ENV FLASK_ENV=development

EXPOSE ${PORT}

# CMD(command)
# 
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```



