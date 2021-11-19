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
