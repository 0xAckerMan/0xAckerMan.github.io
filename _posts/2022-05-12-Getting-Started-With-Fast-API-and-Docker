---
layout: post
title: "Getting Started With Fast API and Docker"
date: 2022-05-012 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: software.jpg # Add image post (optional)
---

# Getting Started With Fast API and Docker
![](https://i.imgur.com/7MCgioK.jpg)

# FASTAPI
## What is fastAPI
FastAPI is a modern, fast (high-performance), web framework for building APIs with Python 3.6+ based on standard Python type hints to validate, serialize, and deserialize data, and automatically auto-generate OpenAPI documents.

## Why fastAPI
FastAPI has is one of the latest python api's for web development. It has some of this features:
#### 1. It is indeed fast
It is fast when we compare it to other major Python frameworks like Flask and Django.
#### 2. Support for asynchronous code
This is one of exciting feature of FastAPI that it supports asynchronous code out of the box using the async/await Python keywords.
#### 3. Very short development time
It takes like 8 lines of code to comeup with a hello, word program
#### 4. Easy testing
It has a feature of test driven development with the help using the [TestClient](https://fastapi.tiangolo.com/tutorial/testing/) provided by fastAPI.
#### 5. Excellent documentation
It has one of the easiest [documentation](https://fastapi.tiangolo.com/) that one can understand.
#### 6. Easy deployment
You can easily deploy your FastAPI app via Docker using FastAPI provided [docker image](https://github.com/tiangolo/uvicorn-gunicorn-fastapi-docker)

### Requirements
```
Python 3.6+
```
### Installation of fastapi on windows.
```
pip install fastapi
```
### Installation on Mac and Linux
```
pip3 install fastapi uvicorn
```
## Example
First create a folder at your own location of choice
```
mkdir foldername && cd foldername
```
create a virtual environment inside the folder you created
```
python3 -m venv env
```
activate the virtual environment created called env
```
source env/bin/activate
```
Create a python file with the following code and save as myapp.py
```python
from fastapi import FastAPI, Depends

app = FastAPI()

@app.get('/')
async def root():
    return {'message': 'Hello World!'}
```
Then to run the server, we use the following command:
```
 uvicorn myapp:app --reload
```
To know it it works with no error, you will see this in your commandline
```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [84109] using statreload
INFO:     Started server process [84111]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```
Now to open in browser, use the following link
```
http://127.0.0.1:8000/docs
```
you will see the following output
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8thigl8xfeqj8m4ogec0.png)
before going to docker, we can create a requirement.txt file that has all the tools that you have installed in one place by running;
```
pip3 freeze > requirement.txt
```
# DOCKER
[Docker](https://www.docker.com/) is an open source platform for building ,deploying and managing containerized applications.
Docker container is a virtualized runtime environment that provides isolation capabilities for separating the execution of applications from the underpinning system
### Why docker
1. speed - docker has both execution speed,startuo speed and operational speed.
2. consistency - Build an image once and use it any where. The same image that is used to run the tests is used in production. This avoids the works in my machine problems.
3. Flexibility - Containers are portable. Well to an extent (as long as the host is running some form of linux or linux vm).
 
#### Building a docker image
Create a docker file and save with this commands
```docker
FROM python:3.9

WORKDIR /code

COPY ./requirements.txt ./requirements.txt

RUN pip install --no-cache-dir --upgrade -r /code/requirements.txt

COPY ./app 

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "80"]

```
Then build your image 
```
docker build -t myimage ./
```
