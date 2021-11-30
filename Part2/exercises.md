# Migrating to docker-compose.yml

## 2.1

````
#docker-compose.yml

version: '3.5'

services:

  simple-web-service-test:
    image: devopsdockeruh/simple-web-service
    build: .
    volumes:
      - ./text.log:/usr/src/app/text.log
    container_name: simple-web-service-test
````

````
touch text.log
docker-compose up
````

![e1a](https://i.imgur.com/5EEuK3Y.png)
![e1b](https://i.imgur.com/XGF71lj.png)

## 2.2

````
#docker-compose.yml

version: '3.5'

services:

  simple-web-service-test:
    image: devopsdockeruh/simple-web-service
    build: .
    ports: 
      - 3000:8080
    command: server
    container_name: simple-web-service-test
````

````
docker-compose up
````

![e2](https://i.imgur.com/8o91v95.png)

## 2.3

Using old images (frontendtest/backendtest) created in [Part1](https://github.com/realclever/devops-with-docker2021/blob/main/Part1/exercises.md)

````
#docker-compose.yml

version: '3.5'

services:

  frontendtest:
    image: frontendtest
    environment:
      - REACT_APP_BACKEND_URL=http://localhost:8080
    build: ./example-frontend
    ports: 
      - 5000:5000
    command: serve -s -l 5000 build

  backendtest:
    image: backendtest
    environment:
      - REQUEST_ORIGIN=http://localhost:5000
    build: ./example-backend
    ports:
      - 8080:8080
    command: ./server  
````

````
docker-compose up
````

Both http://localhost:5000 and http://localhost:8080/ping work. 

# Docker networking

## 2.4

````
docker pull redis
````

````
#docker-compose.yml

version: '3.5'

services:

  frontendtest:
    image: frontendtest
    environment:
      - REACT_APP_BACKEND_URL=http://localhost:8080
    build: ./example-frontend
    ports: 
      - 5000:5000
    command: serve -s -l 5000 build

  backendtest:
    image: backendtest
    environment:
      - REQUEST_ORIGIN=http://localhost:5000
      - REDIS_HOST=redis
    build: ./example-backend
    ports:
      - 8080:8080
    command: ./server

  redis:
    image: redis
    ports:
      - 6379:6379 
````

````
docker-compose up
````

Both http://localhost:5000 and http://localhost:8080/ping work.

![e4a](https://i.imgur.com/KST4n2S.png)

````
curl http://localhost:8080/ping?redis=true
curl http://localhost:8080/ping?redis=false
````

It's easy to tell the difference.

![e4b](https://i.imgur.com/bPQGNzY.png)

# Scaling

## 2.5

Clone the project. 

Navigate to the scaling-exercise folder and test it with ````docker-compose up```` *not working*

Looks like three instances is enough to get the job done. 

````
docker-compose up -d --scale compute=3
````

![e5](https://i.imgur.com/lxoRPn8.png)

# Larger application with volumes

## 2.6

Let's continue configuring exercise 2.4

Install Postgres ````docker pull postgres````

````
#docker-compose.yml

version: '3.5'

services:

  frontendtest:
    image: frontendtest
    environment:
      - REACT_APP_BACKEND_URL=http://localhost:8080
    build: ./example-frontend
    ports:
      - 5000:5000
    command: serve -s -l 5000 build

  backendtest:
    image: backendtest
    environment:
      - REQUEST_ORIGIN=http://localhost:5000
      - REDIS_HOST=redis
      - POSTGRES_PASSWORD=example
      - POSTGRES_HOST=db
    build: ./example-backend
    ports:
      - 8080:8080
    command: ./server

  redis:
    image: redis
    ports:
      - 6379:6379

  db:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      - POSTGRES_PASSWORD=example
````

````
docker-compose up
^C terminates
````
![e6](https://i.imgur.com/VgykBD5.png)

## 2.7

Clone all three parts. 

````
#docker-compose.yml

version: '3.5'

services:

  ml-frontendtest:
    image: ml-frontendtest
    build: ./ml-kurkkumopo-frontend
    ports:
      - 3000:3000

  ml-backendtest:
    image: ml-backendtest
    build: ./ml-kurkkumopo-backend
    ports:
      - 5000:5000
    volumes:
      - ./model:/src/model

  ml-trainingtest:
    image: ml-trainingtest
    build: ./ml-kurkkumopo-training
    volumes:
      - ./model:/src/model
      - ./imgs:/src/imgs

volumes:
  model: null
````

````
docker-compose up
^C terminates
````
Ran into several problems while configuring this project, but looks like it "works". 

![e7](https://i.imgur.com/ZCFCShb.png)

## 2.8

Let's continue configuring exercise 2.6

Pull Nginx ````docker pull nginx ```` 

````
#docker-compose.yml

version: '3.5'

services:

  frontendtest:
    image: frontendtest
    environment:
      - REACT_APP_BACKEND_URL=http://localhost:8080
    build: ./example-frontend
    ports:
      - 5000:5000
    command: serve -s -l 5000 build

  backendtest:
    image: backendtest
    environment:
      - REQUEST_ORIGIN=http://localhost:5000
      - REDIS_HOST=redis
      - POSTGRES_PASSWORD=example
      - POSTGRES_HOST=db
    build: ./example-backend
    ports:
      - 8080:8080
    command: ./server

  redis:
    image: redis
    ports:
      - 6379:6379

  db:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      - POSTGRES_PASSWORD=example

  web:
    image: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - 80:80
````
````
#nginx.conf

events { worker_connections 1024; }

http {
  server {
    listen 80;

    location / {
      proxy_pass http://frontendtest:5000; 
    }

    location /api/ {
      proxy_set_header Host $host;
      proxy_pass http://backendtest:8080/;
    }
  }
}
````

````
docker-compose up
^C terminates
````

![e8](https://i.imgur.com/CDAH8ny.png)

## 2.9

Let's continue configuring exercise 2.8

According to Postgres image documentation the default location for database files is ```/var/lib/postgresql/data```. Just need to add volumes to db. 

````
#docker-compose.yml

version: '3.5'

services:

  frontendtest:
    image: frontendtest
    environment:
      - REACT_APP_BACKEND_URL=http://localhost:8080
    build: ./example-frontend
    ports:
      - 5000:5000
    command: serve -s -l 5000 build

  backendtest:
    image: backendtest
    environment:
      - REQUEST_ORIGIN=http://localhost:5000
      - REDIS_HOST=redis
      - POSTGRES_PASSWORD=example
      - POSTGRES_HOST=db
    build: ./example-backend
    ports:
      - 8080:8080
    command: ./server

  redis:
    image: redis
    ports:
      - 6379:6379

  db:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      - POSTGRES_PASSWORD=example
    volumes:
      - ./dbfolder:/var/lib/postgresql/data

  web:
    image: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - 80:80
````

```docker-compose up``` - 
dbfolder is now created.   
```docker-compose down``` - 
dbfolder still exists.   
```docker-compose up``` -  after refreshing the messages still exist.   
```docker-compose down``` -  and delete the dbfolder.  
```docker-compose up``` -  new dbfolder has been created, but the old messages are gone.  
      
      
