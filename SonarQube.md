---
title: SonarQube
tags: [Notebooks/HowTo/Gradle]
created: '2019-03-23T15:35:51.435Z'
modified: '2019-06-11T04:18:25.364Z'
---

# SonarQube

## Configure sonarqube options in gradle configurations.
[GITHUB Example](https://github.com/SonarSource/sonar-scanning-examples/tree/master/sonarqube-scanner-gradle)

**settings.gradle**

```groovy
rootProject.name = 'myproject'
```

**build.gradle**

```groovy
plugins {
    id "org.sonarqube" version "2.6.2"
}

apply plugin: "jacoco"

sonarqube {
    properties {
        property 'sonar.projectKey', 'package:myproject'
        property 'sonar.projectName', 'myproject'  
        property 'sonar.coverage.exclusions', "**/dtos/**,**/test/**" // exclude dtos package and tests from coverage
    }
}
```

## Jenkinsfile example

```groovy
stage 'Sonar Analysis'

if (checkSonarqubePresent()) {
  withSonarQubeEnv('sonarqube') {
    ansiColor('xterm') {
      sh './gradlew sonarqube --info -x test -PignoreFailedTests=true -Dsonar.host.url=http://localhost:9900'
    }
  }
} 
else {
  echo "There is no sonarqube task configured in gradle.build"
}

def checkSonarqubePresent() {
  sonarqubeIsPresent = sh(script: './gradlew tasks --all | grep -o sonarqube | wc -l', returnStdout: true).trim()
  echo "Result of sonarqube GREP command: ${sonarqubeIsPresent}"
  return (!"0".equals(sonarqubeIsPresent))
}
```
