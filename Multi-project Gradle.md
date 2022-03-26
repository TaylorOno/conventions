---
tags: [Notebooks/HowTo/Gradle]
title: Multi-project Gradle
created: '2019-05-16T04:43:33.048Z'
modified: '2022-03-26T14:11:32.041Z'
---

# Multi-project Gradle
Sometimes it makes sense to have multiple gradle projects. This applys to client server scenarios and can also help us seperate the web logic from service logic in the case of services.


## Top Level Settings
### settings.gradle
This holds information on your project structure related to module layout as well as the locations of various modules or subprojects.

_example:_
```groovy
rootProject.name = '${rootProject-Name}'
include ':${moduleFolder}'
include ':${moduleFolder}:${module-1}'
include ':${moduleFolder}:${module-2}'

project(':${moduleFolder}').projectDir = "$rootDir/modules" as File
project(':${moduleFolder}:${module-1}').projectDir = "$rootDir/${moduleFolder}/${folder}" as File
project(':${moduleFolder}:${module-2}').projectDir = "$rootDir/${moduleFolder}/${folder}" as File
```

### gradle.gradle
You can put items here that apply gobally to your project such as plugins, settings, and build script configuration.

_example:_
```groovy
allprojects{
	global settings
}

subprojects{
	subproject settings
}

buildscript {
  global build script settings
}
```
## Sub Module/Project Settings
