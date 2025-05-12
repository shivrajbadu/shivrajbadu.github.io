---
layout: post
title:  "Set up docker on Ruby on Rails application"
date:   2018-07-25 09:23:58 +0545
categories: [Docker, Ruby on Rails]
tags: [docker, ruby on rails]
---

## Docker for Rails: From Development to Deployment (Complete Guide)

##### Docker simplifies building, shipping, and running applications in isolated containers. This guide covers:  

✅ **Running a Rails app with Docker**  
✅ **Deploying to Docker Hub**  
✅ **Running the app on another machine with zero setup**  

## 1. Setting Up a Rails App with Docker

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop) installed  
- [Docker Hub](https://hub.docker.com/) account  

### Step 1: Create a Rails App

```bash
rails new docker_rails_demo --database=postgresql
cd docker_rails_demo
```

#### Add a Welcome Page
```bash
rails generate controller Welcome index
```

Update `config/routes.rb`:
```ruby
Rails.application.routes.draw do
  root 'welcome#index'
end
```

### Step 2: Docker Configuration

# ignore while pushing the code
.dockerignore

# list env vars
.env

#### 1. Write script for installation. `Dockerfile`

```dockerfile
# Use official Ruby image (updated to a valid version)
FROM ruby:3.2.6-slim

# Install dependencies
RUN apt-get update -qq && apt-get install -y \
    build-essential \
    libpq-dev \
    nodejs \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /siv_rails_app_dockerdemo

# Install gems
COPY Gemfile Gemfile.lock ./
RUN bundle install

# Copy application code
COPY . .

# Add a script to be executed every time the container starts
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]

# Expose port
EXPOSE 3001

# Start the main process
CMD ["rails", "server", "-b", "0.0.0.0"]
```

#### 2. `entrypoint.sh`

```bash
#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails
rm -f /siv_rails_app_dockerdemo/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile)
exec "$@"
```

#### 3. `docker-compose.yml`

```yaml
  version: "3.8"

  services:
    db:
      image: postgres:15
      environment:
        POSTGRES_PASSWORD: password
      volumes:
        - postgres_data:/var/lib/postgresql/data
      ports:
        - "5432:5432"

    web:
      build: .
      command: bash -c "rm -f tmp/pids/server.pid && rails server -b 0.0.0.0"
      volumes:
        - .:/siv_rails_app_dockerdemo
      ports:
        - "3001:3000"
      depends_on:
        - db
      environment:
        DATABASE_URL: postgres://postgres:password@db:5432/dockerrails_development

  volumes:
    postgres_data:
```


### Step 3: Run the App

```bash
# Build containers
docker-compose build

# Start services
docker-compose up
```

Access: `http://localhost:3001`

#### Initialize the Database

```bash
docker-compose exec web rails db:create
docker-compose exec web rails db:migrate
```

## 2. Deploying to Docker Hub

### 1. Build & Tag the Image

```bash
docker build -t shivrajbadu/docker_rails_demo:1.0 .
```

### 2. Push to Docker Hub

```bash
docker login
docker push shivrajbadu/docker_rails_demo:1.0
```

## 3. Running the App on Another Machine

### 1. Create a New Directory

```bash
mkdir ~/docker_rails_deploy && cd ~/docker_rails_deploy
```

### 2. Add `docker-compose.yml`

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  web:
    image: shivrajbadu/docker_rails_demo:1.0
    command: bash -c "rm -f tmp/pids/server.pid && rails server -b 0.0.0.0"
    depends_on:
      - db
    environment:
      DATABASE_URL: postgres://postgres:password@db:5432/docker_rails_demo_production
      RAILS_ENV: production
    ports:
      - "3002:3000"

volumes:
  postgres_data:
```

### 3. Start the App

```bash
docker-compose up
```

Access: `http://localhost:3002`

#### Initialize DB (If Needed)

```bash
docker-compose exec web rails db:create db:migrate
```

## Key Docker Commands Cheat Sheet

| Command | Description |
|---------|-------------|
|`docker-compose --build `| Build dependencies |
| `docker-compose up` | Start containers. Run the services and processes. |
| `docker-compose down` | Stop containers |
| `docker-compose logs web` | View Rails logs |
| `docker-compose exec web bash` | Enter container shell |
| `docker-compose exec web rails c` | Open Rails console |
| `docker ps` | List running containers, processes |
| `docker push <image>` | Push to Docker Hub |
| `docker container ls` | to see container list |
| `docker-compose stop` | to stop all the processes for later restart |
| `docker-compose run website rake db:migrate db:seed RAILS_ENV=production` | website means name of service to run rails app |
| `docker-compose run --rm website rake db:create db:migrate` | website is name of service |
| `sudo docker run -i -t <image/id> /bin/bash` | enter into bash shell |
|`docker rmi 24a77bfbb9ee -f`|forcefully remove image|
|`docker rm $(docker ps -a -q)`| remove all the containers |

---

# Note: If you don’t have Docker running on your local machine, you need to replace localhost in the above URL with the IP address of the machine Docker is running on. If you’re using Docker Machine, you can run below cmd to find out the IP.

`docker-machine ip “${DOCKER_MACHINE_NAME}”`

{% include inarticle-adsense.html %}