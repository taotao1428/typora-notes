启动flask

```
flask run -h 0.0.0.0 -p 8000
```





```
docker run -d --name http-server1 -v /Users/hewutao/docker/extfile/nginx/server1:/usr/src/app --ip 172.17.0.100 flask flask run -h 0.0.0.0 -p 4000


docker run -d --name http-server1 -v /Users/hewutao/docker/extfile/nginx/server2:/usr/src/app --ip 172.17.0.100 flask flask run -h 0.0.0.0 -p 4000


docker run -d --name http-server-mynet-1 -v /Users/hewutao/docker/extfile/nginx/server1:/usr/src/app --network mynet --ip 192.168.0.2 flask flask run -h 192.168.0.2 -p 4000

docker run -d --name http-server-mynet-2 -v /Users/hewutao/docker/extfile/nginx/server2:/usr/src/app --network mynet --ip 192.168.0.4 flask flask run -h 192.168.0.4 -p 4000
```



/Users/hewutao/docker/extfile/nginx/server1/app.py





192.168.0.2 server1

192.168.0.4 server2