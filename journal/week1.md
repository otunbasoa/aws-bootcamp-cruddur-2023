# Week 1 — App Containerization

we started off the live stream with guest tutors Edith pucilla and james spurin.

I learnt about docker hub, this is  where docker images are pushed to or pulled from and its free to use

I learnt how to create a dockerfile and wrote my first dockerfile for back-end and front-end. see code below;

## Bank-end Dockerfile
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

## Front-end Dockerfile

```
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

I built my first image

Instead of writing multiple docker files as seen above, i was introduced to docker-compose. The purpose of docker-compose file is to allow us to run multiplr containers at the same time. see code below;

```
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```





