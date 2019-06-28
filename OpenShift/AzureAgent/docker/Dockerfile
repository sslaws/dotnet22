#FROM "__FROM_IMAGE_STREAM_DEFINED_IN_TEMPLATE__"
FROM docker-registry.pathfinder.gov.bc.ca/q6qfzk-tools/dotnet22-base:2.2.107

ENV DEVOPS_AGENT_VERSION=2.153.2
ENV SUMMARY="Azure DevOps agent with .NET"  \
    DESCRIPTION="Azure DevOps agent with .NET v${DOTNET_VERSION}, nodejs v${NODE_VERSION}, Agent v${DEVOPS_AGENT_VERSION}"

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="azure-devlops-.net${NODE_MAJOR_VERSION}" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,azure,devops,agent-v${DEVOPS_AGENT_VERSION},.net-v${NODE_MAJOR_VERSION},nodejs-v${NODE_VERSION}" \
      release="1"

# Fetch stock azure client and install
# Fetch stock azure client and install
RUN mkdir -p ${APP_ROOT}/agent && \
    pushd ${APP_ROOT}/agent && \
    curl -sL https://vstsagentpackage.azureedge.net/agent/${DEVOPS_AGENT_VERSION}/vsts-agent-rhel.6-x64-${DEVOPS_AGENT_VERSION}.tar.gz | tar -zx && \
    echo rm -f vsts-agent-rhel.6-x64-${DEVOPS_AGENT_VERSION}.tar.gz && \
    popd && \
    echo find ${APP_ROOT}/agent -exec chmod 777 {} \;

RUN curl -sL https://raw.githubusercontent.com/sslaws/dotnet22/master/OpenShift/AzureAgent/scripts/start.sh \
    -o start.sh && \
    chmod +x start.sh

# Update environment variables
ENV PATH=$PATH:${APP_ROOT}/agent/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
ENV AZ_DEVOPS_ORG_URL=set_me_to_the_org_url
ENV AZ_DEVOPS_TOKEN=set_me_to_a_pat
ENV AZ_DEVOPS_POOL=set_me_to_an_agent_pool

CMD ["bash", "-c", "./start.sh", "-u", "$AZ_DEVOPS_ORG_URL", "-t", "$AZ_DEVOPS_TOKEN", "-p", "$AZ_DEVOPS_POOL"]
