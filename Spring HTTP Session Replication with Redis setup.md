---
title: Spring HTTP Session Replication with Redis setup
tags: [WebServices]
created: '2019-03-23T15:25:16.072Z'
modified: '2019-03-27T05:21:27.264Z'
---

# Spring HTTP Session Replication with Redis setup

To use HTTP sessions in the Cloud they ___MUST___ not be stored `in-memory`.  All apps must be stateless.  
To use HTTP sessions we must store the data on remote storage.  The follow documentation
explains how to setup a spring boot war file to store session data in `REDIS`.

# Create the REDIS service

In __Cloud Foundry__ We can use the following to create a REDIS database.  Note the speical `-c` args that are needed
to correctly allow REDIS notifcation of session expiration.

```bash
cf create-service p.redis cache-small session_db -c '{"notify-keyspace-events":"Egx"}'
```

# Gradle libs needed
Update your `build.gradle` file with the following libs:

```groovy
    compile('org.springframework.session:spring-session-data-redis:2.0.4.RELEASE')
    compile('org.springframework.boot:spring-boot-starter-cloud-connectors')
    compile('org.springframework.cloud:spring-cloud-cloudfoundry-connector')
    compile('org.springframework.cloud:spring-cloud-spring-service-connector')
    compile('redis.clients:jedis:2.9.0')
```

__Note:__ _That you can replace jedis with the following `lettuce` REDIS client as well_
```groovy
   compile group: 'io.lettuce', name: 'lettuce-core', version: '5.0.4.RELEASE'
   compile group: 'org.apache.commons', name: 'commons-pool2', version: '2.6.0'
```

# Add the following to your application yml/property files:

## application-default.yml
In order to allow an application to run locally (on developer workstation), we don't
want to use REDIS, so we disable it in the `application-default.yml` file.

```yaml
spring:
  session:
    store-type: NONE
```

## application-dev.yml

Set the session storage to REDIS and the HTTP session timeout to 70 minutes:

```yaml
spring:
  session:
    store-type: REDIS
server:
  servlet:
    session:
      timeout: 4200    
```

# Add configuration class to your code

Add the following configuration class to your spring boot application:

```java
package com.corelogic.xxxxx;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.session.data.redis.config.ConfigureRedisAction;
import org.springframework.session.web.http.CookieSerializer;
import org.springframework.session.web.http.DefaultCookieSerializer;

@Configuration
public class SessionConfig {

    /**
     * Need to tell spring not to try to auto-configure REDIS "CONFIG"
     * as it doesn't work in PCF.
     *
     * @return
     */
    @Bean
    public static ConfigureRedisAction configureRedisAction() {
        return ConfigureRedisAction.NO_OP;
    }

    /**
     * By default spring session created a cookie named "SESSION" and
     * to get PCF session affinity we need it called "JESSIONID".
     *
     * @return
     */
    @Bean
    public CookieSerializer cookieSerializer() {
        DefaultCookieSerializer serializer = new DefaultCookieSerializer();
        serializer.setCookieName("JSESSIONID");
        serializer.setCookiePath("/");
        serializer.setDomainNamePattern("^.+?\\.(\\w+\\.[a-z]+)$");
        return serializer;
    }

    /**
     * Instead of using Java JVM native binary serialization, we would like to
     * use JSON instead to store the data in REDIS.
     *
     * @return
     */
    @Bean(name = {"defaultRedisSerializer","springSessionDefaultRedisSerializer"})
    RedisSerializer<Object> defaultRedisSerializer(){
        return new GenericJackson2JsonRedisSerializer();
    }

}
```

# Session creation / destruction notification

If you are intrested in when a session is created and/or destroyed, you can use the following example class.  Since notifications are sent to **ALL** app instances for each session creation/destruction, the class below provides an example how to perform an action on just one of the app instances per session id.

