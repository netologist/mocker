# mocker
docker development environment

## IN PROGRESS

```bash
#!/bin/bash

IMAGE_NAME="my-image"
CONTAINER_NAME="my-container"
VOLUME_NAME="my-volume"
VOLUME_PATH="/data"
NETWORK_NAME="my-network"

function build_image_if_needed() {
    if [[ "$(docker images -q $IMAGE_NAME 2> /dev/null)" == "" ]]; then
        echo "[$IMAGE_NAME] image not found, building..."
        docker build -t $IMAGE_NAME .
    else
        echo "[$IMAGE_NAME] image already exists."
    fi
}

function create_volume_if_needed() {
    if [[ "$(docker volume ls -q | grep -w $VOLUME_NAME)" == "" ]]; then
        echo "[$VOLUME_NAME] volume not found, creating..."
        docker volume create $VOLUME_NAME
    else
        echo "[$VOLUME_NAME] volume already exists."
    fi
}

function create_network_if_needed() {
    if [[ "$(docker network ls -q -f name=^${NETWORK_NAME}$)" == "" ]]; then
        echo "[$NETWORK_NAME] network not found, creating..."
        docker network create $NETWORK_NAME
    else
        echo "[$NETWORK_NAME] network already exists."
    fi
}

function run_or_start_container() {
    build_image_if_needed
    create_volume_if_needed
    create_network_if_needed
    if [[ "$(docker ps -a --format '{{.Names}}' | grep -w $CONTAINER_NAME)" == "" ]]; then
        echo "[$CONTAINER_NAME] container not found, starting a new container..."
        docker run -d \
            --name $CONTAINER_NAME \
            --mount type=volume,src=$VOLUME_NAME,dst=$VOLUME_PATH \
            --network $NETWORK_NAME \
            $IMAGE_NAME
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

function clean_container_and_volume() {
    if [[ "$(docker ps -a --format '{{.Names}}' | grep -w $CONTAINER_NAME)" != "" ]]; then
        echo "Removing [$CONTAINER_NAME]..."
        docker rm -f $CONTAINER_NAME
    else
        echo "[$CONTAINER_NAME] does not exist."
    fi

    if [[ "$(docker volume ls -q | grep -w $VOLUME_NAME)" != "" ]]; then
        echo "Removing [$VOLUME_NAME] volume..."
        docker volume rm $VOLUME_NAME
    else
        echo "[$VOLUME_NAME] volume does not exist."
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
    clean)
        clean_container_and_volume
        ;;
    *)
        echo "Usage: $0 {build|run|start|stop|clean}"
        ;;
esac
```
