# Docker Swarm

  Docker swarm with one node is used for deploying multiple services. Initialize docker swarm mode by

```sh
docker swarm init
```

For More info check Docker's getting started [guide](https://docs.docker.com/get-started/part3/)

## setup

clone this repo somewhere inside your server

```sh
git clone ... /var/www/server
```

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
# .env is not supported while deploying with `docker stack`
# https://github.com/moby/moby/issues/29133
docker-compose config | docker stack deploy -c - traefik

# or
docker stack deploy -c <(docker-compose config) traefik
```

then you will be able to access the web UI of traefik at `traefik.$HOST_NAME`

to view the logs of the service

```sh
docker service logs traefik_traefik -f
```

## How to configure a web service to use traefik

- The service definition should look like

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
        traefik.frontend.rule: 'Host: who.$HOST_NAME'
        traefik.port: 80
        traefik.docker.network: 'proxy'

networks:
  proxy:
    external: true
```

- Deploy it in the same way as traefik's docker compose file

```sh
docker stack deploy -c <(docker-compose config) traefik
```

Notes:

- There is no need to expose ports of these services. only need to label them in `traefik.port`
- Define the Host as you want
- If the service is already configured for SSL then `traefik.port` should be `443` depending on the server used
- Define the same network as traefik
