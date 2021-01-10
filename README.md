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
```
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
```
