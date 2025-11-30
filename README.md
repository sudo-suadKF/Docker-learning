## Mastering Docker Basics Through Practical Projects

**A Structured, Hands-On Introduction to Containers, Networking & Multi-Service Architecture.**

This repository is designed as a foundational learning resource for anyone looking to understand Docker through simple, practical examples. 
Instead of learning Docker in theory, I built multiple containerized applications, each focusing on a core Docker concept:
- Single containers
- Containerizing various applications (Flask & Node.js)
- Redis, MySQL, NginX as services
- Docker Compose orchestration
- Scaling containers with load balancing
- Persistent storage & environment variables
- Pushing & pulling images from public and private repositories (DockerHub & AWS ECR)

**This README explains what was built, why it matters, how it works, and what others can learn from it.**

### 1. hello_flask (Flask with MySQL)

This project demonstrates how web apps interact with relational databases inside Docker.

#### What Was Built
- A simple Flask Web Application connected to a MySQL database
- A multi-stage Dockerfile
- MySQL image pulled from DockerHub
- An image for the Flask App
- Images pushed and pulled from public and private repositories
- A container network
- A docker compose file
- Docker Compose orchestrating the services

#### Why this matters & what you will learn
This project teaches:
- Connecting to MySQL database:
    - install MySQL:
        `sudo apt update`
        `sudo apt install mysql`
    - then connect to it in app file:
        ```python
        db = MySQLdb.connect(
            host="mydb",    # Hostname of the MySQL container
            user="root",    # Username to connect to MySQL
            passwd="my-secret-pw",  # Password for the MySQL user
            db="mysql"      # Name of the database to connect to
        )
        ```
- Multi-stage Dockerfile to optimize the container's deployment time by creating one stage for building the application and another stage for creating the image. Basically it separates the two stages which makes the image build lighter, the image does not require all dependencies for building an application: 
    ```docker
    #Stage 1: Build
    FROM python:3.8-slim as Build
    WORKDIR /app
    RUN apt-get update && apt-get install -y \
        gcc \
        python3-dev \
        libmariadb-dev \
        pkg-config
    COPY . .
    RUN pip install flask mysqlclient

    #Stage 2: Producion
    FROM python:3.8-slim
    WORKDIR /app
    COPY --from=Build /app /app
    EXPOSE 5002
    CMD ["python", "app.py"]
    ```
- Pushing & pulling images from DockerHub (public):
    - Create an account on DockerHub
    - Login to DockerHub through the terminal and enter your details: 
        `docker login`
    - Pushing:
        - Create a repository in DockerHub
        - Build and tag the image in the terminal: 
            `docker build -t <Dockerhub username>/<repository name>:<tag> <dockerfile's directory>`
            example from project: 
            `docker build -t sudosuad/flask-mysql:v1 .`
            The "." at the end says the dockerfile is in the current working directory.
        - Push the image: 
            `docker push sudosuad/flask-mysql:v1` 
            <img src="/images/pushed_image_dockerhub.png"></img>
    - Pulling:
        - Dockerhub is widely used because of it's accesibility. You can easily pull official, secure, up-to-date images from trusted resources, like MySQL, Mongo, Python, Node.js etc. 
        - Pull by following command: 
            `docker pull <image_name>` 
            example from project:  
            `docker pull mysql`
    - Using the images to run containers: 
        `docker run -d --name <name_container> --network <name_network> -p <port number:port number> <image_name>` 
        example: 
        `docker run -d --name my-app2 --network my-network -p 5002:5002 mysql:latest`
- Pushing & pulling images from AWS ECR (private):
    - Create an AWS account
    - Navigate to the ECR (Elastic Container Registry)
    - Create a repository
    - Pushing:
        - In the terminal, login to AWS and choose your local zone: 
        `aws login`
        - On AWS page, enter your repository and check "View push commands"
        - Follow the steps from AWS, 1-4, copy and paste every command to push the image
        - Refresh the page and the image should pop up
            <img src="/images/pushed_image_ECR.png"></img>
    - Pulling:
        - Pull the stored image by using the image's URI
        - In the repository, navigate to Images section and press the highlighted tag
        - Copy the image's URI and head to the terminal
        - Pull the image by following command: 
            `docker pull <image's URI>`
    - Using the images to run containers: 
        `docker run -d --name <name_container> --network <name_network> -p <port number:port number> <image_URI>` 
        example: 
        `docker run -d --name my-app2 --network my-network -p 5002:5002 <image_URI>`
