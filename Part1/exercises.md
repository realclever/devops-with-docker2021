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
# Start from the alpine image 
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

# Start from the ubuntu 18.04 image
FROM ubuntu:18.04

# Use /usr/src/app as our workdir. The following instructions will be executed in this location.
WORKDIR /usr/src/app

# Copy the echo.sh file from this location to /usr/src/app/ creating /usr/src/app/echo.sh.
COPY echo.sh .

# add execution permissions during the build and install curl
RUN chmod +x echo.sh
RUN apt-get update && apt-get install -y curl

# When running docker run the command will be ./echo.sh
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
