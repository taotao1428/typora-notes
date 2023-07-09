



```shell
docker run -v /Users/hewutao/docker/extfile/sentinel4:/usr/local/etc/redis --name sentinel4 redis redis-sentinel /usr/local/etc/redis/sentinel.conf


docker run -d -v /Users/hewutao/docker/extfile/sentinel5:/usr/local/etc/redis --name sentinel5 redis redis-sentinel /usr/local/etc/redis/sentinel.conf
```