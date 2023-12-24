---
layout: post
title:  "Set up docker on Ruby on Rails application"
date:   2018-07-25 09:23:58 +0545
categories: tutorials
---

First you need to install docker, docker-compose, docker-machine, on your system.

# We need following files initially
 
 # install scripts
 dockerfile
 # ignore while pushing the code
 .dockerignore
 # list env vars
 .env
 # run the processes, containers and apps
 docker-compose.yml

# Run the services and processes
docker-compose up 

# Build dependencies
docker-compose --build 

 # to see container list
docker container ls

 # to see all the processes
docker ps

 # to stop all the processes, so instead of killing all the container and processes, later on you can restart again
docker-compose stop

 # website is the name of the service to run rails app
docker-compose run --rm website rake db:create db:migrate

docker-compose run website rake db:migrate db:seed RAILS_ENV=production

# enter into bash shell
sudo docker run -i -t <image/id> /bin/bash

# forcefully remove image
docker rmi 24a77bfbb9ee -f

# remove all the containers
docker rm $(docker ps -a -q)

# Note: If you don’t have Docker running on your local machine, you need to replace localhost in the above URL with the IP address of the machine Docker is running on. If you’re using Docker Machine, you can run below cmd to find out the IP.
docker-machine ip “${DOCKER_MACHINE_NAME}”

# To run rails console
docker-compose exec website rails console
