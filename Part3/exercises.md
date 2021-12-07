# Deployment pipeline with docker-compose

## 3.1 A deployment pipeline to heroku

This exercise was quite a handful, but seems like everything works. 

https://github.com/realclever/devops-with-anecdotes

Kudos to: https://github.com/marketplace/actions/build-push-and-release-a-docker-container-to-heroku

## 3.3 

Using Dockerfiles from exercises 1.13/1.14 with minor modifications.

Frontend
````
# Dockerfile front

FROM node:14

EXPOSE 5000
 
WORKDIR /usr/src/app

COPY . .

ENV REACT_APP_BACKEND_URL=http://localhost:8080

RUN npm install

RUN npm run build

RUN npm install -g serve

RUN useradd -m appuser

RUN chown appuser: /usr/src/app

USER appuser

CMD ["serve", "-s", "-l", "5000", "build"]
````

Backend
````
# Dockerfile back

FROM golang:1.16

EXPOSE 8080

WORKDIR /usr/src/app

COPY . .

RUN go build

ENV REQUEST_ORIGIN=http://localhost:5000

RUN useradd -m appuser

RUN chown appuser: /usr/src/app

USER appuser

CMD ["./server"]
````

Built and ran both Dockerfiles. 


## 3.4

Used the Dockerfiles from previous exercise.

Frontend [optimized] 
````
FROM node:14

EXPOSE 5000
 
WORKDIR /usr/src/app

COPY . .

ENV REACT_APP_BACKEND_URL=http://localhost:8080

RUN npm install && \ 
    npm run build && \
    npm install -g serve && \
    useradd -m appuser && \
    chown appuser: /usr/src/app && \
    rm -rf /var/lib/apt/lists/*

USER appuser

CMD ["serve", "-s", "-l", "5000", "build"]
````

Backend [optimized]
````
FROM golang:1.16

EXPOSE 8080

WORKDIR /usr/src/app

COPY . .

ENV REQUEST_ORIGIN=http://localhost:5000

RUN go build && \
    useradd -m appuser && \
    chown appuser: /usr/src/app && \
    rm -rf /var/lib/apt/lists/* 

USER appuser

CMD ["./server"]
````

```docker image inspect [image] --format='{{.Size}}'```


Before/After (bytes)

| Frontend | Backend |
| ------------- | ------------- |
| 1171007075  | 1065539902  |
| 1171001685  | 1065302358  |

Very minor changes. 

## 3.5

Using the Dockerfiles with Alpine variants. 

Frontend [optimized] 
````
FROM alpine

EXPOSE 5000
 
WORKDIR /usr/src/app

COPY . .

ENV REACT_APP_BACKEND_URL=http://localhost:8080

RUN apk add --no-cache nodejs npm && \
    npm install && \ 
    npm run build && \
    npm install -g serve && \
    adduser -D appuser && \
    chown -R appuser:appuser /usr/src/app && \
    rm -rf /var/lib/apt/lists/* 

USER appuser

CMD ["serve", "-s", "-l", "5000", "build"]
````

Backend [optimized] 
````
FROM golang:alpine

EXPOSE 8080

WORKDIR /usr/src/app

COPY . .

ENV REQUEST_ORIGIN=http://localhost:5000

RUN go build && \
    adduser -D appuser && \
    chown appuser: /usr/src/app && \
    rm -rf /var/lib/apt/lists/* 

USER appuser

CMD ["./server"]
````
Built and ran both Dockerfiles. 

Before/After (bytes)

| Frontend | Backend |
| ------------- | ------------- |
| 1171001685  | 1065302358  |
|  351009337 |  466868037 |

According to [this article](https://itnext.io/lightweight-and-performance-dockerfile-for-node-js-ec9eed3c5aef) nodejs variant alpine is only 5mb compared to other 100mb+ variants. <strike> Unfortunately ```chown``` stopped working on frontend and requires configuring the package manager in order to work</strike>.  Now we are noticing significant changes to image sizes. 
EDIT: ```chown``` fixed

## 3.6 Multi-stage frontend

````
FROM alpine as build-stage

EXPOSE 5000
 
WORKDIR /usr/src/app

COPY . .

ENV REACT_APP_BACKEND_URL=http://localhost:8080

RUN apk add --no-cache nodejs npm && \
    npm install && \ 
    npm run build && \
    adduser -D appuser && \
    chown -R appuser: /usr/src/app && \
    rm -rf /var/lib/apt/lists/* 

USER appuser

FROM node:alpine

COPY --from=build-stage /usr/src/app/build build/

RUN npm install -g serve

CMD ["serve", "-s", "-l", "5000", "build"]
````

Before/After (bytes)

| Frontend |
| ------------- |
| 351009337  |
|  180149466 |

## 3.6 Multi-stage backend

````
FROM golang:alpine as build-stage 

EXPOSE 8080

WORKDIR /usr/src/app

COPY . .

ENV REQUEST_ORIGIN=http://localhost:5000

RUN CGO_ENABLED=0 GOOS=linux go build . && \
    adduser -D appuser && \
    chown -R appuser: /usr/src/app && \
    rm -rf /var/lib/apt/lists/* 

USER appuser

FROM scratch 

COPY --from=build-stage /usr/src/app .

CMD ["./server"]
````
Before/After (bytes)

| Backend |
| ------------- |
| 466868037  |
|  17647316 [17.65MB] |

## 3.7

Before

````
FROM node:14

EXPOSE 3000
 
WORKDIR /usr/src/app

COPY . .

RUN npm install

RUN npm run build

CMD [ "npm", "start" ]
````

After

