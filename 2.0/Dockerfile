# vim:set ft=dockerfile:
FROM debian:jessie-backports

# explicitly set user/group IDs
RUN groupadd -r cassandra --gid=999 && useradd -r -g cassandra --uid=999 cassandra

# grab gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& rm -r "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& apt-get purge -y --auto-remove ca-certificates wget

# solves warning: "jemalloc shared library could not be preloaded to speed up memory allocations"
RUN apt-get update && apt-get install -y --no-install-recommends curl libjemalloc1 && rm -rf /var/lib/apt/lists/*

RUN \
	cd ~ && \
	curl -o apache-cassandra-2.0.17-bin.tar.gz 'http://archive.apache.org/dist/cassandra/2.0.17/apache-cassandra-2.0.17-bin.tar.gz' && \
	tar xvfz apache-cassandra-2.0.17-bin.tar.gz

ENV CASSANDRA_VERSION 2.0.17

# https://issues.apache.org/jira/browse/CASSANDRA-11661
RUN sed -ri 's/^(JVM_PATCH_VERSION)=.*/\1=25/' /root/apache-cassandra-2.0.17/conf/cassandra-env.sh

ENV CASSANDRA_CONFIG /root/apache-cassandra-2.0.17/conf

COPY docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

RUN mkdir -p /var/lib/cassandra /var/log/cassandra "$CASSANDRA_CONFIG" \
	&& chown -R cassandra:cassandra /var/lib/cassandra "$CASSANDRA_CONFIG" \
	&& chmod 777 /var/lib/cassandra /var/log/cassandra "$CASSANDRA_CONFIG"
VOLUME /var/lib/cassandra


RUN curl -o oracle-java8-installer_8u131-1~webupd8~2_all.deb http://ppa.launchpad.net/webupd8team/java/ubuntu/pool/main/o/oracle-java8-installer/oracle-java8-installer_8u131-1~webupd8~2_all.deb


# Install Java 8
RUN \
  echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | debconf-set-selections && \
  apt-get update && \
  apt-get install -y apt-utils software-properties-common wget binutils java-common locales && \
  apt-get update && \
  dpkg -i oracle-java8-installer_8u131-1~webupd8~2_all.deb && \
  apt-get install -y -f && \
  rm -rf /var/lib/apt/lists/* && \
  rm -rf /var/cache/oracle-jdk8-installer


# 7000: intra-node communication
# 7001: TLS intra-node communication
# 7199: JMX
# 9042: CQL
# 9160: thrift service
EXPOSE 7000 7001 7199 9042 9160
CMD ["/root/apache-cassandra-2.0.17/bin/cassandra", "-f"]
