# Dockerfile - simple nginx

## Writting the Dockerfile
Here we are going to build a simple `Dockerfile`. We will base our image on the official `Nginx` image.

Create a folder `hello-nginx` and a `Dockerfile` inside it

```
mkdir hello-nginx
cd hello-nginx
vi Dockerfile
```

The image we want, should be based on the official Nginx image, so we will start with 
```
FROM nginx:mainline-alpine
```

Next we want to have some variables so that its easy to maintain the image in the future. So we add

```
ENV NAME="Henrik"
ENV PICTURE="https://i1.wp.com/hajdupress.hu/wp-content/uploads/2020/11/How-To-Say-Hello-in-10-Languages.jpg"
```

The first line uses `ENV` to create an environment variable in the container image called `NAME` and sets it to the value `Henrik`.
The next like also uses `ENV` to create a environment variable called PICTURE and sets it to the adress of an image from the internet.

Next, we fetch the image from the internet by running `curl` and outputs the image at a specific location as `hello.jpg`

> The image from the internet in this example needs to be in jpg format to work

```
RUN curl -s -k ${PICTURE} -o /usr/share/nginx/html/hello.jpg
```

Now we are ready to create our custom `html` file that nginx will be serving.
```
RUN echo "<center><h1>Hello $NAME </h1><p> <img src=\"hello.jpg\"></center>" > /usr/share/nginx/html/index.html
```

We wont create any `CMD` or `ENTRYPOINT` steps, as these are inherit by the nginx image. 

The complete `Dockerfile` looks like this
```
FROM nginx:mainline-alpine

ENV NAME="Henrik"
ENV PICTURE="https://i1.wp.com/hajdupress.hu/wp-content/uploads/2020/11/How-To-Say-Hello-in-10-Languages.jpg"

RUN curl -s -k ${PICTURE} -o /usr/share/nginx/html/hello.jpg

RUN echo "<center><h1>Hello $NAME </h1><p> <img src=\"hello.jpg\"></center>" > /usr/share/nginx/html/index.html
```

## Building the Docker image
Lets try and build the image, giving it the name `webserver` and the tag `simple-nginx`

```
docker build -t webserver:simple-web .
Sending build context to Docker daemon  2.048kB
Step 1/5 : FROM nginx:mainline-alpine
 ---> 7ce0143dee37
Step 2/5 : ENV NAME="Henrik"
 ---> Using cache
 ---> 1121b2289ce2
Step 3/5 : ENV PICTURE="https://i1.wp.com/hajdupress.hu/wp-content/uploads/2020/11/How-To-Say-Hello-in-10-Languages.jpg"
 ---> Using cache
 ---> d508d8561adf
Step 4/5 : RUN curl -s -k ${PICTURE} -o /usr/share/nginx/html/hello.jpg
 ---> Using cache
 ---> 91788aa02eba
Step 5/5 : RUN echo "<center><h1>Hello $NAME </h1><p> <img src=\"hello.jpg\"></center>" > /usr/share/nginx/html/index.html
 ---> Using cache
 ---> aced176b010f
Successfully built aced176b010f
Successfully tagged webserver:simple-web
```

We can now see that we have a docker image on our laptop called webserver with the tag simple-nginx

```
docker images
REPOSITORY   TAG               IMAGE ID       CREATED          SIZE
webserver    simple-web        aced176b010f   12 minutes ago   22.9MB
```

> Since we are using an alpine version of the base image, it only takes up ~23 MB.

## Running a container
Now, lets try and run our new Docker image. We will run it it the background. 

```
docker run -d -p 80:80 webserver:simple-web
```
> You can add --name=webserver to make it easy to specify the container later on

> The `-d` tells Docker to run the container in the background. The `-p 80:80` tells Docker to map the port `80` on the host to port `80` in the container.

Now open a browser and go to http://localhost. You should see a webpage saying hello to me.

If we want to see the logs of the container, we first find the container id, and then the logs:

```
docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                               NAMES
1ef1bccf03f2   webserver:simple-web   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   nifty_hypatia

docker logs 1ef1bccf03f2
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2021/08/21 11:41:35 [notice] 1#1: using the "epoll" event method
2021/08/21 11:41:35 [notice] 1#1: nginx/1.21.1
2021/08/21 11:41:35 [notice] 1#1: built by gcc 10.3.1 20210424 (Alpine 10.3.1_git20210424) 
2021/08/21 11:41:35 [notice] 1#1: OS: Linux 5.13.7-200.fc34.x86_64
2021/08/21 11:41:35 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2021/08/21 11:41:35 [notice] 1#1: start worker processes
2021/08/21 11:41:35 [notice] 1#1: start worker process 33
2021/08/21 11:41:35 [notice] 1#1: start worker process 34
......
```
> Or, if given a name simply run `docker logs webserver`

If we want to go inside the container and have a look around, we can run

```
docker exec -it 1ef1bccf03f2 /bin/sh
/ # 
```

> To exit, type `exit` or `Ctrl + d`.

## Stopping a container
The container is still running, so lets stop it
```
docker stop 1ef1bccf03f2
```

## Shipping the container
I have a personal account on `docker hub` called `hoeghh`. So to push my image, i would need to re-tag it

```
docker tag webserver:simple-web hoeghh/webserver:simple-web
```

Now i can push it to the Docker hub
```
docker push hoeghh/webserver:simple-web
The push refers to repository [docker.io/hoeghh/webserver]
2f4c14406b9a: Pushed 
a12477f7b961: Pushed 
cb460864147e: Mounted from library/nginx 
c126a1b4f317: Mounted from library/nginx 
a33cbf3e5439: Mounted from library/nginx 
b9239390c1b8: Mounted from library/nginx 
82c3b921d80c: Mounted from library/nginx 
bc276c40b172: Mounted from library/nginx 
simple-web: digest: sha256:59943656801d26de91591cd652b9bc78adab8487096eecd3cb62127ab0840990 size: 1984
```

Now, others can run my container by
```
docker run --name=webserver -d -p 80:80 hoeghh/webserver:simple-web
```

## Cleanup
Lets remove our stuff again
### Deleting the docker container
```
docker rm -f webserver
```

### Deleting the docker image
```
docker rmi -f webserver:simple-web
docker rmi -f hoeghh/webserver:simple-web
docker rmi -f nginx:mainline-alpine
```

## Bonus
Try changing the variables from `ENV` to `ARG` and build it with a custom `NAME`

> help : https://docs.docker.com/engine/reference/builder/#arg