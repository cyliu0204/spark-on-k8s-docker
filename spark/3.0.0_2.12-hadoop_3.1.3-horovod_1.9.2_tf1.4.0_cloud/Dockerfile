### Builder Container
FROM sasnouskikh/livy-builder:0.2 as build

RUN cd / && \
    git clone https://github.com/apache/spark.git --branch v3.0.0-rc2 --single-branch && \
    cd /spark && \
    dev/make-distribution.sh \
        --name hadoop-3.1.3-cloud-scala-2.12 --pip --tgz -DskipTests \
        -Phadoop-3.1 \
        -Dhadoop.version=3.1.3 \
        -Pkubernetes \
        -Phive && \
    cp spark-3.0.0-bin-hadoop-3.1.3-cloud-scala-2.12.tgz /

### Final Container
FROM cyliu0204/horovod:0.19.3-tf1.14.0-torch1.4.0-mxnet1.6.0-py3.6_gpu
#FROM openjdk:8-jdk-slim
#FROM $BASE_IMAGE
LABEL maintainer="Aliaksandr Sasnouskikh <jaahstreetlove@gmail.com>"

ENV BASE_IMAGE cyliu0204/horovod:0.19.3-tf1.14.0-torch1.4.0-mxnet1.6.0-py3.6_gpu


ENV SPARK_VERSION   3.0.0
ENV HADOOP_VERSION  hadoop-3.1.3-cloud
ENV SCALA_VERSION   2.12

ENV SPARK_HOME      /opt/spark
ENV SPARK_CONF_DIR  $SPARK_HOME/conf
ENV SPARK_CLASSPATH $SPARK_HOME/cluster-conf

ENV PYTHONHASHSEED  0
ENV CONDA_DIR       /opt/conda
ENV SHELL           /bin/bash

ENV PATH            $PATH:$SPARK_HOME/bin:$CONDA_DIR/bin

ARG MINICONDA_VERSION=4.6.14
ARG CONDA_VERSION=4.6.14

### install spark
COPY --from=build /spark-${SPARK_VERSION}-bin-${HADOOP_VERSION}-scala-${SCALA_VERSION}.tgz /
RUN tar -xzf /spark-${SPARK_VERSION}-bin-${HADOOP_VERSION}-scala-${SCALA_VERSION}.tgz -C /opt/ && \
    ln -s /opt/spark-${SPARK_VERSION}-bin-${HADOOP_VERSION}-scala-${SCALA_VERSION} $SPARK_HOME && \
    rm -f /spark-${SPARK_VERSION}-bin-${HADOOP_VERSION}-scala-${SCALA_VERSION}.tgz && \
    mkdir -p $SPARK_HOME/work-dir && \
    mkdir -p $SPARK_HOME/spark-warehouse && \
    mkdir -p $SPARK_HOME/cluster-conf

RUN apt-get update && \
    apt-get install -y -q vim wget zip bzip2

# install Conda (https://github.com/frol/docker-alpine-miniconda3/blob/master/Dockerfile)
RUN mkdir -p $CONDA_DIR && \
    cd /tmp && \
    wget -O /tmp/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh  https://repo.continuum.io/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    /bin/bash /tmp/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm /tmp/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda config --system --prepend channels conda-forge && \
    $CONDA_DIR/bin/conda config --system --set auto_update_conda false && \
    $CONDA_DIR/bin/conda config --system --set show_channel_urls true && \
    $CONDA_DIR/bin/conda install --quiet --yes conda="${CONDA_VERSION%.*}.*" && \
    $CONDA_DIR/bin/conda update --all --quiet --yes && \
    conda clean -tipsy && \
    conda install numpy scipy pandas scikit-learn && \
    conda install -c conda-forge pyarrow --yes && \
    conda clean -a -y

COPY conf/* $SPARK_CONF_DIR/
# $SPARK_HOME/conf gets cleaned by Spark on Kubernetes internals, create and add to classpath another directory for logging and other configs
COPY conf/* $SPARK_HOME/cluster-conf/
COPY entrypoint.sh /opt/
COPY Dockerfile /my_docker/

WORKDIR $SPARK_HOME/work-dir
RUN ls /opt/spark/jars/
ENTRYPOINT [ "/opt/entrypoint.sh" ]
