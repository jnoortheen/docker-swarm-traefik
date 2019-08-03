# Docker Swarm

  Docker swarm with one node is used for deploying multiple services. Initialize docker swarm mode by

```sh
docker swarm init
```

For More info check Docker's getting started [guide](https://docs.docker.com/get-started/part3/)

## setup

create a network that is used by traefik and other web services for reverse-proxying

```sh
docker network create proxy
```

clone this repo and fill up the environment variables

```sh
cp .env.template .env
```

to run the traefik

```sh
docker stack deploy -c docker-compose.yml traefik
```

then you will be able to access the web UI of traefik at `traefik.$HOST_NAME`

to view the logs of the service

```sh
docker service logs traefik_traefik -f
```

## How to configure a web service to use traefik

note the labels section and it should define the same network as traefik

```yml

# docker-compose.yml
  # test service
services:
  whoami:
    image: emilevauge/whoami
    networks:
      - proxy
    deploy:
      labels:
        traefik.enable: "true"
        traefik.frontend.rule: 'Host: who.dvlab.in'
        traefik.port: 80
        traefik.docker.network: 'proxy'

networks:
  proxy:
    external: true

```

## ToDos

- create basic auth for traefic web ui
- run gitlab part of the docker swarm
