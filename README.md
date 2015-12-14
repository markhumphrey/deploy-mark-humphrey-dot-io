# deploy-mark-humphrey-dot-io

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

Should see some output at the top simlar to:
```
Starting deploymarkhumphreydotio_sql_1
Starting deploymarkhumphreydotio_api_1
Starting deploymarkhumphreydotio_www_1
```

Copy the container name that you would like to connect to and specify a command to execute:

```
docker exec -it deploymarkhumphreydotio_www_1 bash
```

## Deploy to AWS Elastic Beanstalk
TODO
