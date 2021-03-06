ARG SPARK_BASE=cyliu0204/spark:2.4.5_2.11-hadoop_3.1.0-horovod_1.8_cloud
ARG LIVY_BUILDER=sasnouskikh/livy-builder:0.2

### Livy Builder Container
FROM $LIVY_BUILDER as build

ARG LIVY_GITHUB_REPO=jahstreet/incubator-livy
ARG LIVY_GITHUB_BRANCH=v0.6.0-incubating-kubernetes-support

RUN git clone -b ${LIVY_GITHUB_BRANCH} https://github.com/${LIVY_GITHUB_REPO}.git && \
    cd incubator-livy && \
    mvn clean package -DskipTests=true && \
    cp assembly/target/apache-livy-0.7.0-incubating-SNAPSHOT-bin.zip /

### Final Container
FROM $SPARK_BASE

ARG LIVY_VERSION_ARG=0.7.0-incubating-SNAPSHOT

LABEL maintainer="Aliaksandr Sasnouskikh <jaahstreetlove@gmail.com>"

ENV BASE_IMAGE              $SPARK_BASE#$BASE_IMAGE

ENV LIVY_VERSION            $LIVY_VERSION_ARG
ENV LIVY_HOME               /opt/livy
ENV LIVY_CONF_DIR           $LIVY_HOME/conf

ENV PATH                    $PATH:$LIVY_HOME/bin

# install livy
COPY --from=build /apache-livy-${LIVY_VERSION}-bin.zip /apache-livy-${LIVY_VERSION}-bin.zip
RUN unzip /apache-livy-${LIVY_VERSION}-bin.zip -d / && \
    mv /apache-livy-${LIVY_VERSION}-bin /opt/ && \
    rm -rf $LIVY_HOME && \
    ln -s /opt/apache-livy-${LIVY_VERSION}-bin $LIVY_HOME && \
    rm -f /apache-livy-${LIVY_VERSION}-bin.zip

#RUN ls $SPARK_HOME/jars

#RUN rm $SPARK_HOME/jars/kubernetes-client-4.1.2.jar

#ADD https://repo1.maven.org/maven2/io/fabric8/kubernetes-client/4.4.2/kubernetes-client-4.4.2.jar $SPARK_HOME/jars

RUN wget -O $SPARK_HOME/jars/aws-java-sdk-1.11.777.jar https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.11.777/aws-java-sdk-1.11.777.jar && \
wget -O $SPARK_HOME/jars/hadoop-aws-3.1.0.jar  https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.2.0/hadoop-aws-3.2.0.jar && \
wget -O $SPARK_HOME/jars/delta-core_2.11-0.6.0.jar https://repo1.maven.org/maven2/io/delta/delta-core_2.11/0.6.0/delta-core_2.11-0.6.0.jar

COPY client.jar  $SPARK_HOME/jars/

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


RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		bzip2 \
		unzip \
		xz-utils \
		\
# utilities for keeping Debian and OpenJDK CA certificates in sync
		ca-certificates p11-kit \
		\
