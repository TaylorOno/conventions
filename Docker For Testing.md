---
title: Docker For Testing
tags: [Notebooks/HowTo/Docker, Notebooks/HowTo/Gradle]
created: '2019-03-23T15:52:27.544Z'
modified: '2019-06-11T04:18:10.556Z'
---

# Docker For Testing
## Gradle Tasks
In order to use docker for external resources during gradle test you first you will need to add the appropriate plugins and tasks to the build.gradle file.  You will need to include the `docker` plugin and the `docker-compose` plugin.  In addition you will need to setup the docker compose configurations you can find more details here: [docker-compose plugin Documentation] (https://github.com/avast/gradle-docker-compose-plugin).  Finally you will need to add make dockerCompose.isRequiredBy(test) to make sure that the docker compose plugin gets executed before any tests run.  

See sample build.gradle file below:  

**build.gradle**

```groovy
buildscript {
    ext {
        dockerComposePluginVersion = '0.8.2'
        dockerPluginVersion = '1.2'
    }
    repositories {
        maven {
            url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath("com.avast.gradle:gradle-docker-compose-plugin:$dockerComposePluginVersion")
        classpath("se.transmode.gradle:gradle-docker:${dockerPluginVersion}")
    }
}

apply plugin: 'docker-compose'
apply plugin: 'docker'

dockerCompose {
    dockerComposeWorkingDirectory = 'docker/'  //Path relative to gradlew script
    useComposeFiles = ['docker-compose.yml']   //Name of docker compose file
    removeContainers = true
    captureContainersOutput = true
}

dockerCompose.isRequiredBy(test)
```
## Docker Compose File
This file tells docker what resources to start  addtional information can be found here: [Docker Compose Documentation](https://docs.docker.com/compose/).    

See sample build.gradle file below:  

**docker-compose.yml**
located at relative path `docker/docker-compose.yml`
```yaml
version: '3.0'                                                  #DockerCompose Scipt Version
services:
  elastic-search-6.5.3:                                         #Service Name
    image: docker.elastic.co/elasticsearch/elasticsearch:6.5.3  #Docker Image
    environment:                                                #Image Specific Parameters
      - cluster.name=mynewclustername
      - discovery.type=single-node
    ports:                                                      #Ports to Expose
      - 9200:9200
      - 9300:9300
```
