---
title: Docker fix for my enviroment
layout: post
date: '2023-11-24'
last_modified_at: '2023-11-24'
categories:
  - guide
tags:
  - exceptions
  - exception
  - java
  - programming
  - coding
---
This is just a quick note. I've been changing my working environment a lot lately and making this fix for docker. It is needed to avoid the following problem when running tests for Spring application on M* series processors

```asm
Caused by: java.lang.IllegalStateException: Could not find a valid Docker environment. Please see logs and check configuration
at org.testcontainers.dockerclient.DockerClientProviderStrategy.lambda$getFirstValidStrategy$3(DockerClientProviderStrategy.java:158)
at java.util.Optional.orElseThrow(Optional.java:290)
at org.testcontainers.dockerclient.DockerClientProviderStrategy.getFirstValidStrategy(DockerClientProviderStrategy.java:150)
at org.testcontainers.DockerClientFactory.client(DockerClientFactory.java:111)
at org.testcontainers.containers.GenericContainer.<init>(GenericContainer.java:175)
at org.testcontainers.containers.JdbcDatabaseContainer.<init>(JdbcDatabaseContainer.java:36)
at org.testcontainers.containers.PostgreSQLContainer.<init>(PostgreSQLContainer.java:32)
at com.mastercard.example.testcontainers.testcontainersexampple.DemoControllerTest.<clinit>(DemoControllerTest.java:27)
```

Fix is
```asm
sudo ln -s $HOME/.docker/run/docker.sock /var/run/docker.sock
```
Thanks!