```java
package com.corelogic.service.session;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import javax.servlet.annotation.WebListener;
import javax.servlet.http.HttpSessionBindingEvent;
import javax.servlet.http.HttpSessionBindingListener;
import javax.servlet.http.HttpSessionEvent;
import javax.servlet.http.HttpSessionListener;
import java.util.concurrent.TimeUnit;

@Configuration
@WebListener
@Profile("cloud")
public class SessionExpireNotify implements HttpSessionBindingListener,
        HttpSessionListener {

    private static final Logger LOG = LoggerFactory.getLogger(SessionExpireNotify.class);

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public StringRedisTemplate redisTemplate() {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        RedisSerializer<String> stringSerializer = new StringRedisSerializer();
        stringRedisTemplate.setKeySerializer(stringSerializer);
        stringRedisTemplate.setValueSerializer(stringSerializer);
        stringRedisTemplate.setHashKeySerializer(stringSerializer);
        stringRedisTemplate.setHashValueSerializer(stringSerializer);
        stringRedisTemplate.setConnectionFactory(redisConnectionFactory);
        return stringRedisTemplate;
    }

    @Autowired
    RedisTemplate redisTemplate;

    @Override
    public void valueBound(HttpSessionBindingEvent event) {
        LOG.info("session valueBound event:"+event);
    }

    @Override
    public void valueUnbound(HttpSessionBindingEvent event) {
        LOG.info("session valueUnbound event:"+event);
    }

    @Override
    public void sessionCreated(HttpSessionEvent se) {
        LOG.info("session sessionCreated event:"+se);
    }

    @Override
    public void sessionDestroyed(HttpSessionEvent se) {
        String sessionId = se.getSession().getId();
        LOG.info("session sessionDestroyed event:"+se);
        String destroySyncKey = "destroy-sync:"+sessionId;
        boolean firstToDestroySession = redisTemplate.opsForValue().setIfAbsent(destroySyncKey, "");
        if (firstToDestroySession) {
            redisTemplate.opsForValue().getOperations().expire(destroySyncKey, 1, TimeUnit.MINUTES);
            LOG.info("sessionDestroyed - perform a session action just ONCE here for sessionId:"+destroySyncKey);
        } else {
            LOG.info("sessionDestroyed - Someone else beat me to it -- don't do anything for sessionId:"+destroySyncKey);
        }
    }
}
```

# Debugging 

Access REDIS to verify that sessions are getting stored correctly.

## Get REDIS configuration

Get the `host IP`, `port` and `password` from the `VCAP_SERVICES` environment variable of your application (after the sessions_db service has been bound to the app).

```bash
cf env MY_APP_NAME
```

## Setup ssh tunnel

Using the REDIS information above, create a PCF ssh tunnel.  The `port` below is the REDIS port and the IP is the REDIS IP addresss.

```bash
cf ssh -N -T -L 6379:10.215.117.163:6379 MY_APP_NAME
```

## Use the REDIS CLI
Use the `redis-cli` to access REDIS.  NOTE: if you do not have `redis-cli`, install using `brew`.
```bash
brew tap ringohub/redis-cli
brew install redis-cli
```
### Start the REDIS CLI

The host IP will alway be localhost (127.0.0.1) as we're using the ssh tunnel.  The port will be the tunneled port which we matched with the REDIS port, and the value of the `-a` is the REDIS password.

```bash
redis-cli -h 127.0.0.1 -p 6379 -a "YivKbNdssBWmn8XxIRslSGdV1Q="
```

### Useful commands

To list all the values in REDIS (all the sessions that are stored):
```bash
keys *
```

To get the values stored in a specific HTTP session.  The key name comes from one of the rows returned by `keys *`
```bash
hgetall spring:session:sessions:848ce844-ef43-4f07-87f3-6be568906c20
```

A session will be removed from REDIS storage when its TTL (time to live) expires.  To see the TTL value:
```bash
ttl spring:session:sessions:848ce844-ef43-4f07-87f3-6be568906c20
```

To clean (delete) all stored sessions:
```bash
flushdb
```

## ClassCastException error

All objects stored in the HTTP session need to be serialized.  In the above example setup we are using JSON to serialize.  This means that the Jackson serializer must be able to handle all these objects.  In some cases where there is an issue deserializing (e.g., when the object has been stored and it now being retrieved) the following error can occur:
```
java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to com.xxxx.SomeOtherObject
```
The above is normally caused by the absence of a public zero argument constructor on the target object.
