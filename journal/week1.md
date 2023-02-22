# Week 1 — App Containerization

## Building the Backend container with python

### create the environment variables & configure the host and port 

```
cd backend-flask
run pip3 install -r requirements.txt
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```

![python_envirnment](assets/python_environ.png)

*Get to the port and unlock port 4567 and click the link and the append /api/activities/home to the url in the browser*

## Building the Backend docker container

### Create a dockerfile for the backend 
Create a Dockerfile in the backend-flask folder

```
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

```

### Build the backend container
First, Lets unset the environment variables 
```
unset FRONTEND
unset BACKEND
```
We can verify the existence of the variables with the grep command
``` 
env | grep FRONTEND_URL
env | grep BACKEND_URL
````
Now return to the home directory and run the command below
```
docker build -t  backend-flask ./backend-flask
```

### Run backend docker container
```
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```
*Get to the port and unlock port 4567 and click the link and the append /api/activities/home to the url in the browser*

## Building the Frontend docker container

### Create a dockerfile for the Frontend
Change directory to frontend-js and install npm since the container needs to copy the contents of node_modules

![npm_install](assets/npm.png)

Create a Dockerfile in the frontend-react-js folder

```
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

### Build the frontend container
```
docker build -t frontend-react-js ./frontend-react-js
```


### Run frontend docker container
```
docker run -p 3000:3000 -d frontend-react-js
```
## Running and managing multiple containers

Docker Compose is used to create manage and cleanup multi-container applications by reading and applying rules defined in a docker compose file.

Create the docker compose file under the home directory

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

To run the docker compose file right click on the docker compose file and select compose up.

