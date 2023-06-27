---
title : Deploying SpringBoot toy projects with Heroku
date : 2023-04-10 01:56:00 +0900
categories : [Development, Spring]
tags : [backend, spring, springboot, server, heroku] #소문자만 가능
pinned : 0
image:
    path: /assets/img/posts/heroku_init_build.png
---
## Setup
Specify jar source in <b>Procfile</b>.

```
web: java -Dserver.port=$PORT $JAVA_OPTS -jar target/demo-0.0.1-SNAPSHOT.jar
```


## Deploy
Deployment automatically starts with Heroku CLI with the following command:

```
git add .
git commit "message"
git push heroku master
```

## Errors in initial build

### Invalid target release
```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.10.1:compile (default-compile) on project demo: Fatal error compiling: invalid target release: 17 -> [Help 1]
```

### Solution
Add matching java version to force the version in <b>system.properties</b> in root.
```
java.runtime.version=17
```

## Success
![Heroku Initial Build](/assets/img/posts/heroku_init_build.png)