# java.lang.UnsatisfiedLinkError: /usr/local/openjdk-11/lib/libfontmanager.so: libfreetype.so.6: cannot open shared object file: No such file or directory
# java.lang.NoClassDefFoundError: Could not initialize class sun.awt.X11FontManager
# https://github.com/docker-library/openjdk/pull/235#issuecomment-424466077
		fontconfig libfreetype6 \
	; \
	rm -rf /var/lib/apt/lists/*

# Default to UTF-8 file.encoding
ENV LANG C.UTF-8

ENV JAVA_HOME /usr/local/openjdk-8
ENV PATH $JAVA_HOME/bin:$PATH

# backwards compatibility shim
RUN { echo '#/bin/sh'; echo 'echo "$JAVA_HOME"'; } > /usr/local/bin/docker-java-home && chmod +x /usr/local/bin/docker-java-home && [ "$JAVA_HOME" = "$(docker-java-home)" ]

# https://adoptopenjdk.net/upstream.html
# >
# > What are these binaries?
# >
# > These binaries are built by Red Hat on their infrastructure on behalf of the OpenJDK jdk8u and jdk11u projects. The binaries are created from the unmodified source code at OpenJDK. Although no formal support agreement is provided, please report any bugs you may find to https://bugs.java.com/.
# >
ENV JAVA_VERSION 8u252
ENV JAVA_BASE_URL https://github.com/AdoptOpenJDK/openjdk8-upstream-binaries/releases/download/jdk8u252-b09/OpenJDK8U-jdk_
ENV JAVA_URL_VERSION 8u252b09
# https://github.com/docker-library/openjdk/issues/320#issuecomment-494050246
# >
# > I am the OpenJDK 8 and 11 Updates OpenJDK project lead.
# > ...
# > While it is true that the OpenJDK Governing Board has not sanctioned those releases, they (or rather we, since I am a member) didn't sanction Oracle's OpenJDK releases either. As far as I am aware, the lead of an OpenJDK project is entitled to release binary builds, and there is clearly a need for them.
# >

RUN set -eux; \
	\
	dpkgArch="$(dpkg --print-architecture)"; \
	case "$dpkgArch" in \
		amd64) upstreamArch='x64' ;; \
		arm64) upstreamArch='aarch64' ;; \
		*) echo >&2 "error: unsupported architecture: $dpkgArch" ;; \
	esac; \
	\
	wget -O openjdk.tgz.asc "${JAVA_BASE_URL}${upstreamArch}_linux_${JAVA_URL_VERSION}.tar.gz.sign"; \
	wget -O openjdk.tgz "${JAVA_BASE_URL}${upstreamArch}_linux_${JAVA_URL_VERSION}.tar.gz" --progress=dot:giga; \
	\
	export GNUPGHOME="$(mktemp -d)"; \
# TODO find a good link for users to verify this key is right (https://mail.openjdk.java.net/pipermail/jdk-updates-dev/2019-April/000951.html is one of the only mentions of it I can find); perhaps a note added to https://adoptopenjdk.net/upstream.html would make sense?
# no-self-sigs-only: https://salsa.debian.org/debian/gnupg2/commit/c93ca04a53569916308b369c8b218dad5ae8fe07
	gpg --batch --keyserver ha.pool.sks-keyservers.net --keyserver-options no-self-sigs-only --recv-keys CA5F11C6CE22644D42C6AC4492EF8D39DC13168F; \
# also verify that key was signed by Andrew Haley (the OpenJDK 8 and 11 Updates OpenJDK project lead)
# (https://github.com/docker-library/openjdk/pull/322#discussion_r286839190)
	gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys EAC843EBD3EFDB98CC772FADA5CD6035332FA671; \
	gpg --batch --list-sigs --keyid-format 0xLONG CA5F11C6CE22644D42C6AC4492EF8D39DC13168F \
		| tee /dev/stderr \
		| grep '0xA5CD6035332FA671' \
		| grep 'Andrew Haley'; \
	gpg --batch --verify openjdk.tgz.asc openjdk.tgz; \
	gpgconf --kill all; \
	rm -rf "$GNUPGHOME"; \
	\
	mkdir -p "$JAVA_HOME"; \
	tar --extract \
		--file openjdk.tgz \
		--directory "$JAVA_HOME" \
		--strip-components 1 \
		--no-same-owner \
	; \
	rm openjdk.tgz*; \
	\
# TODO strip "demo" and "man" folders?
	\
# update "cacerts" bundle to use Debian's CA certificates (and make sure it stays up-to-date with changes to Debian's store)
# see https://github.com/docker-library/openjdk/issues/327
#     http://rabexc.org/posts/certificates-not-working-java#comment-4099504075
#     https://salsa.debian.org/java-team/ca-certificates-java/blob/3e51a84e9104823319abeb31f880580e46f45a98/debian/jks-keystore.hook.in
#     https://git.alpinelinux.org/aports/tree/community/java-cacerts/APKBUILD?id=761af65f38b4570093461e6546dcf6b179d2b624#n29
	{ \
		echo '#!/usr/bin/env bash'; \
		echo 'set -Eeuo pipefail'; \
		echo 'if ! [ -d "$JAVA_HOME" ]; then echo >&2 "error: missing JAVA_HOME environment variable"; exit 1; fi'; \
# 8-jdk uses "$JAVA_HOME/jre/lib/security/cacerts" and 8-jre and 11+ uses "$JAVA_HOME/lib/security/cacerts" directly (no "jre" directory)
		echo 'cacertsFile=; for f in "$JAVA_HOME/lib/security/cacerts" "$JAVA_HOME/jre/lib/security/cacerts"; do if [ -e "$f" ]; then cacertsFile="$f"; break; fi; done'; \
		echo 'if [ -z "$cacertsFile" ] || ! [ -f "$cacertsFile" ]; then echo >&2 "error: failed to find cacerts file in $JAVA_HOME"; exit 1; fi'; \
		echo 'trust extract --overwrite --format=java-cacerts --filter=ca-anchors --purpose=server-auth "$cacertsFile"'; \
	} > /etc/ca-certificates/update.d/docker-openjdk; \
	chmod +x /etc/ca-certificates/update.d/docker-openjdk; \
	/etc/ca-certificates/update.d/docker-openjdk; \
	\
# https://github.com/docker-library/openjdk/issues/331#issuecomment-498834472
	find "$JAVA_HOME/lib" -name '*.so' -exec dirname '{}' ';' | sort -u > /etc/ld.so.conf.d/docker-openjdk.conf; \
	ldconfig; \
	\
# basic smoke test
	javac -version; \
	java -version

RUN pip install horovod[spark]

#  4040 - Spark UI port
#  7078 - Driver RPC port
#  7079 - Blockmanager port
#  8088 - JMX Exporter for Prometheus
# 10000 - Livy RPC Server for Jupyter integration
EXPOSE 4040 7078 7079 8088 10000
