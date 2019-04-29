FROM centos:centos6

RUN set -ex \
  && echo ===== Create user ===== \
  && groupadd --gid 1000 centos \
  && echo 'create home directory manually will prevent os copying any file from skel directory into it' \
  && useradd -c "centos user" --home-dir /home/centos --shell /bin/bash -g centos -m --uid 1000 centos && chown -R centos:centos /home/centos \
  && usermod -a -G root centos \
  && mkdir -p /etc/sudoers.d \
  && echo "centos ALL=(ALL) NOPASSWD: ALL" | tee /etc/sudoers.d/centos \
  && chmod 0440 /etc/sudoers.d/centos \
  && yum -y install yum-utils \
  && yum -y install sudo \
# clean
  && package-cleanup --leaves \
  && yum clean all

RUN set -ex \
# install mysql client
  && yum -y localinstall https://dev.mysql.com/get/mysql80-community-release-el6-1.noarch.rpm \
  && yum --disablerepo=mysql80-community --enablerepo=mysql57-community --showduplicates list mysql-community-client \
  && yum -y --disablerepo=mysql80-community --enablerepo=mysql57-community install mysql-community-client-5.7.22-1.el6.x86_64 \
# clean
  && package-cleanup --leaves \
  && yum clean all

ENV CATALINA_HOME=/usr/local/tomcat \
    TOMCAT_MAJOR_VERSION=7 \
    TOMCAT_MINOR_VERSION=7.0.79

RUN set -ex \
# install utils
  && yum -y install tar wget \
# install Tomcat7
  && mkdir -p "${CATALINA_HOME}" \
  && cd ${CATALINA_HOME} \
  && wget -q https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz \
  && wget -qO- https://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR_VERSION}/v${TOMCAT_MINOR_VERSION}/bin/apache-tomcat-${TOMCAT_MINOR_VERSION}.tar.gz.md5 | md5sum -c - \
  && tar zxf apache-tomcat-*.tar.gz \
  && mv apache-tomcat-${TOMCAT_MINOR_VERSION}/* . \
  && sed -i "s/port=\"8080\"/port=\"8080\"\ URIEncoding=\"utf-8\"/g" ${CATALINA_HOME}/conf/server.xml \
  && rm -rf apache-tomcat-* \
  && mkdir -p ${CATALINA_HOME}/conf/Catalina/localhost ${CATALINA_HOME}/work/Catalina/localhost \
  && chown -R 0:1000 ${CATALINA_HOME} \
  && chmod -R 0775 ${CATALINA_HOME} \
  && chmod 07777 ${CATALINA_HOME}/logs ${CATALINA_HOME}/webapps ${CATALINA_HOME}/conf/Catalina/localhost ${CATALINA_HOME}/work/Catalina/localhost \
  && ls -la ${CATALINA_HOME} \
# clean
  && yum -y remove --setopt=clean_requirements_on_remove=1 tar wget \
  && package-cleanup --leaves \
  && yum clean all

ENV PATH=${CATALINA_HOME}/bin:$PATH

USER centos

ENV HOME=/home/centos \
    JAVA_HOME=/usr/lib/jvm/java \
    JAVA_VERSION=1.8.0 \
    SHELL=/bin/bash

COPY --chown=centos:centos . ${HOME}/workspace/cat

RUN	set -ex \
  && echo running under user $(whoami) \
  && ls -la ${HOME}/workspace/cat \
  && cp -f ${HOME}/workspace/cat/docker${CATALINA_HOME}/conf/tomcat-users.xml ${CATALINA_HOME}/conf/tomcat-users.xml \
# install Java7
  && sudo yum -y install java-${JAVA_VERSION}-openjdk java-${JAVA_VERSION}-openjdk-devel \
  && java -Xmx32m -version \
# build CAT
  && cd ${HOME}/workspace/cat \
  && if [[ -d $(pwd)/.mvn/repository ]]; then \
       ./mvnw clean install -Dmaven.repo.local=$(pwd)/.mvn/repository -DskipTests; \
     else \
       ./mvnw clean install -DskipTests; \
     fi \
  && sudo cp cat-home/target/*.war ${CATALINA_HOME}/webapps/cat.war \
  && sudo chown 1000:1000 ${CATALINA_HOME}/webapps/cat.war \
  && ./mvnw clean \
# datasource config and script
  && sudo mkdir -p /app /data/appdatas/cat && sudo chown -R 1000:1000 /data /app \
  && cp docker/datasources.xml /data/appdatas/cat/datasources.xml \
  && sudo cp docker/datasources.sh /datasources.sh \
  && sudo chmod +x /datasources.sh \
# clean
  && rm -rf $(pwd)/.mvn/repository ${HOME}/.m2 \
  && sudo yum -y remove --setopt=clean_requirements_on_remove=1 java-${JAVA_VERSION}-openjdk-devel \
  && sudo package-cleanup --leaves \
  && sudo yum clean all

ENV JAVA_HOME=/usr/lib/jvm/jre \
    PATH=${JAVA_HOME}/bin:$PATH

CMD java -version
EXPOSE 8080
WORKDIR /app
