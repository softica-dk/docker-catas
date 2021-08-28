# Docker login

## Why
Docker hub has added a rate limit to docker hub, meaning that if you pull images anonymously you will be limited to 100 pulls pr 6 hours. This is based on the ip the docker hub sees, which is everyone in this building.

To overcome that, we need to register at docker hub by creating an account and then login from the cli.

## Create an docker hub account
Go to `https://hub.docker.com` and enter an `Docker id` (username), your `Email` and a `password`. Complete the Captcha and click `Sign up`.

## Logging in to the terminal
In your terminal run `docker login`. This should create a file called `~/.docker/config.json` with your login information.

## Test your rate limit
To test you rate limit, run the following bash commands
> To run the command, you need jq installed in your terminal

```
TOKEN=$(curl --user 'username:password' "https://auth.docker.io/token?service=registry.docker.io&scope=repository:ratelimitpreview/test:pull" | jq -r .token)
curl --head -H "Authorization: Bearer $TOKEN" https://registry-1.docker.io/v2/ratelimitpreview/test/manifests/latest
```
If you dont have `jq` installed, simply extract the token manually and put it in the second command manually as well. 

You should get an output that looks like this
```
HTTP/1.1 200 OK
content-length: 2782
content-type: application/vnd.docker.distribution.manifest.v1+prettyjws
docker-content-digest: sha256:767a3815c34823b355bed31760d5fa3daca0aec2ce15b217c9cd83229e0e2020
docker-distribution-api-version: registry/2.0
etag: "sha256:767a3815c34823b355bed31760d5fa3daca0aec2ce15b217c9cd83229e0e2020"
date: Sat, 28 Aug 2021 10:01:50 GMT
strict-transport-security: max-age=31536000
ratelimit-limit: 200;w=21600
ratelimit-remaining: 200;w=21600
```

## More info
Go to `https://docs.docker.com/docker-hub/download-rate-limit/` and `https://docs.docker.com/engine/reference/commandline/login/`