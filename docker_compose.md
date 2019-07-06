# Docker Compose

Learning docker compose is very useful both for including the compose
methodology to our software development lifecycle and also to learn docker swarm
which allows us to use docker on a cluster of machines.


Docker Compose has two components:
* a docker-compose configuration file
* a docker-compose command line command

If we have a docker-compose.yml file in our project root directory or wherever
we want to run our docker-compose commands we do not have to specify the
configuration file,


We can launch docker-compose by doing:
```sh
docker-compose up -d 
# we can launch all the docker architecture with `up`
# and with `-d` everything will be in background daemonized
```

We can stop or restart the infrastructure by doing:
```sh
docker-compose stop
```

or to restart:
```sh
docker-compose restart
```


In order to inspect the output we can do:
```sh
docker-compose logs
```
we can also follow logs by doing:
```sh
docker-compose logs -f
```

we can also only follow the output of a specific service by appending the name
of that service, for example:
```sh
docker-compose logs <service_name>
```


We can inspect networks by doing:
```sh
docker network ls
```

To inspect a specific network we can do:
```sh
docker network inspect <network_name>
```

We cann check the running processes in our compose directory by doing:
```sh
docker-compose ps
```

We can also execute commands inside specific docker services by doing:
```sh
docker-compose exec <service_name> <command> 
```
for example to get a shell inside a container we can do:
```sh
docker-compose exec <servicename> /bin/bash
# here as shell we may also have /bin/ash if we have an alpine container
```

## Launching replicas of a service

If we don't have specific port mappings in our docker compose file, we can also
launch more infrastructures by doing:
```sh
docker-compose up --scale <service_name>=3
# this compose will launch 3 instances of the mentioned service
```

