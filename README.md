[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-ruby/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-ruby/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/ruby)](https://hub.docker.com/r/bitnami/ruby/)

# Supported tags and respective `Dockerfile` links

 - [`2.4`](https://github.com/bitnami/bitnami-docker-ruby/blob/ccdf5081431b4b762d33a100717cada808c1d6ca/2.4/Dockerfile) [`2.4.0-r1`](https://github.com/bitnami/bitnami-docker-ruby/blob/ccdf5081431b4b762d33a100717cada808c1d6ca/2.4/Dockerfile) [`latest`](https://github.com/bitnami/bitnami-docker-ruby/blob/ccdf5081431b4b762d33a100717cada808c1d6ca/2.4/Dockerfile)
 - [`2.3`](https://github.com/bitnami/bitnami-docker-ruby/blob/c488c2a9ff54e1cce7e5aea59a8abdd9adc99c61/2.3/Dockerfile) [`2.3.3-r0`](https://github.com/bitnami/bitnami-docker-ruby/blob/c488c2a9ff54e1cce7e5aea59a8abdd9adc99c61/2.3/Dockerfile)
 - [`2.2`](https://github.com/bitnami/bitnami-docker-ruby/blob/262c37150d9cb93fe85d81877942e3d26aa64b00/2.2/Dockerfile) [`2.2.6-r0`](https://github.com/bitnami/bitnami-docker-ruby/blob/262c37150d9cb93fe85d81877942e3d26aa64b00/2.2/Dockerfile)
 - [`2.1`](https://github.com/bitnami/bitnami-docker-ruby/blob/ca242044d5436a5b8f75e23613f39cd8fd65e308/2.1/Dockerfile) [`2.1.10-r0`](https://github.com/bitnami/bitnami-docker-ruby/blob/ca242044d5436a5b8f75e23613f39cd8fd65e308/2.1/Dockerfile)

Subscribe to project updates by watching the [bitnami/ruby GitHub repo](https://github.com/bitnami/bitnami-docker-ruby).

# What is Ruby?

> Ruby is a dynamic, open source programming language with a focus on simplicity and productivity. It has an elegant syntax that is natural to read and easy to write.

[ruby-lang.org](https://www.ruby-lang.org/en/)

# TLDR

```bash
docker run -it --name ruby bitnami/ruby:latest
```

## Docker Compose

```
ruby:
  image: bitnami/ruby:latest
  command: ruby script.rb
  volumes:
    - .:/app
```

# Get this image

The recommended way to get the Bitnami Ruby Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/ruby).

```bash
docker pull bitnami/ruby:latest
```

To use a specific version, you can pull a versioned tag. You can view the [list of available versions](https://hub.docker.com/r/bitnami/ruby/tags/) in the Docker Hub Registry.

```bash
docker pull bitnami/ruby:[TAG]
```

If you wish, you can also build the image yourself.

```bash
docker build -t bitnami/ruby:latest https://github.com/bitnami/bitnami-docker-ruby.git
```

# Entering the REPL

By default, running this image will drop you into the Ruby REPL (`irb`), where you can interactively test and try things out in Ruby.

```bash
docker run -it --name ruby bitnami/ruby:latest
```

**Further Reading:**

  - [Ruby IRB Documentation](http://ruby-doc.org/stdlib-2.0.0/libdoc/irb/rdoc/IRB.html)

# Running your Ruby script

The default work directory for the Ruby image is `/app`. You can mount a folder from your host here that includes your Ruby script, and run it normally using the `ruby` command.

```bash
docker run -it --name ruby -v /path/to/app:/app bitnami/ruby:latest \
  ruby script.rb
```

# Running a Ruby app with gems

If your Ruby app has a `Gemfile` defining your app's dependencies and start script, you can install the dependencies before running your app.

```bash
docker run -it --name ruby -v /path/to/app:/app bitnami/ruby:latest \
  sh -c "bundle install && ruby script.rb"
```

or using Docker Compose:

```
ruby:
  image: bitnami/ruby:latest
  command: "sh -c 'bundle install && ruby script.rb'"
  volumes:
    - .:/app
```

**Further Reading:**

  - [rubygems.org](https://rubygems.org/)
  - [bundler.io](http://bundler.io/)

# Accessing a Ruby app running a web server

This image exposes port `3000` in the container, so you should ensure that your web server is binding to port `3000`, as well as listening on `0.0.0.0` to accept remote connections from your host.

Below is an example of a [Sinatra](http://www.sinatrarb.com/) app listening to remote connections on port `3000`:

```
require 'sinatra'

set :bind, '0.0.0.0'
set :port, 3000

get '/hi' do
  "Hello World!"
end
```

To access your web server from your host machine you can ask Docker to map a random port on your host to port `3000` inside the container.

```bash
docker run -it --name ruby -P bitnami/ruby:latest
```

Run `docker port` to determine the random port Docker assigned.

```bash
$ docker port ruby
3000/tcp -> 0.0.0.0:32769
```

You can also manually specify the port you want forwarded from your host to the container.

```bash
docker run -it --name ruby -p 8080:3000 bitnami/ruby:latest
```

Access your web server in the browser by navigating to [http://localhost:8080](http://localhost:8080/).


# Connecting to other containers

If you want to connect to your Ruby web server inside another container, you can use docker networking to create a network and attach all the containers to that network.

## Serving your Ruby app through an nginx frontend

We may want to make our Ruby web server only accessible via an nginx web server. Doing so will allow us to setup more complex configuration, serve static assets using nginx, load balance to different Ruby instances, etc.

### Step 1: Create a network

```bash
docker network create app-tier --driver bridge
```

or using Docker Compose:

```yaml
version: '2'

networks:
  app-tier:
    driver: bridge
```

### Step 2: Create a virtual host

Let's create an nginx virtual host to reverse proxy to our Ruby container.

```
server {
    listen 0.0.0.0:80;
    server_name yourapp.com;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header HOST $http_host;
        proxy_set_header X-NginX-Proxy true;

        # proxy_pass http://[your_ruby_container_link_alias]:3000;
        proxy_pass http://myapp:3000;
        proxy_redirect off;
    }
}
```

Notice we've substituted the link alias name `myapp`, we will use the same name when creating the container.

Copy the virtual host above, saving the file somewhere on your host. We will mount it as a volume in our nginx container.

### Step 3: Run the Ruby image with a specific name

```
docker run -it --name myapp \
  --network app-tier \
  -v /path/to/app:/app \
  bitnami/ruby:latest ruby script.rb
```

or using Docker Compose:

```yaml
version: '2'
myapp:
  image: bitnami/ruby:latest
  command: ruby script.rb
  networks:
    - app-tier
  volumes:
    - .:/app
```

### Step 4: Run the nginx image

```bash
docker run -it \
  -v /path/to/vhost.conf:/bitnami/nginx/conf/vhosts/yourapp.conf \
  --network app-tier \
  bitnami/nginx:latest
```

or using Docker Compose:

```yaml
version: '2'
nginx:
  image: bitnami/nginx:latest
  networks:
    - app-tier
  volumes:
    - /path/to/vhost.conf:/bitnami/nginx/conf/vhosts/yourapp.conf
```

# Maintenance

## Upgrade this image

Bitnami provides up-to-date versions of Ruby, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container.

### Step 1: Get the updated image

```bash
docker pull bitnami/ruby:latest
```

or if you're using Docker Compose, update the value of the image property to `bitnami/ruby:latest`.

### Step 2: Remove the currently running container

```bash
docker rm -v ruby
```

or using Docker Compose:

```bash
docker-compose rm -v ruby
```

### Step 3: Run the new image

Re-create your container from the new image.

```bash
docker run --name ruby bitnami/ruby:latest
```

or using Docker Compose:

```bash
docker-compose start ruby
```

# Notable Changes

## 2.3.1-r0 (2016-05-11)
- Commands are now executed as the `root` user. Use the `--user` argument to switch to another user or change to the required user using `sudo` to launch applications. Alternatively, as of Docker 1.10 User Namespaces are supported by the docker daemon. Refer to the [daemon user namespace options](https://docs.docker.com/engine/reference/commandline/daemon/#daemon-user-namespace-options) for more details.

## 2.2.3-0-r02 (2015-09-30)

- `/app` directory no longer exported as a volume. This caused problems when building on top of the image, since changes in the volume were not persisted between RUN commands. To keep the previous behavior (so that you can mount the volume in another container), create the container with the `-v /app` option.

## 2.2.3-0-r01 (2015-08-26)

- Permissions fixed so `bitnami` user can install gems without needing `sudo`.

# Contributing

We'd love for you to contribute to this Docker image. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-ruby/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-ruby/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-ruby/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# License
Copyright (c) 2015-2016 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
