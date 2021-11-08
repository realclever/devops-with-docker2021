## 1.1: Getting started

![e1](https://i.imgur.com/hfpxRAL.png)

## 1.2: Cleanup

![e2](https://i.imgur.com/06gTeDB.png)

## 1.3: Secret message

![e3a](https://i.imgur.com/mKT75cL.png)

![e3b](https://i.imgur.com/grNdr3X.png)

## 1.4: Missing dependencies

```
docker run -d -it ubuntu sh -c 'echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;'
docker ps -a 
docker exec -it xenodochial_montalcini bash
apt-get update
apt-get install curl
```
![e4f](https://i.imgur.com/VfwQk5o.png)

## 1.5: Sizes of images

```
docker pull devopsdockeruh/simple-web-service:ubuntu
docker pull devopsdockeruh/simple-web-service:alpine
docker run -d --name alps devopsdockeruh/simple-web-service:alpine
```
![e5a](https://i.imgur.com/btIUNGm.png)
![e5b](https://i.imgur.com/T3Zb5qu.png)
