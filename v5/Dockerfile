FROM registry.access.redhat.com/rhel7
MAINTAINER carlos@mattermost.com

# Some ENV variables
ENV PATH="/mattermost/bin:${PATH}"
ENV MM_VERSION=5.13.2
ENV MM_RHEL_PACKAGE_VERSION=1.0-1

LABEL name="Mattermost-Enterprise-Edition-5" \
      maintainer="carlos@mattermost.com" \
      vendor="Mattermost" \
      version="$MM_RHEL_PACKAGE_VERSION" \
      release="$MM_VERSION" \
      summary="Install Mattermost Enterprise Edition" \
      description="Mattermost is an open source, hybrid messaging workspace that brings together your conversations and tools to get more done, faster."

RUN REPOLIST=rhel-7-server-rpms,rhel-7-server-optional-rpms \
    INSTALL_PKGS="ca-certificates curl libc6-compat libffi-dev linux-headers mailcap netcat-openbsd xmlsec-dev tzdata" && \
    yum -y update-minimal --disablerepo "*" --enablerepo rhel-7-server-rpms --setopt=tsflags=nodocs \
      --security --sec-severity=Important --sec-severity=Critical && \
    yum -y install --disablerepo "*" --enablerepo ${REPOLIST} --setopt=tsflags=nodocs ${INSTALL_PKGS} && \
    yum clean all

# Build argument to set Mattermost edition
ARG edition=enterprise
ARG PUID=2000
ARG PGID=2000
ARG MM_BINARY=

# Get Mattermost
RUN mkdir -p /mattermost/data /mattermost/plugins /mattermost/client/plugins \
    && if [ ! -z "$MM_BINARY" ]; then curl $MM_BINARY | tar -xvz ; \
      elif [ "$edition" = "team" ] ; then curl https://releases.mattermost.com/$MM_VERSION/mattermost-team-$MM_VERSION-linux-amd64.tar.gz | tar -xvz ; \
      else curl https://releases.mattermost.com/$MM_VERSION/mattermost-$MM_VERSION-linux-amd64.tar.gz | tar -xvz ; fi \
    && cp /mattermost/config/config.json /config.json.save \
    && rm -rf /mattermost/config/config.json \
    && mkdir -p /licenses && cp /mattermost/ENTERPRISE-EDITION-LICENSE.txt /licenses


# Get ready for production
RUN groupadd -g ${PGID} mattermost \
    && useradd -u ${PUID} -g mattermost -d /mattermost mattermost \
    && chown -R mattermost:mattermost /mattermost /config.json.save /mattermost/plugins /mattermost/client/plugins

USER mattermost

#Healthcheck to make sure container is ready
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost:8065/api/v4/system/ping || exit 1

# Configure entrypoint and command
COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]
WORKDIR /mattermost
CMD ["mattermost"]

EXPOSE 8065 8067 8074 8075

# Declare volumes for mount point directories
VOLUME ["/mattermost/data", "/mattermost/logs", "/mattermost/config", "/mattermost/plugins", "/mattermost/client/plugins"]
