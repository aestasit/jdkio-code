FROM java:8

ADD . /src

RUN /usr/bin/javac /src/*.java

WORKDIR /src

EXPOSE 4444

ENV CLASSPATH=/src

CMD /usr/bin/java KnockKnockServer
