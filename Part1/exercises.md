# Image and containers

## 1.1: Getting started

![e1](https://i.imgur.com/hfpxRAL.png)

## 1.2: Cleanup

![e2](https://i.imgur.com/06gTeDB.png)

# Running and stopping containers

## 1.3: Secret message

![e3a](https://i.imgur.com/mKT75cL.png)

![e3b](https://i.imgur.com/grNdr3X.png)

## 1.4: Missing dependencies

```
docker run -d -it ubuntu sh -c 'echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;'
docker ps -a 
docker exec -it xenodochial_montalcini bash
apt-get update && apt-get install -y curl
```
![e4f](https://i.imgur.com/VfwQk5o.png)

# In depth dive to images

## 1.5: Sizes of images

```
docker pull devopsdockeruh/simple-web-service:ubuntu
docker pull devopsdockeruh/simple-web-service:alpine
docker run -d --name alps devopsdockeruh/simple-web-service:alpine
```
![e5a](https://i.imgur.com/btIUNGm.png)
![e5b](https://i.imgur.com/T3Zb5qu.png)

## 1.6: Hello Docker Hub

```
docker run -it devopsdockeruh/pull_exercise
Docker Hub -> GitHub source dir -> readme.md "This is the readme, use input "basics" to complete this exercise."
```
![e6](https://i.imgur.com/yNzOop0.png)

# Building images

## 1.7: Two line Dockerfile

Create Dockerfile ($touch Dockerfile)

```
# Dockerfile
# start from the alpine image 
FROM devopsdockeruh/simple-web-service:alpine

CMD server
```
apparently Terminal/Bash requires Full Disk Access from macOS before allowing to build new images.
```
docker build . -t web-server
docker run web-server
```
![e7](https://i.imgur.com/o2KVfiN.png)

## 1.8: Image for script

Create echo.sh and Dockerfile

```
# echo.sh
echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;
```
```
# Dockerfile

# start from the ubuntu 18.04 image
FROM ubuntu:18.04

# use /usr/src/app as our workdir. The following instructions will be executed in this location.
WORKDIR /usr/src/app

# copy the echo.sh file from this location to /usr/src/app/ creating /usr/src/app/echo.sh.
COPY echo.sh .

# add execution permissions during the build and install curl
RUN chmod +x echo.sh
RUN apt-get update && apt-get install -y curl

# when running docker run the command will be ./echo.sh
CMD ./echo.sh
```

```
docker build . -t curler
```
![e8a](https://i.imgur.com/UNMWrKK.png)

```
docker run -it curler
```
![e8b](https://i.imgur.com/HCdNwJ6.png)

# More complex image/Volumes: bind mount

## 1.9: Volumes

```
touch text.log
docker run -v "$(pwd)/text.log:/usr/src/app/text.log" devopsdockeruh/simple-web-service
```

![e9b](https://i.imgur.com/QM2NxKD.png)
![e9a](https://i.imgur.com/39tgW70.png)

# Allowing external connections into containers

## 1.10: Ports open

```
docker run -p 3000:8080 web-server
```
![e10a](https://i.imgur.com/4aplzyF.png)
![e10b](https://i.imgur.com/gUWtHk8.png)

# Technology agnostic

## 1.11 Spring

Clone the spring-example project

```
# Dockerfile

# make sure you have java _8_ installed
FROM openjdk:8

# expose port 3000
EXPOSE 3000

# use /usr/src/app as our workdir. 
WORKDIR /usr/src/app

# copy the spring project from this location to /usr/src/app/
COPY /spring-example-project .

# build the project with
RUN ./mvnw package

# when running docker run the command will be 
CMD ["java", "-jar", "./target/docker-example-1.1.3.jar"]
```

```
docker build . -t springtest
docker run -p 3000:8080 springtest
```

![e11](https://i.imgur.com/osEpVtG.png)

## 1.12: Hello, frontend!

Clone the frontend-example project

```
# Dockerfile

# LTS node
FROM node:14

# expose port 5000
EXPOSE 5000

# use /usr/src/app as our workdir. 
WORKDIR /usr/src/app

# copy the frontend project from this location to /usr/src/app/
COPY /example-frontend .

# install all packages
RUN npm install

# make sure node/npm are properly installed
RUN node -v && npm -v

# build
RUN npm run build

# install serve package
RUN npm install -g serve

# run serve
CMD ["serve", "-s", "-l", "5000", "build"]
```

```
docker build . -t frontendtest
docker run -p 5000:5000 frontendtest
```
![e12](https://i.imgur.com/hoECBAb.png)

## 1.13: Hello, backend!

Clone the backend-example project

```
# Dockerfile

# golang 1.16
FROM golang:1.16

# expose port 8080
EXPOSE 8080

# use /usr/src/app as our workdir. 
WORKDIR /usr/src/app

# copy the backend project from this location to /usr/src/app/
COPY /example-backend .

# build
RUN go build

# pass url through the cors check
ENV REQUEST_ORIGIN=http://localhost:5000

# test project
RUN go test ./...

# run serve
CMD ["./server"]
```

```
docker build . -t backendtest
docker run -p 8080:8080 backendtest
```

![e13](https://i.imgur.com/dURPlLM.png)

## 1.14: Environment

No modifications to backend except changing the ENV REQUEST_ORIGIN port to 5000 (previous exercise 1.13 fix, needs a new build)

```
docker build . -t backendtest
docker run -p 8080:8080 backendtest
```

Minor frontend update: 

```
# Dockerfile

# LTS node
FROM node:14

# expose port 5000
EXPOSE 5000

# use /usr/src/app as our workdir. 
WORKDIR /usr/src/app

# copy the frontend project from this location to /usr/src/app/
COPY /example-frontend .

# so both halves of the stack can connect
ENV REACT_APP_BACKEND_URL=http://localhost:8080

# install all packages
RUN npm install

# make sure node/npm are properly installed
RUN node -v && npm -v

# build
RUN npm run build

# install serve package
RUN npm install -g serve

# run serve
CMD ["serve", "-s", "-l", "5000", "build"]
```

```
docker build . -t frontendtest
docker run -p 5000:5000 frontendtest
```

![e14](https://i.imgur.com/I4l4Ejh.png)
