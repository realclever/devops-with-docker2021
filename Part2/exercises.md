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


