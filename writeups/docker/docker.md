# Docker Intro
This is the log to create docker images and run programs

## 3. Docker

--> instructions at https://docs.nvidia.com/deeplearning/frameworks/user-guide/index.html

### 3.1 Docker build

--> get the version of the docker with pytorch https://catalog.ngc.nvidia.com/orgs/nvidia/containers/pytorch/tags

#### Dockerfile (reccomended👍)

Docker File:

```
FROM your-base-image:latest
ARG USER=standard
ARG USER_ID=1000 # uid from the previus step --> from command id and e3da group
ARG USER_GROUP=standard
ARG USER_GROUP_ID=1000 # gid from the previus step --> from command id and e3da
ARG USER_HOME=/home/${USER}
# create a user group and a user (this works only for debian based images)
RUN groupadd --gid $USER_GROUP_ID $USER \
    && useradd --uid $USER_ID --gid $USER_GROUP_ID -m $USER
# setup image istructions
RUN apt-get update && apt-get install -y curl
# set container user
USER $USER
# run script as non-root user
ENTRYPOINT ["python", "helloworld.py"]
```

then run:
```
docker build -t yolo:latest .
```

General
```
docker build -t {name}:{tag} .
```
{name} -> name of the container
{tag} -> just the tag or version of the docker, usually latest

### 3.3 Docker Run 

--> RAM SIZE MAX --> 1/N_gpu * GPU prenotate da te
1/3 per me

Run docker external file
```
docker run --name test-0 -d --gpus all --ipc=host --shm-size=4gb yolo:latest ./main.sh
```

Run docker importing a single file
```
docker run --name test-0 -d --gpus all --ipc=host --shm-size=4gb -v /raid/home/e3da/interns/scavada/yolo-docker/main.sh:/workspace/main.sh yolo:latest
```

Run docker importing a directory
```
docker run --name test-0 -d --gpus all --ipc=host --shm-size=32gb -v $PWD/volume:/workspace/ micromind:latest
```

Run docker importing a directory - with pwd
```
docker run --name test-0 -d --gpus all --ipc=host --shm-size=32gb -v "$(pwd)/data:/home/standard/data" yolo:latest
```

Run docker with interactive bash
```
docker run --name test-0 --gpus all --rm -it --shm-size=32gb yolo:latest bash
```

### 3.4 Docker Remove container

```
docker rm {containerID}
```

### 3.5 Docker Remove image
```
docker rmi {imageID}
```


#### NOTE:
Always create on the host PC a build.sh file and a run.sh file to make everything faster.

Example:

##### build.sh
```
docker build -t yolo:latest .
```

##### run.sh
```
docker run --name test-1 --gpus all --rm -it --shm-size=32gb -v $PWD/volume:/workspace/ -v /raid/home/e3da/datasets/coco/:/workspace/scripts/datasets/coco/ yolo:latest bash
```


## Others

#### main.sh (on the docker)

```
CUDA_VISIBLE_DEVICES=1 python classification.py ~/data/mnist -b 128 --dataset torch/mnist --num-classes 10 \ --model phinet --input-size 1 28 28 --epochs 20 --amp \ --opt adam --lr 0.01 --weight-decay 0.01 --no-aug \ --pin-mem --apex-amp --use-multi-epochs-loader --mean 0.1307 --std 0.3081 --dataset-download --log-interval 100 \ --alpha 0.5 --num_layers 4 --beta 1 --t_zero 6 --experiment mnist
```

priviledges --> 
```
chmod +x {file.sh}
```

Aggiungere in base alla GPU che si vuole andare ad utilizzare
```
CUDA_VISIBLE_DEVICES=1
```

### 3.6 Logs

Follow the logs:
```
docker logs 21fdd7104177 -f --tail 50
```

## 3.7 Magic screen

Connect to the bash of the image and then detach when you need to exit without stopping the machine.

```
screen -S <session-name>
```

#### NOTE: -S is capital letter

Run ./run.sh

To exit --> ctrl-a + ctrl-d

To reconnect

```
screen -r <session-name>
```