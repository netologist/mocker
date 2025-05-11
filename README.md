# mocker
docker development environment

## IN PROGRESS

```bash
#!/bin/bash

IMAGE_NAME="my-image"
CONTAINER_NAME="my-container"

function build_image_if_needed() {
    if [[ "$(docker images -q $IMAGE_NAME 2> /dev/null)" == "" ]]; then
        echo "[$IMAGE_NAME] image not found, building..."
        docker build -t $IMAGE_NAME .
    else
        echo "[$IMAGE_NAME] image already exists."
    fi
}

function run_or_start_container() {
    build_image_if_needed
    if [[ "$(docker ps -a --format '{{.Names}}' | grep -w $CONTAINER_NAME)" == "" ]]; then
        echo "[$CONTAINER_NAME] container not found, starting a new container..."
        docker run -d --name $CONTAINER_NAME $IMAGE_NAME
    else
        STATUS=$(docker inspect -f '{{.State.Status}}' $CONTAINER_NAME)
        if [[ "$STATUS" == "running" ]]; then
            echo "[$CONTAINER_NAME] is already running."
        else
            echo "[$CONTAINER_NAME] exists, starting..."
            docker start $CONTAINER_NAME
        fi
    fi
}

function stop_container_if_exists() {
    if [[ "$(docker ps -a --format '{{.Names}}' | grep -w $CONTAINER_NAME)" == "" ]]; then
        echo "[$CONTAINER_NAME] container not found, nothing to stop."
    else
        STATUS=$(docker inspect -f '{{.State.Status}}' $CONTAINER_NAME)
        if [[ "$STATUS" == "exited" ]]; then
            echo "[$CONTAINER_NAME] is already stopped."
        else
            echo "Stopping [$CONTAINER_NAME]..."
            docker stop $CONTAINER_NAME
        fi
    fi
}

case "$1" in
    build)
        build_image_if_needed
        ;;
    run|start)
        run_or_start_container
        ;;
    stop)
        stop_container_if_exists
        ;;
    *)
        echo "Usage: $0 {build|run|start|stop}"
        ;;
esac
```
