FROM debian:stretch

# Update system and install required packages
ENV DEBIAN_FRONTEND noninteractive

# Install git, download and extract Stash and create the required directory layout.
# Try to limit the number of RUN instructions to minimise the number of layers that will need to be created.
RUN apt-get update -qq \
    && apt-get install -y wget curl git unzip libtcnative-1 xmlstarlet gosu apt-transport-https gnupg2 software-properties-common \
    && apt-get clean autoclean \
    && apt-get autoremove --yes \
    && rm -rf /var/lib/{apt,dpkg,cache,log}/

# Docker
RUN curl -fsSL https://download.docker.com/linux/debian/gpg |  apt-key add - && \
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" && \
    apt-get update && apt-get install -y docker-ce=17.06.* && \
    echo docker-ce hold | dpkg --set-selections && \
    curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose && chmod +x /usr/local/bin/docker-compose

# Download Oracle JDK
ENV ORACLE_JDK_VERSION jdk-8u151
ENV ORACLE_JDK_URL http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz
RUN mkdir -p /opt/jdk/$ORACLE_JDK_VERSION && \
    wget --header "Cookie: oraclelicense=accept-securebackup-cookie" -O /opt/jdk/$ORACLE_JDK_VERSION/$ORACLE_JDK_VERSION.tar.gz $ORACLE_JDK_URL && \
    tar -zxf /opt/jdk/$ORACLE_JDK_VERSION/$ORACLE_JDK_VERSION.tar.gz --strip-components=1 -C /opt/jdk/$ORACLE_JDK_VERSION && \
    rm /opt/jdk/$ORACLE_JDK_VERSION/$ORACLE_JDK_VERSION.tar.gz && \
    update-alternatives --install /usr/bin/java java /opt/jdk/$ORACLE_JDK_VERSION/bin/java 100 && \
    update-alternatives --install /usr/bin/javac javac /opt/jdk/$ORACLE_JDK_VERSION/bin/javac 100

ENV DOWNLOAD_URL        https://www.atlassian.com/software/bamboo/downloads/binary/atlassian-bamboo-

ENV JAVA_HOME /opt/jdk/${ORACLE_JDK_VERSION}
ENV JAVA_TRUSTSTORE ${JAVA_HOME}/jre/lib/security/cacerts
ENV JAVA_TRUSTSTORE_PASSWORD changeit
ENV JAVA_OPTS "-Djavax.net.ssl.trustStore=${JAVA_TRUSTSTORE} -Djavax.net.ssl.trustStorePassword=${JAVA_TRUSTSTORE_PASSWORD}"

# Use the default unprivileged account. This could be considered bad practice
# on systems where multiple processes end up being executed by 'daemon' but
# here we only ever run one process anyway.
ENV RUN_USER            atlassian
ENV RUN_USER_UID        5000
ENV RUN_GROUP           atlassian
ENV RUN_GROUP_GID       5000

RUN \
    groupadd --gid ${RUN_GROUP_GID} -r ${RUN_GROUP} && \
    useradd -r --uid ${RUN_USER_UID} -g ${RUN_GROUP} ${RUN_USER} && \
    usermod -a -G docker ${RUN_USER}

ENV BAMBOO_INSTALL_DIR   /opt/atlassian/bamboo
ENV BAMBOO_HOME   /var/atlassian/bamboo/data
ENV BAMBOO_VERSION 6.2.5

RUN mkdir -p ${BAMBOO_INSTALL_DIR} ${BAMBOO_HOME} && \
    curl -L --silent ${DOWNLOAD_URL}${BAMBOO_VERSION}.tar.gz | tar -xz --strip=1 -C "$BAMBOO_INSTALL_DIR" && \
    chown -R ${RUN_USER}:${RUN_GROUP} ${BAMBOO_INSTALL_DIR} ${BAMBOO_HOME}


# MySQL connector
ENV MYSQL_CONNECTOR_VERSION 5.1.44
RUN \
    wget -O ${BAMBOO_INSTALL_DIR}/mysql-connector-java-${MYSQL_CONNECTOR_VERSION}.tar.gz http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-${MYSQL_CONNECTOR_VERSION}.tar.gz && \
    tar xzf ${BAMBOO_INSTALL_DIR}/mysql-connector-java-${MYSQL_CONNECTOR_VERSION}.tar.gz -C ${BAMBOO_INSTALL_DIR} && \
    mv ${BAMBOO_INSTALL_DIR}/mysql-connector-java-${MYSQL_CONNECTOR_VERSION}/mysql-connector-java-${MYSQL_CONNECTOR_VERSION}-bin.jar ${BAMBOO_INSTALL_DIR}/lib/ && \
    rm -rf ${BAMBOO_INSTALL_DIR}/mysql-connector-java-${MYSQL_CONNECTOR_VERSION}.tar.gz ${BAMBOO_INSTALL_DIR}/mysql-connector-java-${MYSQL_CONNECTOR_VERSION}

ENV PATH $BAMBOO_INSTALL_DIR/bin:$JAVA_HOME/bin:$PATH

EXPOSE 8085

COPY docker-entrypoint.sh /entrypoint.sh

RUN chmod a+x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]

WORKDIR $BAMBOO_INSTALL_DIR

CMD ["start-bamboo.sh", "-fg"]
