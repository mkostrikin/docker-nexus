FROM       centos:centos7
MAINTAINER Sonatype <cloud-ops@sonatype.com>

ENV SONATYPE_WORK /sonatype-work
ENV NEXUS_VERSION 2.3.1-01

RUN yum install -y \
  curl tar createrepo \
  && yum clean all

RUN cd /var/tmp \
  && curl --fail --silent --location --retry 3 -O \
  --header "Cookie: oraclelicense=accept-securebackup-cookie; " \
  http://download.oracle.com/otn-pub/java/jdk/7u76-b13/jdk-7u76-linux-x64.rpm \
  && rpm -Ui jdk-7u76-linux-x64.rpm \
  && rm -rf jdk-7u76-linux-x64.rpm

RUN mkdir -p /opt/sonatype/nexus \
  && curl --fail --silent --location --retry 3 \
    http://download.sonatype.com/nexus/oss/nexus-2.3.1-bundle.tar.gz \
  | gunzip \
  | tar x -C /tmp nexus-${NEXUS_VERSION} \
  && mv /tmp/nexus-${NEXUS_VERSION}/* /opt/sonatype/nexus/ \
  && rm -rf /tmp/nexus-${NEXUS_VERSION} \
  && mkdir -p /opt/sonatype/sonatype-work/nexus/tmp \
  && sed 's|\(nexus-webapp-context-path=/\).*$|\1|' -i /opt/sonatype/nexus/conf/nexus.properties \
  && chown 200 -R /opt/sonatype

RUN useradd -r -u 200 -m -c "nexus role account" -d ${SONATYPE_WORK} -s /bin/false nexus

VOLUME ${SONATYPE_WORK}

EXPOSE 8081
WORKDIR /opt/sonatype/nexus
USER nexus
ENV CONTEXT_PATH /
ENV MAX_HEAP 768m
ENV MIN_HEAP 256m
ENV JAVA_OPTS -server -XX:MaxPermSize=192m -Djava.net.preferIPv4Stack=true
ENV LAUNCHER_CONF ./conf/jetty.xml
CMD java \
  -Dnexus-work=${SONATYPE_WORK} -Dnexus-webapp-context-path=${CONTEXT_PATH} \
  -Xms${MIN_HEAP} -Xmx${MAX_HEAP} \
  -cp 'conf/:lib/*' \
  ${JAVA_OPTS} \
  org.sonatype.nexus.bootstrap.Launcher ${LAUNCHER_CONF}

