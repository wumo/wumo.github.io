---
title:  "Gradle Usage"
tags:
- gradle
---

Using Intellij to create Gradle project.
startup script for `build.gradle`:
```groovy
group 'lab.mars'
version '1.0-SNAPSHOT'

apply plugin: 'java'
apply plugin: 'maven'

sourceCompatibility = 1.8

repositories {
    maven {
        url "${nexusUrl}/content/groups/public"
    }
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "${nexusUrl}/content/repositories/releases") {
                authentication(userName: nexusUsername, password: nexusPassword)
            }
            snapshotRepository(url: "${nexusUrl}/content/repositories/snapshots") {
                authentication(userName: nexusUsername, password: nexusPassword)
            }
        }
    }
}

task wrapper(type: Wrapper, description: "create a gradlew") {
    gradleVersion = '3.3'
}
```

Create `gradle.properties` to store private/configuration variables:
```
nexusUrl=http://192.168.10.203:8081/nexus
nexusUsername=hhx
nexusPassword=password
```
To avoid download gradle, modify `./gradle/wrapper/gradle-wrapper.properties`:
```
distributionUrl=gradle-3.3-bin.zip
```
And add the `distributionUrl=gradle-3.3-bin.zip` to your source control system.

<!--more-->