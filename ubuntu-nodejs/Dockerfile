FROM ubuntu:18.04 as base

LABEL maintainer="nguyenvan.thuoc@etc.vn"
LABEL version="1.0.0"

ENV TZ='Asia/Ho_Chi_Minh'

ENV PM2_VERSION=4.4.0
ENV NODE_VERSION=12.16.3
ENV NODE_FILE=node-v$NODE_VERSION-linux-x64.tar.xz

ENV NODE_DIR /usr/lib/node
ENV APP_DIR /home/c08/app
ENV DB_DRIVER_DIR /opt/oracle

#utils package
RUN apt-get update \
    && apt-get install -y tzdata xz-utils  unzip libaio1 \
    && rm /etc/localtime \
    && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
    && dpkg-reconfigure -f noninteractive tzdata \
    && groupadd -r c08 && useradd -r -m -s /bin/bash -g c08 c08 \
    && mkdir -p /home/c08/setup \
    && mkdir -p "${APP_DIR}" \
    && mkdir -p "${NODE_DIR}" \
    && mkdir -p "${DB_DRIVER_DIR}"

COPY instantclient-basic-linux.x64-19.6.0.0.0dbru.zip $DB_DRIVER_DIR

#for oracledb driver
RUN unzip /opt/oracle/instantclient-basic-linux.x64-19.6.0.0.0dbru.zip -d /opt/oracle && \
    mv /opt/oracle/instantclient_19_6 /opt/oracle/instantclient && \
    ln -sf /opt/oracle/instantclient/libclntsh.so.19.1 /opt/oracle/instantclient/libclntsh.so && \
    rm -rf /opt/oracle/instantclient-basic-linux.x64-19.6.0.0.0dbru.zip && \
    echo "/opt/oracle/instantclient" > /etc/ld.so.conf.d/oracle-instantclient.conf && \
    ldconfig

ENV LD_LIBRARY_PATH=/opt/oracle/instantclient_19_6:$LD_LIBRARY_PATH

WORKDIR /home/c08/setup
COPY $NODE_FILE .

RUN tar -xJf $NODE_FILE -C "${NODE_DIR}"
ENV NODE_HOME ${NODE_DIR}/node-v${NODE_VERSION}-linux-x64
ENV PATH $PATH:${NODE_HOME}/bin

RUN ["npm", "i", "npm@latest", "-g"]
RUN npm i pm2@$PM2_VERSION -g
RUN chown -R c08:c08 /home/c08
RUN chown -R c08:c08 $NODE_DIR
USER c08

#for build layer
FROM base as build

COPY package*.json ./
COPY tsconfig*.json ./
RUN ["npm", "install"]
COPY src ./
RUN ["npm", "run", "build"]

#for application layer
FROM base as app

COPY --from=build /home/c08/setup/dist $APP_DIR/dist
COPY --from=build /home/c08/setup/node_modules $APP_DIR/node_modules
COPY ecosystem.config.js $APP_DIR
WORKDIR ${APP_DIR}

EXPOSE 3000
CMD ["pm2-runtime", "start", "ecosystem.config.js"]

