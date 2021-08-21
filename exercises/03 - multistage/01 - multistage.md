# Multistage builds

## Getting Medusa example
We will look at a multistage build done by the project `Medusa`

> Medusa is a cli tool currently for importing and exporting a json or yaml file into HashiCorp Vault. 

Clone the medusa project down

```
git clone https://github.com/jonasvinther/medusa.git
```

## The Dockerfile
Examine the provided Dockerfile
```
cd medusa

cat Dockerfile
FROM golang:alpine AS builder

RUN apk update && apk add --no-cache git

ENV GO111MODULE=on

WORKDIR /app

ADD . .

RUN go mod download
RUN go get -d -v
RUN CGO_ENABLED=0 GOOS=linux go build -o /go/bin/medusa

RUN adduser -S scratchuser
RUN chown scratchuser /go/bin/medusa

FROM scratch
COPY --from=builder /go/bin/medusa /medusa
COPY --from=builder /etc/passwd /etc/passwd
USER scratchuser
ENTRYPOINT ["/medusa"]
```

First we create a layer from the golang:alpine docker image and call it `builder`.

Then we update and add `git` to the image. 

We set a variable and adds all files from the current location

We then setup go and build the binary in `/go/bin/medusa`.

A user is then created called `scratchuser` and given ownership of the go binary we created.

Then we shift to the `scratch` image which basically clears everything from the last image. 

We can still access things from the last image, and we do that by the copy command and use the `--from=builder` flag. The medusa binary is copyed to /medusa and so is the /etc/passwd to get the ownership and user to match again.

We then shift to the user `scratchuser` before running `medusa`.

## Lets build medusa
We can build `Medusa` by running a normal build command
```
docker build -t medusa .
Sending build context to Docker daemon  399.4kB
Step 1/15 : FROM golang:alpine AS builder
alpine: Pulling from library/golang
29291e31a76a: Already exists 
e4bc8fc554c3: Pull complete 
803daa35ea47: Pull complete 
6c2d8e2bae38: Pull complete 
......
```

When done, we have a small Medusa image that doesn't have all the go libraries needed to build the binary. Lets checkout the size

```
docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
medusa        latest    8d8b2c95c500   9 seconds ago    11.5MB
```

Wov, under 12 MB. 

Let's see if it works. Sinse the Medusa container runs `medusa` as an entrypoint, we dont need to specify the medusa command, only the parameters.

```
docker run medusa -h 
Medusa is a cli tool currently for importing a json or yaml file into HashiCorp Vault.
Created by Jonas Vinther & Henrik Høegh.

Usage:
  medusa [command]

Available Commands:
  export      Export Vault secrets as yaml
  help        Help about any command
  import      Import a yaml file into a Vault instance

Flags:
  -a, --address string   Address of the Vault server
  -h, --help             help for medusa
  -k, --insecure         Allow insecure server connections when using SSL
  -t, --token string     Vault authentication token

Use "medusa [command] --help" for more information about a command.
```