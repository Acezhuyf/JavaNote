# DockerNote

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