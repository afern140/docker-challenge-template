# Setting up a Docker Server for Dummies

In this guide for dummies I (Angelo Fernando) will be explaining a foolproof way to setup your own Docker server. 
I'll be explaining how to setup 2 types of webpages, one written in HTML, and one in NodeJS.  
This guide will assume that you are using a Windows PC. If you are using Linux, this guide is not for you.

## Prerequisites

- You must have Git installed

- You must have a GitHub account

- You must have Docker installed (this will also be explained, however)

- You must be technologically literate enough to do Chapter 0.1 & 0.2

## Chapter 0.1 - Installing Docker

### Step 1

- Download Docker from here: https://www.docker.com/products/docker-desktop/  
(Make sure to click **Download for Windows**)

### Step 2

- Keep clicking next to install Docker with default settings

- Restart your computer when prompted

### Step 3

- After restarting, accept the Docker User Agreement

## Chapter 0.2 - Setting up the Development Environment

### Step 1

- Fork this repository: https://github.com/eduluz1976/docker-challenge-template

### Step 2

- Use your preferred method to clone the repository to your pc (this will not be explained, use google)

### Step 3

- Open the repository in your preferred editor (this guide will assume VS Code)

## Chapter 1 - Setting up an HTML Server

### Step 1

- Open the `challenge1` folder

- Create a new folder called `public`

### Step 2

- Create an `index.html` file inside the folder

- Open the file and add your name and student id (specific tags don't matter)

### Step 3

- Create a `Dockerfile` (no extension) in the `challenge1` folder

- Place the following code in the file  
```Dockerfile
FROM nginx:latest
COPY /public /usr/share/nginx/html
EXPOSE 8080
```

### Step 4

- Open CMD and `cd` to the `challenge1` folder

- Build the Docker Image using the command `docker build -t dockerimage .`

### Step 5

- Using the same terminal from before, run `docker run -d -p 8080:80 dockerimage` to start the Docker server

- Open your browser and navigate to http://localhost:8080 to view the website

- The result should be something like this  
![result](img/challenge1.png)

### Step 6

- Push your changes to your GitHub repository fork

## Chapter 2 - Setting up a NodeJS Server

### Step 1

- Download the `challenge2.zip` file

- Open the `challenge2` folder

- Extract `challenge2.zip` to the `challenge2` folder (do not create a new folder when extracting)

### Step 2

- Create a `Dockerfile` (no extension) in the `challenge2` folder

- Place the following code in the folder
```Dockerfile
FROM node:latest
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 8080
CMD ["npm", "start"]
```

### Step 3

- Create a `docker-compose.yml` in the `challenge2` folder

- Place the following code in the folder
```yml
version: '3'
services:
    nginx:
        image: nginx:latest
        ports:
            - "8080:80"
        volumes:
            - ./nginx.conf:/etc/nginx/nginx.conf
        depends_on:
            - nodejs
    nodejs:
        build: .
        ports:
            - "3000:3000"
        volumes:
            - .:/app
```

### Step 4

- Create a `nginx.conf` in the `challenge2` folder

- Place the following code in the folder
```nginx
events {}

http {
	server {
		listen 80;

		location / {
			proxy_pass http://nodejs:3000;
			proxy_http_version 1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection 'upgrade';
			proxy_set_header Host $host;
			proxy_cache_bypass $http_upgrade;
		}
	}
}
```

### Step 5

- Open CMD and `cd` to the `challenge2` folder

- Build the Docker image using the command `docker-compose up --build`

- Open your browser and navigate to http://localhost:8080/api/books

- The result should be something like this  
![result](img/challenge2.png)

### Step 6

- Push your changes to your GitHub repository fork

## Chapter 3 - Setting up a MariaDB Database

### Step 0.1

- Download `challenge3.zip`

- Download MariaDB from here: https://mariadb.org/download/  
(Make sure to click **Download for Windows**)

- Install MariDB

### Step 0.2

- Create a new user using the command `CREATE USER 'docker'@'localhost' IDENTIFIED BY 'dockerpassword';`

### Step 0.3

- Create a new database using the command `CREATE OR REPLACE DATABASE dockerchallenge;`

- Run the provided `init.sql` file to create and populate tables

### Step 1

- Create a new file called `.env` in the `challenge3` folder

- Add the following fields to the file
```sh
DB_ROOT_PASSWORD=password
DB_DATABASE=dockerchallenge
DB_USERNAME=docker
DB_PASSWORD=dockerpassword
DB_HOST=db

MYSQL_ROOT_PASSWORD=password
MYSQL_DATABASE=dockerchallenge
MYSQL_USER=docker
MYSQL_PASSWORD=dockerpassword
MYSQL_HOST=db
```

- NOTE - If you used a different database name or a different name for any other fields, don't just copy paste this and actually update it with the names you used

### Step 2

- Create a new file called `docker-compose.yml` in the `challenge3` folder

- Add the following code to the file
```yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "8080:80"
    depends_on:
      - node-service
  node-service:
    build: ./docker/api
    environment:
      - DB_HOST=${DB_HOST}
      - DB_USERNAME=${DB_USERNAME}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_DATABASE=${DB_DATABASE}
    depends_on:
      - db
  db:
    image: mariadb
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    volumes:
      - ./docker/db/init/init.sql:/docker-entrypoint-initdb.d/init.sql
volumes:
  db-data:
```

- This will pass all the environment variables to the api, and create a server at http://localhost:8080

### Step 3

- Open CMD and `cd` to the `challenge3` folder

- Build the server using the command `docker-compose up --build`

- Navigate to http://localhost:8080/api/books

- The result should be something like this  
![result](img/challenge3.png)

- Navigate to http://localhost:8080/api/books/1

- You should see an individual book



## Chapter 4 - Scaling the Application

### Step 1

- Duplicate the `challenge3` folder, and name it `challenge4`

### Step 2

- Open CMD and `cd` to the `challenge4` folder

- Build the server using the command `docker-compose up --build --scale node-service=3`

### Step 3

- Open a new CMD window and `cd` to the `challenge4` folder

- Run the command `docker-compose ps`

- The result should be similar to this  
![result](img/challenge4.png)

### Step 4

- Push your changes to your GitHub repository fork