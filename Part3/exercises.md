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

RUN node -v && npm -v

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

RUN go test ./...

RUN useradd -m appuser

RUN chown appuser: /usr/src/app

USER appuser

CMD ["./server"]
````

Built and ran both Dockerfiles. 
