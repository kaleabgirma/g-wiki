---
title: Docker Garbage Collection
description: This documentation  will guide you through how to collect garbage and clear log from your docker container
published: true
date: 2019-10-22T16:12:31.101Z
tags: 
---

## View Docker log file sizes

To view the docker container log file sizes use the following command in your console 
``` sh
du -chs /var/lib/docker/containers/*/*json.log
```
## Clearing garbage’s
If your system disk is getting full use the following command to clear the garages in your docker container engine

The following command will remove all unused networks, volumes, docker images etc.
``` sh
docker system prune -a --volumes
```

## Clearing docker container log 
If your docker container log is growing unexpectly and fill your system disk then use the following step to clear your larger container log, to get the bigger size of your log use the above log size shower command.

First create the following file in your **/usr/local/bin** directory by executing the following command

``` sh
nano /usr/local/bin/docker-clear-log
```
Then paste the following code in the file, save and close <kbd>ctrl</kbd>+<kbd>x</kbd> then <kbd>y</kbd>

``` bash
#!/bin/bash -e

if [[ -z $1 ]]; then
    echo "No container specified"
    exit 1
fi

if [[ "$(docker ps -aq -f name=^/${1}$ 2> /dev/null)" == "" ]]; then
    echo "Container \"$1\" does not exist, exiting."
    exit 1
fi

log=$(docker inspect -f '{{.LogPath}}' $1 2> /dev/null)
truncate -s 200 $log
```
Then run the following command to give permission for the file to make it runnable
``` sh
chmod 755 /usr/local/bin/docker-clear-log
```

Finally run the following command to clear the log until it reaches 100Mb
``` sh
docker-clear-log <container name>
```
Example:
``` sh
docker-clear-log jhipster-console-logstash-1
```