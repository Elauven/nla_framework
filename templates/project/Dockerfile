FROM golang:1.20-alpine AS builder_go

WORKDIR /tmp/server
COPY ./src/go.mod .
RUN go mod download
COPY ./src .
RUN CGO_ENABLED=0 go build -o ./build/app

FROM node:14.21.3-alpine3.17 AS builder_node

WORKDIR /tmp/webClient
# RUN npm install --global yarn
COPY ./src/webClient .
RUN yarn install
# RUN export NODE_ENV='development'
# RUN echo $NODE_ENV
RUN NODE_ENV=development npx quasar build

FROM alpine
# Update package index
RUN apk add --no-cache tzdata
ENV TZ=Europe/Moscow
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apk update && apk add ca-certificates && apk add --update curl && apk add zip && rm -rf /var/cache/apk/*

COPY --from=builder_go /tmp/server/build/app /app
COPY ./src/config.toml /config.toml
COPY ./src/sql /sql
COPY --from=builder_node /tmp/webClient/dist /webClient/dist

RUN chmod -Rf 777 /app
ENTRYPOINT ["/app"]
