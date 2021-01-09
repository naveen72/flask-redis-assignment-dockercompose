Flask Webapp - Redis storage deployment by Docker swarm

Pre-requisites:
Docker and Docker compose should be installed on one linux server where this deployment needs to be done.
Ingress on port 22(SSH ACCESS), 5000(To access Flask Webapp) should be allowed in this linux server.

Setup:Setup
=================

Create one directory on linux server for this activity. 
ubuntu@ip-172-31-20-149:$ mkdir docker-compose-assignment 
ubuntu@ip-172-31-20-149:$ cd docker-compose-assignment/

Create a Flash webapp python file app.py in directory(docker-compose-assignment) and paste below code: 

from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello Naveen! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)

In this code, redis is the hostname of the redis container on the application’s network. We use the default port for Redis, 6379.

Step: 2 Create a Dockerfile
===========================

Here we have written a Dockerfile that builds a Docker image. The image contains all the dependencies the Python application requires, including Python itself. 
In the current directory, create a file named Dockerfile and paste the following:

FROM python:3.10.0a4-alpine
ADD . /code
WORKDIR /code
RUN pip install flask redis
CMD ["python", "flask_app.py"]

Building an image starting with the Python 3.10 image. Add the current directory . into the path /code in the image. Set the working directory to /code. 
Install the Python dependencies. Set the default command for the container to python flask_app.py.

Step 3: Define services in a Compose file
=========================================

Create a file called docker-compose.yml in your project directory and paste the following:

version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"


This Compose file defines two services, web and redis. 
The web service uses an image that’s built from the Dockerfile in the current directory.
Forwards the exposed port 5000 on the container to port 5000 on the host machine. 
We use the default port for the Flask web server, 5000.
The redis service uses a public Redis image pulled from the Docker Hub registry.

Step 4: Build and run your app with Compose
===========================================

1. From your project directory, start up your application by running docker-compose up.

ubuntu@ip-172-31-20-149:~/test$ docker-compose up
Creating network "test_default" with the default driver
Building web
Step 1/5 : FROM python:3.10.0a4-alpine
 ---> cff9819fdc84
Step 2/5 : ADD . /code
 ---> 6da28bbecdf8
Step 3/5 : WORKDIR /code
 ---> Running in 94165fa39628
Removing intermediate container 94165fa39628
 ---> 1084c5ac337e
Step 4/5 : RUN pip install flask redis
 ---> Running in fde88cae826f
Collecting flask
  Downloading Flask-1.1.2-py2.py3-none-any.whl (94 kB)
Collecting click>=5.1
  Downloading click-7.1.2-py2.py3-none-any.whl (82 kB)
Collecting itsdangerous>=0.24
  Downloading itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting Jinja2>=2.10.1
  Downloading Jinja2-2.11.2-py2.py3-none-any.whl (125 kB)
Collecting MarkupSafe>=0.23
  Downloading MarkupSafe-1.1.1.tar.gz (19 kB)
Collecting Werkzeug>=0.15
  Downloading Werkzeug-1.0.1-py2.py3-none-any.whl (298 kB)
Collecting redis
  Downloading redis-3.5.3-py2.py3-none-any.whl (72 kB)
Building wheels for collected packages: MarkupSafe
  Building wheel for MarkupSafe (setup.py): started
  Building wheel for MarkupSafe (setup.py): finished with status 'done'
  Created wheel for MarkupSafe: filename=MarkupSafe-1.1.1-py3-none-any.whl size=12629 sha256=4fe8807e62e7b074b98bf8ad2d4aad17245998ebf321b3710e92b652eb03ff0a
  Stored in directory: /root/.cache/pip/wheels/a6/81/81/3fcafa7c24e4b4e25bcf383c792b343e53c38e6196f44bc3e3
Successfully built MarkupSafe
Installing collected packages: MarkupSafe, Werkzeug, Jinja2, itsdangerous, click, redis, flask
Successfully installed Jinja2-2.11.2 MarkupSafe-1.1.1 Werkzeug-1.0.1 click-7.1.2 flask-1.1.2 itsdangerous-1.1.0 redis-3.5.3
Removing intermediate container fde88cae826f
 ---> 34ff6b33aa1d
Step 5/5 : CMD ["python", "flask_app.py"]
 ---> Running in 850bf74aa098
Removing intermediate container 850bf74aa098
 ---> 213c342e872e
Successfully built 213c342e872e
Successfully tagged test_web:latest
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating test_web_1 ...
Creating test_redis_1 ...
Creating test_web_1
Creating test_web_1 ... done
Attaching to test_redis_1, test_web_1
redis_1  | 1:C 09 Jan 2021 18:07:17.772 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 09 Jan 2021 18:07:17.772 # Redis version=6.0.9, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 09 Jan 2021 18:07:17.772 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1  | 1:M 09 Jan 2021 18:07:17.777 * Running mode=standalone, port=6379.
redis_1  | 1:M 09 Jan 2021 18:07:17.777 # Server initialized
redis_1  | 1:M 09 Jan 2021 18:07:17.777 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
redis_1  | 1:M 09 Jan 2021 18:07:17.778 * Ready to accept connections
web_1    |  * Serving Flask app "flask_app" (lazy loading)
web_1    |  * Environment: production
web_1    |    WARNING: This is a development server. Do not use it in a production deployment.
web_1    |    Use a production WSGI server instead.
web_1    |  * Debug mode: on
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1    |  * Restarting with stat
web_1    |  * Debugger is active!
web_1    |  * Debugger PIN: 280-211-975

Compose pulls a Redis image, builds an image for your code, and start the services you defined. 
In this case, the code is statically copied into the image at build time.

2. Enter http://<ip_of_server>:5000/ in a browser to see the application running.

You should see a message in your browser saying:

Hello NClouds! I have been seen 1 times. 

3. Refresh the page.

The number should increment.

Hello NClouds! I have been seen 2 times. 

4. Switch to another terminal window, and type docker image ls to list local images.

ubuntu@ip-172-31-20-149:~/test$ docker ps -a
CONTAINER ID        IMAGE                         COMMAND                  CREATED             STATUS                   PORTS                    NAMES
ce4362dc6ed2        test_web                      "python flask_app.py"    5 minutes ago       Up 5 minutes             0.0.0.0:5000->5000/tcp   test_web_1
fe41724be078        redis:alpine                  "docker-entrypoint.s…"   5 minutes ago       Up 5 minutes             6379/tcp                 test_redis_1
fb4ded28011a        dockercomposeassignment_web   "python flask_app.py"    4 hours ago         Exited (0) 4 hours ago                            dockercomposeassignment_web_1
399e1886c6df        redis:alpine                  "docker-entrypoint.s…"   4 hours ago         Exited (0) 4 hours ago                            dockercomposeassignment_redis_1
ubuntu@ip-172-31-20-149:~/test$ ^C

At this point, you have seen the basics of how Compose works.
