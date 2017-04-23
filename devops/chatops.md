
#### ChatOps - To make team communication and collaboration better


> Put commands and conversation into one fire-room, better to log, co-work and learn.
> Better to use and easily reolve problems with commands in Chat, making everyone pairing every time.

Architecture Diagram

![image](images/chatops.jpg)

Reference Links

- https://www.youtube.com/watch?v=NST3u-GjjFw  A video from Github which is practicing ChatOps
- https://blog.hipchat.com/2016/02/05/chatops-guide-evolution-adoption-significance/  A good article to introduce
- http://nordicapis.com/12-frameworks-to-build-chatops-bots/  Famouse Frameworks

Two well-known framework:
- hubot : writen in Go , not a chance to practice ;
- errbot : writen in Python

Demo with ErrBot and Slack :

> Setup Demo environment into Container on Kubernetes

1. Based on docker image [rroemhild/errbot](https://hub.docker.com/r/rroemhild/errbot/), make some changes to support storage plugin ```redis```

- Please refer to this source https://github.com/jianzj/devops-sys/tree/master/docker/errbot and build out docker image manually
```docker build -t errbot:special .```

2. Prepare environment, create directories for errbot persistent data, plugins and storage plugins
```
mkdir -p /tmp/err-srv/plugins
mkdir -p /tmp/err-srv/data
mkdir -p /tmp/err-srv/storages
mkdir -p /tmp/err-srv/errbackends
```

- Put errbot redis plugin into directory, source code could be found https://github.com/sijis/err-storage-redis

You could write storage plugin yourself ether with guidance http://errbot.io/en/latest/user_guide/storage_development/index.html . Only a few functions need to be implemented. For this demo, I used one plugin found in github which support for redis

```
copy ./err-storage-redis/redisstorage.py /tmp/err-srv/storages
copy ./err-storage-redis/redisstorage.plug /tmp/err-srv/storages
```

3. Create deployment for ErrBot in Kubernetes

Notice : Sometimes in your kubernetes cluster, maybe in your pod, ```slack.com``` could not be resloved. You should change some configurations for your ```kube-dns``` deployment.

- Deployment Yaml file could be found https://github.com/jianzj/devops-sys/blob/master/kubernetes/others/errbot/errbot-deployment.yml
- Please install redis service into your kubernetes cluster, one easy way to do this is using ```helm``` and ```charts```
- Put demo plugin into /tmp/err-srv/plugins/demo01 , demo source code could be found https://github.com/jianzj/devops-sys/tree/master/demo/errbot/democmd
- Create errbot deployment with command
```kubectl create -f ./errbot-deployment.yml```

4. Verify your environment

In your Slack channel, set for ErrBot, could run command ```!help``` and ```!democmd add``` and ```!democmd get```


