Flask Webapp - Redis storage deployment by Docker swarm


Pre-requisites:
===============
1. Docker and Docker compose should be installed on one linux server where this deployment needs to be done.
2. Ingress on port 22(SSH ACCESS), 5000(To access Flask Webapp) should be allowed in this linux server.

Setup:Setup
==============
1. Create one directory on linux server for this activity.
ubuntu@ip-172-31-20-149:~$ mkdir docker-compose-assignment
ubuntu@ip-172-31-20-149:~$ cd docker-compose-assignment/

2. Create a Flash webapp python file app.py in directory(docker-compose-assignment) and paste below code:
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello NClouds Team! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
    
In this code, redis is the hostname of the redis container on the application’s network.
We use the default port for Redis, 6379.

Step: 2 Create a Dockerfile
===========================

Here we have written a Dockerfile that builds a Docker image. The image contains all the dependencies the Python application requires, including Python itself.
In the current directory, create a file named Dockerfile and paste the following:

FROM python:3.10.0a4-alpine
ADD . /code
WORKDIR /code
RUN pip install flask redis
CMD ["python", "flask_app.py"]

Building an image starting with the Python 3.10 image. Add the current directory . into the path /code in the image.
Set the working directory to /code. Install the Python dependencies. Set the default command for the container to python flask_app.py.
 
