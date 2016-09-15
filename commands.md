
# 0. Preparation

## Pull images

    docker pull java:8
    docker pull maven:3
    docker pull oraclelinux:7
    docker pull tomcat:8
    docker pull mysql:5
    docker pull elasticsearch:2.4
    docker pull kibana:4.6.1
    docker pull logstash:2.4
    docker pull jenkins:2.7.4

## Save images

    docker save java:8 | gzip > java_8.tar.gz
    docker save maven:3 | gzip > maven_3.tar.gz
    docker save oraclelinux:7 | gzip > oraclelinux_3.tar.gz
    docker save tomcat:8 | gzip > tomcat_8.tar.gz
    docker save mysql:5 | gzip > mysql_5.tar.gz
    docker save elasticsearch:2.4 | gzip > elasticsearch_2_4.tar.gz
    docker save kibana:4.6.1 | gzip > kibana_4_6_1.tar.gz
    docker save logstash:2.4 | gzip > logstash_2_4.tar.gz
    docker save jenkins:2.7.4 | gzip > jenkins_2_7_4.tar.gz

## Load images

    docker load -i java_8.tar.gz
    docker load -i maven_3.tar.gz
    docker load -i oraclelinux_3.tar.gz
    docker load -i tomcat_8.tar.gz
    docker load -i mysql_5.tar.gz
    docker load -i elasticsearch_2_4.tar.gz
    docker load -i kibana_4_6_1.tar.gz
    docker load -i logstash_2_4.tar.gz
    docker load -i jenkins_2_7_4.tar.gz

# 1. Java in container

Pull java image from Docker Hub:

    docker pull java:8

Check java version:

    docker run java:8 /usr/bin/java -version
    docker run java:8 /usr/bin/javac -version

List docker containers:

    docker ps -a

Remove containers:

    docker kill $(docker ps -q)
    docker rm $(docker ps -a -q)

Use throw-away containers:

    docker run --rm java:8 /usr/bin/java -version

Or create named container and use that with `exec`:

    docker run --name=myjava -dit java:8 /bin/sh
    docker exec myjava /usr/bin/java -version

Or run shell in container and use container's environment directly:

    docker run --name=myjava -it java:8 /bin/bash -l
    root@16fdd31f7ba2:/# java -version

Let's use 2nd way and add source files and compile files them:

    docker cp HelloWorld.java myjava:/
    docker exec /usr/bin/javac HelloWorld.java
    docker exec myjava /usr/bin/java HelloWorld

Create new image out of our container:

    docker commit myjava myjava:1.0

Run container based on new image:

    docker run --rm myjava:1.0 /usr/bin/java HelloWorld

Compile java code on the host (do not explicitly copy sources into container and do not install java on the host):

    docker run --rm -v`pwd`:/src java:8 ls /src
    docker run --rm -v`pwd`:/src java:8 /usr/bin/javac /src/HelloWorld.java
    docker run --rm -v`pwd`:/src -w /src java:8 /usr/bin/javac HelloWorld.java

Add some aliases:

    alias dj="docker run --rm -v`pwd`:/src -w /src java:8 /usr/bin/java"
    alias djc="docker run --rm -v`pwd`:/src -w /src java:8 /usr/bin/javac"

# 2. Building KnockKnock

Compile code inside an image:

    docker build --tag=knock:1.0 .

Start container from new image:

    docker run --name=knock -dit -p 4444:4444 knock:1.0

To talk to the server:

    telnet 127.0.0.1 4444

# 3. Running PetClinic (as-is)

Get the source code:

    git clone https://github.com/spring-projects/spring-petclinic

Build image:

    docker build --tag=petclinic:1.0 .

Run container:

    docker run -p 9966:9966 --name=petclinic -dit petclinic:1.0

Follow logs:
   
    docker logs -f petclinic

Access the web application:

    open http://192.168.123.45:9966/petclinic

## 3.1. Splitting PetClinic build 

Build PetClinic without installing maven:

    docker run --rm -v `pwd`/spring-petclinic:/app -w /app maven:3 mvn -e clean install -DskipTests

Build PetClinic by preserving libraries:

    docker run --rm -v `pwd`/.m2:/root/.m2 -v `pwd`/spring-petclinic:/app -w /app maven:3 mvn -e clean install -DskipTests

Use volume as maven cache:

    docker volume create --name=m2cache

Use shared volume to build maven projects:

    docker run --rm -v m2cache:/root/.m2 -v `pwd`/spring-petclinic:/app -w /app maven:3 mvn -e clean install -DskipTests

Find volume location:

    docker volume inspect m2cache

Add alias:

    alias dmvn="docker run --rm -v m2cache:/root/.m2 -v `pwd`:/app -w /app maven:3 mvn"

Run maven as container:

    dmvn clean install -e clean install -DskipTests

# 5. Running in Tomcat

Run tomcat container:

    docker run -dit -v `pwd`:spring-petclinic/target/petclinic.war:/usr/local/tomcat/webapps/petclinic.war -p 8080:8080 --name=petclinic-tomcat tomcat:8

Build image with PetClinic WAR file: 

    docker build --tag=petclinic-tomcat:1.0 .

Run new container:

    docker run -dit -p 8080:8080 --name=petclinic-tomcat petclinic-tomcat:1.0

# 6. Running in WebLogic

Get Oracle docker files:

    git clone https://github.com/oracle/docker-images.git oracle-docker-images

Dowload Oracle Java JRE 8 u101 and place it under:

    oracle-docker-images/OracleJDK/java-8/server-jre-8u101-linux-x64.tar.gz

Build Docker image with Oracle Java:
 
    cd oracle-docker-images/OracleJDK/java-8
    docker build -t oracle/jdk:8 . 

Test java version:

    docker run --rm oracle/jdk:8 java -version

Download Oracle WebLogic 12.2.1 and place it under:

    oracle-docker-images\OracleWebLogic\dockerfiles\12.2.1\fmw_12.2.1.0.0_wls_Disk1_1of1.zip

Build Docker image with Oracle WebLogic:

    cd oracle-docker-images/OracleWebLogic/dockerfiles/12.2.1
    docker build -t oracle/weblogic:12.2.1-generic -f Dockerfile.generic .

Build Docker image with WebLogic domain:

    cd oracle-docker-images/OracleWebLogic/samples/1221-domain

Build Docker image with PetClinic deployed to that:

    cd oracle-docker-images/OracleWebLogic/samples/1221-appdeploy

# 7. Adding database layer

# 8. Adding logging layer

Build ELK stack:

    git clone https://github.com/aestasit/docker-elk
    cd docker-elk
    docker-compose up -d 

