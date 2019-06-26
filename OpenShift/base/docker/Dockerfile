FROM registry.access.redhat.com/openshift3/rhel7:latest

ARG NODE_MAJOR_VERSION=10
ARG NODE_VERSION=v10.13.0
ARG DOTNET_MAJOR_VERSION=2
ARG DOTNET_VERSION=2.2
ENV SUMMARY=".NET Core 2.2 Base image"  \
    DESCRIPTION="base app image with .NET v${DOTNET_VERSION}, nodejs v${NODE_VERSION}"

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="azure-devlops-.net${NODE_MAJOR_VERSION}" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,azure,devops,agent-v${DEVOPS_AGENT_VERSION},.net-v${NODE_MAJOR_VERSION},nodejs-v${NODE_VERSION}" \
      release="1"

ENV PATH=$HOME/.local/bin/:$PATH \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 

# We need to call 2 (!) yum commands before being able to enable repositories properly
# This is a workaround for https://bugzilla.redhat.com/show_bug.cgi?id=1479388
RUN yum repolist > /dev/null && \
    yum install -y yum-utils && \
    yum-config-manager --disable \* &> /dev/null && \
    yum-config-manager --enable rhel-server-rhscl-7-rpms && \
    yum-config-manager --enable rhel-7-server-rpms && \
    yum-config-manager --enable rhel-7-server-optional-rpms && \
    yum-config-manager --enable rhel-7-server-dotnet-rpms && \
    yum-config-manager --enable rhel-7-server-ose-3.11-rpms && \
    INSTALL_PKGS="nss_wrapper libtool-ltdl \
        scl-utils rh-dotnet22 atomic-openshift-clients" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    # Remove redhat-logos (httpd dependency) to keep image size smaller.
    # rpm -e --nodeps redhat-logos && \
    yum clean all -y && \
    rm -rf /var/cache/yum

# enable the rh-dotnet22 software collection environment 
RUN scl enable rh-dotnet22 bash

RUN curl -sL http://download.icu-project.org/files/icu4c/57.1/icu4c-57_1-RHEL6-x64.tgz | \
    tar -zx -C /

RUN curl -sL https://github.com/jleach/openshift-components/blob/master/cicd/azure-devops-agent/git-2.21.0-bin.tgz?raw=true | \
    tar -zx -C / 

RUN curl -sL https://raw.githubusercontent.com/jleach/openshift-components/master/cicd/azure-devops-agent/scripts/start.sh \
    -o /opt/az/start.sh && \
    chmod +x /opt/az/start.sh

# Fetch stock nodejs and install
RUN mkdir -p /opt && \
    pushd /opt && \
    curl -sL https://nodejs.org/dist/${NODE_VERSION}/node-${NODE_VERSION}-linux-x64.tar.xz | tar -Jx && \
    rm -f node-${NODE_VERSION}-linux-x64.tar.xz

# Update environment variables
ENV PATH=$PATH:/opt/rh/rh-dotnet22/root/bin:/usr/local/git/bin:/opt/node-${NODE_VERSION}-linux-x64/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
ENV NODE_HOME=/opt/node-${NODE_VERSION}-linux-x64

# Update the version of `npm` that came with the packages above
# and install a few additional tools.
RUN npm i -g npm@latest yarn@latest nsp@latest nodemon@latest && \
    rm -rf ~/.npm && \
    node -v && \
    npm -v

#Create a default App user
RUN useradd -m app

#Set default user and working dir
USER app
WORKDIR /home/app

#CMD ["bash", "-c", "/opt/az/start.sh", "-u", "$AZ_DEVOPS_ORG_URL", "-t", "$AZ_DEVOPS_TOKEN"]