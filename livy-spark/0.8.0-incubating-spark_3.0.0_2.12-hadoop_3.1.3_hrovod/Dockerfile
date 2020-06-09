#ARG SPARK_BASE=cyliu0204/spark:2.4.5_2.11-hadoop_3.1.0-horovod_1.8_cloud
ARG SPARK_BASE=cyliu0204/spark:3.0.0_2.12-hadoop_3.1.3-horovod_1.9.2_tf1.4.0_cloud
ARG LIVY_BUILDER=sasnouskikh/livy-builder:0.2

### Livy Builder Container
FROM $LIVY_BUILDER as build

ARG LIVY_GITHUB_REPO=cyliu0204/incubator-livy

ARG LIVY_GITHUB_BRANCH=spark3.0-kubernetes

RUN echo "test2"
RUN git clone -b ${LIVY_GITHUB_BRANCH} https://github.com/${LIVY_GITHUB_REPO}.git && \
    cd incubator-livy && \
    mvn clean package -DskipTests -Pthriftserver -Pspark-3.0 

RUN ls incubator-livy/assembly/target/
RUN  cp incubator-livy/assembly/target/apache-livy-0.8.0-incubating-SNAPSHOT-bin-2.12.zip /

### Final Container
FROM $SPARK_BASE

ARG LIVY_VERSION_ARG=0.8.0-incubating-SNAPSHOT

LABEL maintainer="Aliaksandr Sasnouskikh <jaahstreetlove@gmail.com>"

ENV BASE_IMAGE              $SPARK_BASE#$BASE_IMAGE

ENV LIVY_VERSION            $LIVY_VERSION_ARG
ENV LIVY_HOME               /opt/livy
ENV LIVY_CONF_DIR           $LIVY_HOME/conf

ENV PATH                    $PATH:$LIVY_HOME/bin

# install livy
COPY --from=build /apache-livy-${LIVY_VERSION}-bin-2.12.zip /apache-livy-${LIVY_VERSION}-bin.zip
RUN unzip /apache-livy-${LIVY_VERSION}-bin.zip -d / && \
    mv /apache-livy-${LIVY_VERSION}-bin-2.12 /opt/ && \
    rm -rf $LIVY_HOME && \
    ln -s /opt/apache-livy-0.8.0-incubating-SNAPSHOT-bin-2.12  $LIVY_HOME && \
    rm -f /apache-livy-${LIVY_VERSION}-bin.zip


RUN ls $LIVY_CONF_DIR/

#RUN rm $SPARK_HOME/jars/kubernetes-client-4.1.2.jar

#ADD https://repo1.maven.org/maven2/io/fabric8/kubernetes-client/4.4.2/kubernetes-client-4.4.2.jar $SPARK_HOME/jars
RUN wget -O $SPARK_HOME/jars/aws-java-sdk-1.11.271.jar https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/1.11.271/aws-java-sdk-bundle-1.11.271.jar  && \
wget -O $SPARK_HOME/jars/hadoop-aws-3.1.3.jar  https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.1.3/hadoop-aws-3.1.3.jar 

#RUN git clone  https://github.com/delta-io/delta.git && \
#    cd delta && \
#    build/sbt  package &&\
#    copy target/scala-2.12/delta-core_2.12-0.6.1-SNAPSHOT.jar $SPARK_HOME/jars/

COPY delta-core_2.12-0.6.1-SNAPSHOT.jar $SPARK_HOME/jars/

RUN rm -rf $SPARK_HOME/jars/guava-14.0.1.jar

COPY alluxio-2.2.0-client.jar  $SPARK_HOME/jars/

#ENV LIVY_SPARK_JAVA_OPT_LOGINNAME -Dalluxio.security.login.username=alluxio
#RUN conda  install -y -q horovod
#ENV LIVY_SPARK_JAVA_OPT_TEST -Dalluxio.security.login.username=hdfs -Dalluxio.security.login.impersonation.username=NONE -Dalluxio.security.underfs.hdfs.impersonation.enabled=false

COPY core-site.xml $SPARK_HOME/conf/

COPY Dockerfile /my_docker/

RUN set -ex && \
    apt-get update && \
    ln -s /lib /lib64 && \
    apt install -y bash libc6 libpam-modules libnss3 wget bzip2 && \
    rm /bin/sh && \
    ln -sv /bin/bash /bin/sh && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    chgrp root /etc/passwd && chmod ug+rw /etc/passwd && \
    rm -rf /var/cache/apt/*


ENV TINI_VERSION v0.19.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /usr/bin/tini
RUN chmod +x /usr/bin/tini

#RUN conda install -y  pytorch  torchvision -c pytorch
#RUN conda install -y tensorflow-gpu=1.13* tensorboard keras=2.2*

#RUN sudo apt-get install build-essential gcc

#RUN pip install horovod


RUN mkdir ~/.gnupg
RUN echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf

RUN \
  apt-get -y update && \
  apt-get -y install software-properties-common && \
  apt-get -y install --reinstall locales && \
  rm -rf /root/.cache && \
  apt-get purge -y $(apt-cache search '~c' | awk '{ print $2 }') && \
  apt-get -y autoremove && \
  apt-get -y autoclean && \
  apt-get -y clean all && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/apt && \
  rm -rf /tmp/*

RUN locale-gen en_US.UTF-8

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8

ADD jdk-8u251-linux-x64.tar.gz /usr/local/share/
RUN ls /usr/local/share/
#RUN tar -xzvf jdk-8u251-linux-x64.tar.gz -C /usr/local/share/jdk1.8.0_251
#RUN rm ./*.tar.gz
ENV JAVA_HOME=/usr/local/share/jdk1.8.0_251
ENV PATH="$JAVA_HOME/bin:$PATH"
#RUN conda install -y pyarrow=0.16.0
#RUN pip install -U pyarrow==0.16.0

RUN pip install horovod[spark]
RUN pip install -U keras==2.2.5


ENV HADOOP_HOME=/opt/hadoop-3.1.3/

ADD hadoop-3.1.3.tar.gz /opt/
RUN cp -rf /opt/hadoop-3.1.3/share/hadoop/common/lib/guava-27.0-jre.jar $SPARK_HOME/jars/

RUN cp -rf $HADOOP_HOME/lib/native/* $HADOOP_HOME/ 
RUN ls $LIVY_CONF_DIR/

#  4040 - Spark UI port
#  7078 - Driver RPC port
#  7079 - Blockmanager port
#  8088 - JMX Exporter for Prometheus
# 10000 - Livy RPC Server for Jupyter integration
EXPOSE 4040 7078 7079 8088 10000
