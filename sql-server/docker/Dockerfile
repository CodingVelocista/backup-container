# SQL Server Command Line Tools
FROM ubuntu:16.04

# Install Microsoft SQL Server Command Line Tools (sqlcmd/bcp)

# apt-get and system utilities
RUN apt-get update && apt-get install -y \
	curl apt-transport-https debconf-utils \
    && rm -rf /var/lib/apt/lists/*

# adding custom MS repository
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
RUN curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list

# install SQL Server drivers and tools
RUN apt-get update && ACCEPT_EULA=Y apt-get install -y msodbcsql mssql-tools
RUN echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
RUN /bin/bash -c "source ~/.bashrc"

ENV PATH=${PATH}:/opt/mssql/bin:/opt/mssql-tools/bin

RUN apt-get -y install locales
RUN locale-gen en_US.UTF-8
RUN update-locale LANG=en_US.UTF-8

COPY uid_entrypoint /opt/mssql-tools/bin/
RUN chmod -R a+rwx /opt/mssql-tools/bin/uid_entrypoint

RUN mkdir -p /var/opt/mssql && \
    chmod -R g=u /var/opt/mssql /etc/passwd

# Set the workdir to be root
WORKDIR /

# Load the backup script into the container (must be executable).
COPY backup.sh /
COPY webhook-template.json /

# ========================================================================================================
# Install go-crond (from https://github.com/BCDevOps/go-crond)
#  - Adds some additional logging enhancements on top of the upstream project; 
#    https://github.com/webdevops/go-crond
#
# CRON Jobs in OpenShift:
#  - https://blog.danman.eu/cron-jobs-in-openshift/
# --------------------------------------------------------------------------------------------------------
ARG SOURCE_REPO=BCDevOps
ARG GOCROND_VERSION=0.6.2
ADD https://github.com/$SOURCE_REPO/go-crond/releases/download/$GOCROND_VERSION/go-crond-64-linux /usr/bin/go-crond

USER root
RUN chmod ug+x /usr/bin/go-crond
# ========================================================================================================

### Containers should not run as root as a good practice
USER 10001

### user name recognition at runtime w/ an arbitrary uid - for OpenShift deployments
ENTRYPOINT [ "/opt/mssql-tools/bin/uid_entrypoint" ]

# Set the default CMD.
CMD /backup.sh