- Creating images for applications of the dockerfile: 
    `docker build -t <add image_name> .` 
    example: 
    `docker build -t my-image`
- Creating networks to link multiple containers together:
    - Create a network with following command: 
        `docker network <name>` 
        example: 
        `docker network my-network` 
    - Run the containers linking to the created network: 
        `docker run -d --name <name_container> --network <name_network> -p <port number:port number> <image_name>` 
        example: 
        `docker run -d --name my-app1 --network my-network -p 5002:5002 my-image` 
        And run this for the other container as well. 
        `docker run -d --name my-app2 --network my-network -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql:5.7` 
- Creating docker compose file makes it much easier, you basically run one command and Docker runs all the commands above, creating network to running the containers:
    `docker compose up` 
    Docker compose file:
    ```docker
    services:
        web:
            image: my-flask-app:multistage
            ports:
                - "5002:5002"
            depends_on:
                - mydb

        mydb:
            image: mysql:5.7
            environment:
                MYSQL_ROOT_PASSWORD: my-secret-pw

#### Key Lessons
- How to connect to a MySQL database
- Understand the workflow, app -> create dockerfile ->  create or pull docker image -> create network -> build and run container with linking to network.
- Docker Compose simplifies the whole workflow, app -> create dockerfile -> create or pull docker image -> create docker-copmpose.yml -> run containers with docker compose.
- How networks allow app -> database communication with containers.
- Understanding and connecting to DockerHub/AWS ECR and pushing/pulling images

### 2. CoderCo Multi-Container Challenge

A simple multi-container setup that demonstrates persistent storage, environment variables and scaling with load balancing.

#### What was built
- A simple **Flask Web Application** with two endpoints:
    - **/** - Welcome message
        <img src="/images/welcome_message.png"></img>
    - **/count** - Persistent visit counter stored in Redis
        <img src="images/counter.png"></img>
- A **Redis database container** functioning as a key-value store
- A **Docker Compose** stack managing both services
- Persistent Redis storage using volume
- Environment variables
- Scaling of Flask application with NginX load balancing

#### Why this matters & what you will learn
This project teaches:

- How stateful services like Redis is used as a key-value store and to persist data with a volume:
    - Redis is keeping track of the visit count, everytime you refresh the page, the counter increases. This is specified by following code:
    ```python
    @app.route('/count')
    def count():
    count = r.incr('visits')
    return f'this page has been visited {count} times'
    ```
    - Redis persist its data by configuring Redis to use a volume. This saves the counter data and when you restart the container, the increment will start from the saved value of the counter. This is specified in the docker-compose file by following code: 
    inside of service, implementing volumes to Redis:
    ```docker
    redis:
        image: redis:latest
        ports:
            - '6379:6379'
        volumes:
            - redis-data:/data
    ```
    outside of service, adding volumes:
    ```docker
    volumes:
        redis-data:
            driver: local
    ```
- How to scale application when the traffic load increases:
    - In production environment, it is necessary to scale your services to ensure high availability. The services can't map to a specific port, it has to be exposed to a port. Otherwise it will lead to a port conflict if you want to connect multiple instances to same port. Expose  This is specified by following code:
    ```docker
    services:
        web:
            image: 541064517911.dkr.ecr.eu-west-2.amazonaws.com/flask-redis
            expose:
                - "5002"
            depends_on:
                - redis
            environment:
                - REDIS_HOST=redis
                - REDIS_PORT=6379
    ```
    Expose directive makes the Flask app's port 5002 available to other services within the docker network, but doesn't bind it to the host network.
    - The application is not accessible from the host machine at the moment since it is not mapped to a port, it is exposed. To make it accessible, here is where the load balancer comes in.  
    To access the application, we have to bind to a port on the load balancer, which becomes a single point of accessing the application.  
    Nginx service is used here as a load balancer and is implemented in the docker-compose file: 
    ```docker
    nginx:
        image: nginx:latest
        ports:
            - "5002:5002"
        volumes:
            - ./nginx.conf:/etc/nginx/nginx.conf
            depends_on:
                - web
    ```
    Here, the port is being mapped which makes the application accessible. It uses the latest Nginx image which will be pulled from DockerHub by Docker Compose and it is dependent on the application to run first.  
    The volume is basically mounting a custom NGINX config file, NGINX container, to configure load balancing, which means now we have to create a new file called nginx.conf: 
    ```docker
    events {}

    http {
        upstream flask_app {
            server web:5002;
        }

        server {
            listen 5002;

            location / {
                proxy_pass http://flask_app;
            }
        }
    }
    ```
    - Scaling web services with Docker Compose:  
        `docker compose up --scale <name of app service in compose file>="number of instances"` 
        example from the project:  
        `docker compose up --scale web=3`

#### Key Lessons
- Understanding the importance and how to scale an application with Docker Compose in production environment
- Understanding how to use Redis as key-value storage and persist the data with a volume
- How to build applications with nginx service as load balancer for managing high traffic

### 3. Dockerized node.js application

A clean introduction to containerizing JavaScript applications.

#### What was built
- A simple node.js server with the package.json files and express framework
- Installed Nodemon, a command-line tool for node.js
- Deployed with Docker Compose


#### Why this matters & what you will learn
- How to download and setup a simple node.js application:
    - Run the following commands in terminal to download Node.js and NPM in Ubuntu: 
        `sudo apt update` 
        `sudo apt install nodejs npm`
    - Verify the installation by checking the following commands: 
        `node -v` 
        `npm -v` 
        You should get the version as output.
    - Navigate to your project directory and intialize a new node.js project and have the package.json file setup like down below:
        - `npm init -y`
        - Setup package.json:
        ```js
        {
            "name": "node.js",
            "version": "1.0.0",
            "description": "",
            "main": "server.js",
            "scripts": {
                "dev": "nodemon server.js",
                "test": "echo \"Error: no test specified\" && exit 1"
            },
            "keywords": [],
            "author": "",
            "license": "ISC",
            "dependencies": {
                "express": "^5.1.0"
            },
            "devDependencies": {
                "nodemon": "^3.1.11"
            }
        }
        ```
        
    - Install the express framework that provides a robust set of features for web applications: 
        `npm install express`
    - Add this following simple Node.js application code:
        ```js
        const express = require('express');
        const app = express();
        const PORT = 1199;

        app.get('/', (req, res) => {
        console.log('root');
        res.send('<h1>Welcome to my node app!</h1>');
        });

        console.log('Hello World!');

        app.listen(PORT, () => console.log(`server has started on: ${PORT}`));
        ```
- Installing Nodemon, a command-line tool, for helping the development of Node.js based applications by automatically restarting the node application when file changes in the directory are detected. 
    - By following command you can download the tool: 
        `npm install --save-dev nodemon`
    - Run the app with following command to be able to use nodemon: 
        `npm run dev`
    - Test the code with changing the content of the application code down below and see the output in the terminal how the nodemon behaves:
        ```js
        console.log('Hello World!');
        ```
- Time to containerize the node.js application:
    - Create a dockerfile, it is different from flask.  
    Important to include the dependencies from package*.json files and run the right CMD with nodemon:
        ```docker
        FROM node:24-slim

        WORKDIR /app

        COPY package*.json .

        RUN npm install

        EXPOSE 1199

        COPY . .

        CMD ["npm", "run", "dev"]
        ```
    - Build the image: 
        `docker build -t <image_name>`
    - Create a docker-compose file with the image:
        ```docker
        services:
            app:
                image: <image_name:tag>
            ports:
                - '1199:1199'
        ```
    - Run the application with Docker Compose: 
        `docker compose -d --build`
    - Check if it worked by searching for:
        - **127.0.0.1:1199** 
        <img src="images/nodejs_app.png"></img>

#### Key Lessons
- Understand how to setup a simple node.js application with the dependencies and express framework.
- Understand what nodemon is and how it works.
- How to containerize a node.js application and unerstand the differences from  other applications, for example a flask application.







    

