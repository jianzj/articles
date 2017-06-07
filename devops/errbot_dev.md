
# An Introduction about how to setup ErrBot in container and develop extra plugins

Errbot is a chatbot, a daemon that connects to your favorite chat service and brings your tools into the conversation.

## How to setup ErrBot environment

### Step 1 Preparation

Errbot allows you connecting several famous chat services, such as Slack, IRC, HipChat and so on. In this article, we are using Slack as chat service. 

- Prepare one slack environment with a bot user, which also generates an API token for bot user. [More Slack](https://slack.com/)

- Prepare one docker environment to run ErrBot container. [More Docker](https://www.docker.com/)

So now we have two items : `$slack_token` and `$demo_host` .

### Step 2 Build ErrBot Image

ErrBot provides much detail steps on document about how to install ErrBot , [More ErrBot Installation](http://errbot.io/en/latest/user_guide/setup.html#installation)

Thanks for Rafael Römhild help, he provides a way to run ErrBot in Docker, [rroemhild/docker-errbot](https://github.com/rroemhild/docker-errbot)

Based on personal situation, I forked the repo, and did some little change, to make sure we could remain `config.py` file if it exists in the directory during `docker run`. [jianzj/docker-errbot](https://github.com/jianzj/docker-errbot)

To build out ErrBot image, run the following steps :

- Git clone ErrBot repo on `$demo_host`

```
git clone https://github.com/jianzj/docker-errbot
cd ./docker-errbot
```

- Build out ErrBot image with name `errbot` and version `v1.0`

```
docker build -t errbot:v1.0 .
```

- Verify

```
docker images | grep errbot
```

### Step 3 Run ErrBot container

Detail info could be found in [README](https://github.com/jianzj/docker-errbot/blob/master/README.md)

Bellow is an example command

```
# Create a directory for ErrBot data persistent, will be mount to ErrBot container as volume
mkdir /tmp/errbot

docker run -it -d --name errbot -e BACKEND='Slack' -e BOT_TOKEN='$slack_token' -e BOT_ADMINS='@$your_slack_admin_user' -e BOT_ALT_PREFIXES='@$your_bot_name' -e CHATROOM_PRESENCE='' -v /tmp/errbot:/srv errbot:v1.0
```

To verify if your ErrBot container is up, you could 

```
# Find your container id and make sure status is Up
docker ps -a | grep errbot

# Go through container logs to make sure everything is fine
docker logs $containerId

# Using commands in Slack to make sure everything is well

!help
```

### Step 4 (Optional) Add Redis Plugin to implement data persistence

Now we have an environment to exercise and expand ErrBot functions. Please remember
  in ErrBot, __every module will be loaded as Plugin__ , it will be helpful for furture development.

In ErrBot, there is a way to store data in persistent database, so that these data will not be affected
  when ErrBot is restarted or rebuild. [Persistence](http://errbot.io/en/latest/user_guide/plugin_development/persistence.html)

In this article, we use Redis as a persistent database, which is a key-value database. 

- Prepare an available environment for Redis

```
# Based on Redis official docker images, we could create a redis container : https://hub.docker.com/_/redis/

mkdir /tmp/redis

docker run --it -d -v /tmp/redis:/data -p 6379:6379 --name redis redis redis-server --appendonly yes
```

- Add redis-storage plugin in ErrBot

```
# Refer to https://github.com/sijis/err-storage-redis

mkdir /tmp/errbot/plugins/redis-storage

git clone https://github.com/sijis/err-storage-redis.git
cd err-storage-redis

cp redisstorage.py /tmp/errbot/plugins/redis-storage
cp redisstorage.plug /tmp/errbot/plugins/redis-storage
```

- Add Redis Storage section into `config.py`

```
# Edit config.py in /tmp/errbot to add redis sections
BOT_EXTRA_STORAGE_PLUGINS_DIR='/srv/plugins/redis-storage'
STORAGE = 'Redis'
STORAGE_CONFIG = {
    'host': '$demo_host_ip',
    'port': 6379,
    'db': 0
}

# Restart container to load
docker restart $containerId
```

### Step 4 Develop personal plugins

ErrBot provide a detail introduction about every aside on how to develop plugins. [More Develop Guide](http://errbot.io/en/latest/user_guide/plugin_development/index.html)

In this article, we will introduce an ExtraConfig plugin, which provides a way to add your personal configuration file, used by other plugins; also we will show you how to use feature `DependsOn` to combine plugins.

- Create a plugin named ExtraConfig

The purpose of this plugin is to provide a way to configure other plugins with customer config file, seperated with ErrBot  configure file.

ExtraConfig code is here : https://github.com/jianzj/errbot-plugins/tree/master/extraConfig

There are two ways to assign config file

```
# One way is to override file config.json in ./extraConfig/etc

# The other way is to create your own config.json and provide to ErrBot by adding section in config.py

EXTRA_CONFIG_FILE = "$your_config_file"
```

- Create one plugin using ExtraConfig as DependsOn

ErrBot provides a way to import other plugins as dependence, [Plugin Dependencies](http://errbot.io/en/latest/user_guide/plugin_development/dependencies.html). We provide a plugin named PagerDuty as example.

Create `pagerduty.plug` file, which contains all necessary metadata. Section `DependsOn` describes which plugins you will use.

```
# Create pagerduty.plug
[Core]
Name = PagerDuty
Module = pagerduty
DependsOn = ExtraConfig, UtilsFunc

[Python]
Version = 2+

[Documentation]
Description = Plugin for PagerDuty
```

Create `pagerduty.py` file, to implement features.

```
class PagerDuty(BotPlugin):
    
    def activate(self):
        super().activate()
        
        self.extraConfig = self.get_plugin("ExtraConfig")

    def get_service(self, serviceId):
    	demoData = self.extraConfig.load("PagerDuty")
    	token = demoData['token']
```

We could import any depends plugins with func `get_plugin(PluginName)` , `PagerDuty` plugin code could be found in https://github.com/jianzj/errbot-plugins/tree/master/pagerduty .
