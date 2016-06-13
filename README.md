# docker-lifecycle-script-generator
Docker container lifecycle script generator

example:
 ```
$ ./generate-docker-lifecycle-script.py
please input the container name: docker.io/my-awesome-container
please input any docker run args: -d --ulimit nofile=65535:65535 -p 8080:8080 -p 9090:9090 -v `pwd`:/tmp
Your new docker lifecycle script is at: my-awesome-container.sh
```

output file (my-awesome-container.sh):
```bash
#!/usr/bin/env bash

current_container=$(docker ps | grep docker.io/my-awesome-container  | awk '{print $1}')
operation=$1
shift

NO_CONTAINER="There is no docker.io/my-awesome-container container running"
CONTAINER_RUNNING="A(n) docker.io/my-awesome-container container is running with id: $current_container"

start_container(){
  echo -n "Started docker container: "
  docker run -d --ulimit nofile=65535:65535 -p 8080:8080 -p 9090:9090 -v `pwd`:/tmp docker.io/my-awesome-container "$@"
}

stop_container(){
  echo -n "Stopped docker container: "
  docker stop $current_container
}

remove_container(){
  echo -n "Removing docker container: "
  docker rm $current_container
}

case "$operation" in
  start)
    if [[ -z $current_container  ]]; then
      start_container
    else
      echo $CONTAINER_RUNNING
    fi
    ;;
  stop)
    if [[ -z $current_container  ]]; then
      echo $NO_CONTAINER
    else
        stop_container
    fi
    ;;
  status)
    if [[ -z $current_container  ]]; then
      echo echo $NO_CONTAINER
    else
      echo $CONTAINER_RUNNING
    fi
    ;;
  restart)
    if [[ -z $current_container  ]]; then
      start_container
    else
      stop_container
      remove_container
      start_container
    fi
    ;;
  logs)
    if [[ -z $current_container ]]; then
      echo $NO_CONTAINER
    else
      docker logs --tail=10 -f $current_container
    fi
    ;;
  *)
    echo $"Usage: $0 {start|stop|status|restart|logs} <args to pass into container>"
    exit 1
    ;;
esac

```
