<div  align='left'><img src='https://s3.amazonaws.com/weclouddata/images/logos/wcd_logo_new_2.png' width='15%'></div >
<p style="font-size:30px;text-align:left"><b>Docker Compose - Flask App <font size=4 color="#20czd6"><b>(Beginner Level)</b></font></b></p>
<p style="font-size:20px;text-align:left"><b><font color='#F39A54'>ML Engineering Bootcamp</font></b></p>
<p style="font-size:15px;text-align:left">Content developed by: WeCloudData Academy</p>
<br>


## Reference

> - [Adopted from the official documentation](https://docs.docker.com/compose/gettingstarted/)

## Prerequisites
Make sure you have already installed both Docker Engine and Docker Compose. You don’t need to install Python or Redis, as both are provided by Docker images.

#  1. Set Up

### 1.1 - Create a directory for the project

```bash
mkdir composetest
cd composetest
```

### 1.2 - Create application file `app.py` 

> Create a file called `app.py` in your project directory and paste this in:

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

### 1.3 - Create the `requirement.txt` file

> Create another file called requirements.txt in your project directory and paste this in:

```
flask
redis
```

<br>

# 2. Create `Dockerfile`

In this step, you write a Dockerfile that builds a Docker image. The image contains all the dependencies the Python application requires, including Python itself.

> In your project directory, create a file named Dockerfile and paste the following:

```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

This tells Docker to:

- Build an image starting with the Python 3.7 image.
- Set the working directory to `/code`.
- Set environment variables used by the flask command.
- Install gcc so Python packages such as MarkupSafe and SQLAlchemy can compile speedups.
- Copy `requirements.txt` and install the Python dependencies.
- Copy the current directory `.` in the project to the workdir `.` in the image.
- Set the default command for the container to `flask run`.

<br>
## <img src='https://s3.amazonaws.com/weclouddata/images/logos/asterisk_1.png' width='3%'>  3. Define services in a Compose file

> Create a file called `docker-compose.yml` in your project directory and paste the following:

```
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

<br>

## <img src='https://s3.amazonaws.com/weclouddata/images/logos/asterisk_1.png' width='3%'>  4. Build and run your app with Compose


### 4.1 - Build the app with docker compose
> From your project directory, start up your application by running `docker-compose up`.

```bash
docker-compose up
```

**logs**
> 
```Creating network "composetest_default" with the default driver  
Creating composetest_web_1 ...  
Creating composetest_redis_1 ...  
Creating composetest_web_1  
Creating composetest_redis_1 ... done  
Attaching to composetest_web_1, composetest_redis_1  
web_1    |  * Running on c (Press CTRL+C to quit)  
redis_1  | 1:C 17 Aug 22:11:10.480 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo  
redis_1  | 1:C 17 Aug 22:11:10.480 # Redis version=4.0.1, bits=64, commit=00000000,   modified=0, pid=1, just started  
redis_1  | 1:C 17 Aug 22:11:10.480 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf  
web_1    |  * Restarting with stat  
redis_1  | 1:M 17 Aug 22:11:10.483 * Running mode=standalone, port=6379.  
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.  
web_1    |  * Debugger is active!  
redis_1  | 1:M 17 Aug 22:11:10.483 # Server initialized  
redis_1  | 1:M 17 Aug 22:11:10.483 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.  
web_1    |  * Debugger PIN: 330-787-903  
redis_1  | 1:M 17 Aug 22:11:10.483 * Ready to accept connections  ```

- Compose pulls a Redis image, builds an image for your code, and starts the services you defined. In this case, the code is statically copied into the image at build time.

**errors**

> If you run into the following error, try to delete `~/.docker/config.json`
> `ERROR: (gcloud.auth.docker-helper) There was a problem refreshing your current auth tokens: ('invalid_grant: Bad Request', u'{\n  "error": "invalid_grant",\n  "error_description": "Bad Request"\n}')`

### 4.2 - Check the app in browser

- Enter [http://localhost:5000/](http://localhost:5000/) in a browser to see the application running.


	
<img src='https://s3.amazonaws.com/weclouddata/images/cloud/quick-hello-world-1.png' width="70%">

<br>


## <img src='https://s3.amazonaws.com/weclouddata/images/logos/asterisk_1.png' width='3%'> 5. Edit the Compose file to add a bind mount

### 5.1 - Edit the compose file

Edit `docker-compose.yml` in your project directory to add a bind mount for the web service

```
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

- The new volumes key mounts the project directory (current directory) on the host to /code inside the container, allowing you to modify the code on the fly, without having to rebuild the image. The environment key sets the FLASK_ENV environment variable, which tells flask run to run in development mode and reload the code on change. This mode should only be used in development.


## <img src='https://s3.amazonaws.com/weclouddata/images/logos/asterisk_1.png' width='3%'> 6. Re-build and run the app with Compose

### 6.1 - Stop the app 

```bash
docker-compose down
```

### 6.2 - Rebuild the app
```bash
docker-compose up
```