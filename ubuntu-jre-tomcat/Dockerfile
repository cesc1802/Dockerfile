FROM ubuntu:18.04 AS base

LABEL maintainer="nguyenvan.thuoc@etc.vn"

ENV JDK_VERSION_MAJOR=8 \
    JDK_VERSION_UPDATE=251 
ENV JDK_VERSION=${JDK_VERSION_MAJOR}u${JDK_VERSION_UPDATE}
ENV JDK_VERSION_DOT=1.${JDK_VERSION_MAJOR}.0
ENV JDK_FILE=jre-${JDK_VERSION}-linux-x64.tar.gz

ENV TZ='Asia/Ho_Chi_Minh'
ENV JVM_DIR /usr/lib/jvm

RUN apt-get update \
    && apt-get install -y tzdata \
    && rm /etc/localtime \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata \
    && groupadd -r c08 && useradd -r -m -s /bin/bash -g c08 c08 \
    && mkdir -p /home/c08/setup \
    && mkdir -p "${JVM_DIR}"

WORKDIR /home/c08/setup
COPY $JDK_FILE .
RUN tar -xvf $JDK_FILE -C "${JVM_DIR}"

ENV JAVA_HOME ${JVM_DIR}/jdk${JDK_VERSION_DOT}_${JDK_VERSION_UPDATE}
ENV PATH $PATH:${JAVA_HOME}/bin

RUN chown -R c08:c08 /home/c08

USER c08

FROM base AS tomcat

ENV TOMCAT_VERSION=8.5.54 \
    TOMCAT_RELEASE=apache-tomcat \
    TOMCAT_DIR=/home/c08/tomcat 
ENV TOMCAT_FILE=${TOMCAT_RELEASE}-${TOMCAT_VERSION}.tar.gz
ENV TOMCAT_HOME=${TOMCAT_DIR}/${TOMCAT_RELEASE}-${TOMCAT_VERSION}

RUN mkdir -p "${TOMCAT_DIR}"
COPY ${TOMCAT_FILE} .
RUN tar -xvf $TOMCAT_FILE -C "${TOMCAT_DIR}"

FROM tomcat

ENV APPLICATION_NAME=c08-bca.war
ENV BILL_PATH /home/c08/bill
COPY ${APPLICATION_NAME} ${TOMCAT_HOME}/webapps

RUN mkdir -p "${BILL_PATH}"

VOLUME "/home/c08/tomcat/apache-tomcat-8.5.54/logs"
VOLUME "${BILL_PATH}"
WORKDIR /home/c08
RUN rm -rf setup

EXPOSE 8080

CMD [ "/home/c08/tomcat/apache-tomcat-8.5.54/bin/catalina.sh", "run" ]
