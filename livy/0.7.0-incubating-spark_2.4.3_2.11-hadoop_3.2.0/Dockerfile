ARG SPARK_BASE=sasnouskikh/livy-spark:0.7.0-incubating-spark_2.4.3_2.11-hadoop_3.2.0

FROM $SPARK_BASE

LABEL maintainer="Aliaksandr Sasnouskikh <jaahstreetlove@gmail.com>"

ENV BASE_IMAGE              $SPARK_BASE#$BASE_IMAGE

RUN cp $LIVY_CONF_DIR/log4j.properties.template $LIVY_CONF_DIR/log4j.properties && \
    mkdir -p $LIVY_HOME/work-dir && \
    mkdir /var/log/livy && \
    ln -s /var/log/livy $LIVY_HOME/logs

COPY conf/* $LIVY_CONF_DIR/
COPY entrypoint.sh /opt/
COPY Dockerfile /my_docker/

RUN ls $SPARK_HOME/jars/|grep kubernetes

RUN rm $SPARK_HOME/jars/kubernetes-client-4.1.2.jar
ADD https://repo1.maven.org/maven2/io/fabric8/kubernetes-client/4.4.2/kubernetes-client-4.4.2.jar $SPARK_HOME/jars

RUN wget -O $SPARK_HOME/jars/aws-java-sdk-1.11.777.jar https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.11.777/aws-java-sdk-1.11.777.jar && \
wget -O $SPARK_HOME/jars/hadoop-aws-3.1.0.jar  https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.1.0/hadoop-aws-3.1.0.jar
COPY client.jar $SPARK_HOME/jars/

COPY conf/core-site.xml $SPARK_HOME/conf/
 
#  8998 - Livy Server port
# 10000 - Livy RPC Server for Jupyter integration
EXPOSE 8998 10000

WORKDIR $LIVY_HOME/work-dir

ENTRYPOINT [ "/opt/entrypoint.sh" ]
