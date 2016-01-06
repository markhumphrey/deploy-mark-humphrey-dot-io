# deploy-mark-humphrey-dot-io

## Contents

Docker Compose base service definition file
```
docker-compose.yml
```

Docker Compose override definition files
```
docker-compose.dev.yml
docker-compose.prod.yml
```

AWS Elastic Beanstalk multi-container service definition file
```
Dockerrun.aws.json
```

## Dependencies

* Docker
* Docker Machine
* Docker Compose

## Setup
Clone this repo to get the container orchestration and deployment code
```
git clone git@github.com:markhumphrey/deploy-mark-humphrey-dot-io.git
```

The orchestration configuration looks for container definitions in repos within the same parent directory.

Clone the repos of the containers which you want to modify locally. Otherwise the
production container images will be pulled from DockerHub.
```
git clone git@github.com:markhumphrey/api-mark-humphrey-dot-io.git
git clone git@github.com:markhumphrey/www-mark-humphrey-dot-io.git
```

Install Docker Compose and dependencies:
https://docs.docker.com/compose/install/

## Start Docker Machine

To run:
```
docker-machine start default
```

Display the environment settings for that machine:

```
docker-machine env default
```

Should produce output similar to:

```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/mhumphrey/.docker/machine/machines/default"
export DOCKER_MACHINE_NAME="default"
# Run this command to configure your shell:
# eval "$(docker-machine env default)"
```

Run the eval command in each shell that you will be using docker or docker-compose commands in:
```
eval "$(docker-machine env default)"
```

The DOCKER_HOST ip address is the address you will use when connecting to your running docker services.

## Build containers locally
To build all:
```
cd deploy-mark-humphrey-dot-io
docker-compose -f docker-compose.yml -f docker-compose.dev.yml build
```

To build a single container. In this case the container named "www":
```
docker-compose -f docker-compose.yml -f docker-compose.dev.yml build www
```

## Run containers
To run all containers:
```
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

To run a single container. In this the container named "www"
```
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up --no-deps www
```

## Connect to running container
First run containers:
```
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

Should see some output at the top similar to:
```
Starting deploymarkhumphreydotio_sql_1
Starting deploymarkhumphreydotio_api_1
Starting deploymarkhumphreydotio_www_1
```

Copy the container name that you would like to connect to and specify a command to execute:

```
docker exec -it deploymarkhumphreydotio_www_1 bash
```

## Data volume containers

The front end build tooling is not needed in production and would require
merging the nginx and node official images. It is currently separated out into an
image that generates build artifacts and exposes as a volume /usr/share/nginx/html
that is mounted by the nginx container.

IMPORTANT
When a data volume image that is mounted using volumes_from is changed the nginx
container needs to be removed in order for docker-compose to reflect the change.

To remove all stopped containers:
```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml rm
```

## Push new production image to DockerHub
To build production containers locally:
```
cd deploy-mark-humphrey-dot-io
docker-compose -f docker-compose.yml -f docker-compose.prod.yml build
```

To run production containers locally:
```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

To push production containers to DockerHub:
First tag built production image:
```
docker tag <image> <tag>
```

For example:
```
docker tag 59f55c59fcbf markhumphrey/www-mark-humphrey-dot-io:latest
```

To log into DockerHub from the command line:
```
docker login --username=markhumphrey
```

To push image to the repository:
```
docker push markhumphrey/www-mark-humphrey-dot-io
```

## Deploy to Amazon EC2 Container Service
http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html
http://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html

Configure AWS ECS cli:
```
ecs-cli configure --region "us-east-1" --access-key $AWS_ACCESS_KEY_ID --secret-key $AWS_SECRET_ACCESS_KEY --cluster "www-mark-humphrey-dot-io"
```

The ecs-cli can generate a task definition from a docker-compose.yml file, but
some tweaking is necessary so a separate docker-compose.aws.yml is maintained.

Create task definition on cluster:
```
ecs-cli compose -f docker-compose.aws.yml -p "www-markhumphrey-dot-io" create
```

This will create a task named ecscompose-<PROJECT_NAME> that can be viewed in
the AWS ECS Task Definitions console.

IMPORTANT: Need to manually go into the AWS ui and mark any data container tasks
which exit immediately as ```essential: false``` or the AWS task will terminate
