FROM java:8

ADD spring-petclinic /app

ENV M2_HOME /app

EXPOSE 9966

WORKDIR /app

RUN chmod a+x /app/mvnw

RUN ls /app/mvnw

RUN /bin/bash -c '/app/mvnw -e clean install tomcat7:help -DskipTests'

CMD /app/mvnw -e tomcat7:run -DskipTests
