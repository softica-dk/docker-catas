# Entrypoint vs cmd

An `entrypoint` is a step in the container that will always run, unless you explicit tell it not to.

Its a great way to configure your container before running the process you want to run.

Imaging you have a script that creates a html file based on some environment variables. You would want this to run every time the container is run.

The script `create-token-web.sh` could look like this
```
#!/bin/sh

# Create the html file based on variables
echo "
<!DOCTYPE html>
<html>
<body>

<h1>Hello ${USER}</h1>
<p>The secret token is ${TOKEN}</p>
<marquee>shhhh.....</marquee>
</body>
</html>
" > /usr/share/nginx/html/index.html

# Execute the command specified as CMD in Dockerfile:
exec "$@"
```

In the same folder we create a `Dockerfile` and base it on `nginx` like the last exercise, but with some modifications
```
FROM nginx:mainline-alpine

ENV USER="Henrik"
ENV TOKEN="Johnny-doesnt-use-rubberboots"

COPY create-token-web.sh /root/create-token-web.sh
RUN chmod u+x /root/create-token-web.sh

ENTRYPOINT ["/bin/sh", "/root/create-token-web.sh"]
CMD ["nginx", "-g", "daemon off;"]
```

Now we can run our container
```
docker run -d --name=web -p80:80 hoeghh/token-web:1.0
a17c80d0e38c22e1c2e03455dd85bb83baa7a93a8dd1f1036358d6e0da94b5d8
```

Go to a browser and go to `http://localhost` and you should see the secret token

Check the logs of the container
```
docker logs web
2021/08/21 12:30:23 [notice] 1#1: using the "epoll" event method
2021/08/21 12:30:23 [notice] 1#1: nginx/1.21.1
2021/08/21 12:30:23 [notice] 1#1: built by gcc 10.3.1 20210424 (Alpine 10.3.1_git20210424) 
2021/08/21 12:30:23 [notice] 1#1: OS: Linux 5.13.7-200.fc34.x86_64
2021/08/21 12:30:23 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2021/08/21 12:30:23 [notice] 1#1: start worker processes
2021/08/21 12:30:23 [notice] 1#1: start worker process 7
2021/08/21 12:30:23 [notice] 1#1: start worker process 8
2021/08/21 12:30:23 [notice] 1#1: start worker process 9
2021/08/21 12:30:23 [notice] 1#1: start worker process 10
2021/08/21 12:30:23 [notice] 1#1: start worker process 11
2021/08/21 12:30:23 [notice] 1#1: start worker process 12
2021/08/21 12:30:23 [notice] 1#1: start worker process 13
2021/08/21 12:30:23 [notice] 1#1: start worker process 14
```

## Clean up
Lets remove the container and image
```
docker rm -f web
web

docker rmi -f hoeghh/token-web:1.0
Untagged: hoeghh/token-web:1.0
Deleted: sha256:fd8bba6d37950f265f6d5dd1f840dd3c7cf252284c731509fe33f049a0035915
Deleted: sha256:5a662163329a9406547053801bbb149155e548124532d1f03218305c2752962f
Deleted: sha256:babc8937e55159915302c42621bce4349bc85da59389cf6608bec01978b87b05
Deleted: sha256:184b00a460ef3e7a8dacd054a67ad36688c9d799999a456efa8b63c88b9483e6
```