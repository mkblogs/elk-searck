# elk stack using docker volume
 
In this example , we feed generated log files to elasticsearch by using docker volume. 

Server Code generate log files under target/logs folder 

```xml

 <appender name="FILE-ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>

        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/application.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <!-- each archived file, size max 10MB -->
            <maxFileSize>10MB</maxFileSize>
            <!-- total size of all archive files, if total size > 20GB, it will delete old archived file -->
            <totalSizeCap>2GB</totalSizeCap>
            <!-- 60 days to keep -->
            <maxHistory>60</maxHistory>
        </rollingPolicy>

        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

```
```docker
FROM maven:3.5.2-jdk-8-alpine AS MAVEN_TOOL
COPY pom.xml /tmp/
COPY src /tmp/src/
WORKDIR /tmp/
RUN mvn package
WORKDIR /tmp/target
CMD ["java","-jar","elk.jar"]

```
So log files are generated in /tmp/target/logs folder

So we need to move these log files to docker volume as below

```docker

 volumes: 
      - applogs:/tmp/target/logs      

```

We can pass these log files to logstash container as like below

```docker

volumes:
      - applogs:/home/logs  

```

In logstash.conf  we can get logs from these location and pass to elasticsearch

```
input {
   file {
    type => "java"
    path => "/home/logs/*.log"
    start_position => "beginning"
  }
}
 
output {
    elasticsearch {
        hosts => [ "elasticsearch:9200" ]
        index => "logfiles-%{+YYYY.MM.dd}"
    }
     stdout {
        codec => rubydebug
     }
}

```