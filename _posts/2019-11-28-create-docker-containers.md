---
title:  "How to create docker containers"
search: true
toc: true
categories: 
  - Jekyll
last_modified_at: 2018-02-19T08:06:00-05:00
---

Create a Dockerfile in your project 

```dockerfile
# Use an official Python runtime as a parent image
FROM python:2.7-slim
# Set the working directory to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
COPY . /app
# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt
# Make port 80 available to the world outside this container
EXPOSE 80
# Define environment variable
ENV NAME World
# Run app.py when the container launches
CMD ["python", "app.py"]
```

This uses python as base image
changes working directory 
copies the contents to the working directory
the installs the requires libraries - executes the app.py

```bash
docker build --tag=repository_name:tag .
docker build --tag=fisrtdocker:v0.0.1 .
docker build --tag=firstdocker:latest .
```

### Listing the docker images existing in the system
```bash
docker image ls
```

### Viewing the docker containers in the system
```bash
docker container ls
```

### Running the docker app/container
```bash
docker run -p 4000:80 friendlyhello
# 4000 is the computer port - 80 is the port in the docker container that is open to the outside world
```

### Running the docker container
```bash
docker run -p 4000:80 friendlyhello:v0.0.2
docker run -p 4000:80 friendlyhello:latest

# running in detached mode
docker run -d -p 4000:80 friendlyhello:latest
```

### Viewing the up and running docker containers and stopping a docker container is needed
```bash
docker container ls // gives the list of running docker containers
# get the container ID of the docker image you want to stop and then
docker container stop 08a1f8665751 //stops the container
```

### Sending the docker image to the docker hub
```bash
docker login
docker tag image username/repository:tag
# ex - docker tag friendlyhello adhithad/getting-started:part1

docker image ls
docker push username/repository:tag
```

### Pull and run the image from the remote repository
```bash
docker run -p 4000:80 username/repository:tag
# example
docker run -p 4000:80 adhithad/get-started:part2
```

### Logging into a docker container
```bash
docker exec -it ab02046a4a19 sh // ab02046a4a19 is the docker container ID
docker exec -it sdasasdasfas bash
```

