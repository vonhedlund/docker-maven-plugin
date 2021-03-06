[![Build Status](https://travis-ci.org/alexec/docker-maven-plugin.svg?branch=master)](https://travis-ci.org/alexec/docker-maven-plugin)

Docker Maven Plugin
===
Project Goal:

* Make it easy to build an app on a container, test it and push it to a Docker repository, even if it relies on other containers (e.g. a database)
* Talk "Maven" rather than "Docker" (E.g. "package" rather than "build").
* Keep it simple.

Goals
---
* `clean` - delete all containers and images for the project (see know issues)
* `package` - builds the containers based on YAML configuration
* `start` - start the containers in order and ensures they are running
* `stop` - stop all running containers for the project
* `deploy` - push containers to Docker repository

Pre-requisites
---
Docker installed and Docker daemon running, see the docker [getting started guide](https://www.docker.io/gettingstarted/) for e.g. on a mac follow these [instructions](http://docs.docker.io/en/latest/installation/mac/).

Usage
---
The best example to look at is the [one from the tests](src/it/build-test-it) which creates a [Drop-Wizard](https://dropwizard.github.io/dropwizard/) app and builds three containers: __app__ (the dropwizard application) __data__ and __mysql__, and then runs an integration test against the deployed app. Et voila a packaged image!

Typically, you build your app, run your standard unit tests and package it as usual. Then, you build a container with your app deployed onto it, and run integration tests against it. If they pass, deploy your jar into the Maven repository, and optionally, your image into a Docker repository.

To use the plugin, you need to define a docker directory in `${basedir}/src/main` which will include a subdirectory for each container that you wish to deploy.

- `src/main/docker/` contains one folder per container for e.g. the mysql container would have a folder structure as follows:
    - mysql
        - `Dockerfile` a standard Docker file.
        - `conf.yml` configuration:
    - ...
        - `Dockerfile` a standard Docker file.
        - `conf.yml` configuration:

```yml
# additional data require to create the Docker image
packaging:
  # files to add to the build, usually used with ADD in the Dockerfile
  add:
    - target/example-${project.version}.jar
    - hello-world.yml
# optional list of port to expose on the host
ports:
  - 8080
# containers that this should be linked to, started before this one and stopped afterwards
links:
  - mysql
healthChecks:
  pings:
    - url: http://localhost:8080/health-check
      timeout: 60000
# tag to use for images
tag: alex.e.c/app:${project.artifactId}-${project.version}
 ```

Add the following to the `pom.xml` plugins section.

 ```pom.xml
            <plugin>
                <groupId>com.alexecollins.docker</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <configuration>
                    <!-- your installed version -->
                    <version>1.9</version>
                    <!-- used for push -->
                    <username>alexec</username>
                    <email>alex.e.c@gmail.com</email>
                    <!-- remove images created by Dockerfile -->
                    <removeIntermediateImages>true</removeIntermediateImages>
                    <!-- do/do not cache images (default true), disable to get the freshest images -->
                    <cache>true</cache>
                    <!-- change here if you are using another port/host, e.g. 4243 -->
                    <host>http://localhost:2375</host>
                </configuration>
            </plugin>
 ```

Create your `${basedir}/src/main/docker` directory and create a subfolder for your application container

     mkdir -p src/main/docker/app
 
Define your Dockerfile and conf.yml and place in ${basedir}/src/main/docker/app

 ```tree
    src/main/docker/app
    ├── Dockerfile
    └── conf.yml
 ```

You can now invoke functionality from the plugin, information on the plugin can be found by running the following command

     mvn docker:help

For e.g. to build containers from their `Dockerfile` and `conf.yml` files, run the following command

     mvn docker:package

Debugging
---

`docker:start` can be started manually like this:

    docker run -t -i -link example-project_mysql:example-project_mysql -P -p 8080:8080 alex.e.c/example-project-app:1.0-SNAPSHOT  

You can see the requests made to Docker using Wireshark. However, if you're using boot2docker, it'll be on a local loop-back interface. 

It can be useful to see what's running, and `watch` will help you:

    watch docker ps

You can tail the Docker logs, e.g.

```
boot2docker ssh
tail -f /var/log/docker.log
```

Contributing
---
Please do! I'd love to make all the changes that people requested, but time eh?
 
See [CONTRIB.md](CONTRIB.md).
 
