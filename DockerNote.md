# DockerNote

## Docker常用指令
https://www.cnblogs.com/zhujingzhi/p/9815179.html

## 配置container

配置文件为Dockerfile

```docker
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

## 编译构建container

```docker
docker build --tag=friendlyhello .
```

## 运行container

```docker
#前台模式运行
docker run -p 4000:80 friendlyhello
#后台模式运行
docker run -d -p 4000:80 friendlyhello
```

4000是访问的port，80是container开放的port

## 其他command

```docker
#查看正在运行的container
docker container ls
#停止container，只需要输入前几位id即可，类比git
docker container stop 1fa4ab2cf395
```

## 上传image

```docker
#输入账号密码
docker login
#为已存在的image复制并重命名
docker tag image username/repository:tag
#比如命名为acezhuyf/test
docker tag friendlyhello acezhuyf/test
#上传image
docker push acezhuyf/test
#上传成功后，在任何机器上执行这段命令即可运行
docker run -p 4000:80 acezhuyf/test
```

## Docker部署负载均衡集群

建一个文件docker-compose.yml

```yml
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    #会从我的Docker Hub拉取这个名字的image
    image: acezhuyf/test
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "4000:80"
    networks:
      - webnet
networks:
  webnet:
```

```docker
#部署
docker swarm init

docker stack deploy -c docker-compose.yml test
#查看集群
docker ps #等于 docker container ls [-q]
docker stack ps test #等于 docker service ps test_web
docker stack services test #查看stack状态
```

```docker
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```

## Docker-machine安装

```
需要翻墙

base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine

#开启机器
docker-machine start name
```

## 建立cluster

```docker
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2

docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376 

#使用2377端口
#Always run docker swarm init and docker swarm join with port 2377 (the swarm management port), or no port at all and let it take the default.

#The machine IP addresses returned by docker-machine ls include port 2376, which is the Docker daemon port. Do not use this port or you may experience errors.

docker-machine ssh myvm1 "docker swarm init --advertise-addr 192.168.99.100:2377"

Swarm initialized: current node (5uq4vx2y99xlo361s5m73svir) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-22rmqcoj7rs9hyv8dvrgdj3f09f901ujcbv1ry345fb6d8avii-dan1aiw63y3ek1qzuddfolrn6 192.168.99.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

#将一台机器作为worker加入cluster
docker-machine ssh myvm2 “docker swarm join --token SWMTKN-1-22rmqcoj7rs9hyv8dvrgdj3f09f901ujcbv1ry345fb6d8avii-dan1aiw63y3ek1qzuddfolrn6 192.168.99.100:2377”

docker-machine ssh myvm1 "docker node ls"
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
brtu9urxwfd5j0zrmkubhpkbd     myvm2               Ready               Active
rihwohkh3ph38fhillhhb84sk *   myvm1               Ready               Active              Leader
```

## 将app部署到cluster

```docker
#查看可以使用的vm
docker-machine ls
#配置myvm1
eval $(docker-machine env myvm1)

#这时执行命令不需要加sudo，加了sudo是操作本机的环境，直接执行是操作eval设置的虚拟机
#进入到存放docker-compose.yml的文件夹执行
docker stack deploy -c docker-compose.yml getstartedlab

docker stack ps
#部署完毕后访问虚拟机ip:4000(port在配置文件中)
http://192.168.99.101:4000/
```

