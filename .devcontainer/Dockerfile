# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.166.1/containers/java/.devcontainer/base.Dockerfile

# [Choice] Java version: 11, 15
# ARG VARIANT="11"
#FROM mcr.microsoft.com/vscode/devcontainers/java:0-${VARIANT}
FROM alpine:3.13
ENV WORKSPACE_DIR "/workspaces"


# [Option] Install Maven
 ARG INSTALL_MAVEN="false"
ARG MAVEN_VERSION=""
# [Option] Install Gradle
ARG INSTALL_GRADLE="false"
ARG GRADLE_VERSION=""
RUN if [ "${INSTALL_MAVEN}" = "true" ]; then su vscode -c "umask 0002 && . /usr/local/sdkman/bin/sdkman-init.sh && sdk install maven \"${MAVEN_VERSION}\""; fi \
    && if [ "${INSTALL_GRADLE}" = "true" ]; then su vscode -c "umask 0002 && . /usr/local/sdkman/bin/sdkman-init.sh && sdk install gradle \"${GRADLE_VERSION}\""; fi

# [Option] Install Node.js
ARG INSTALL_NODE="false"
ARG NODE_VERSION="lts/*"
RUN if [ "${INSTALL_NODE}" = "true" ]; then su vscode -c "umask 0002 && . /usr/local/share/nvm/nvm.sh && nvm install ${NODE_VERSION} 2>&1"; fi

# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>

# [Optional] Uncomment this line to install global node packages.
# RUN su vscode -c "source /usr/local/share/nvm/nvm.sh && npm install -g <your-package-here>" 2>&1

ENV TOMCAT_MAJOR=9 \
    TOMCAT_VERSION=9.0.44 \
    TOMCAT_HOME=/opt/tomcat \
    CATALINA_HOME=/opt/tomcat \
    CATALINA_OUT=/dev/null

ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

RUN mkdir -p /opt && \
    apk update --update-cache && \
    apk upgrade --update-cache && \
    apk add --update-cache openjdk11-jre curl sudo openssh openrc && \ 
    curl -jksSL -o /tmp/apache-tomcat.tar.gz http://archive.apache.org/dist/tomcat/tomcat-${TOMCAT_MAJOR}/v${TOMCAT_VERSION}/bin/apache-tomcat-${TOMCAT_VERSION}.tar.gz && \
    tar -C /opt -xvzf /tmp/apache-tomcat.tar.gz && \
    ln -s /opt/apache-tomcat-${TOMCAT_VERSION} ${TOMCAT_HOME} && \
    addgroup -g ${USER_GID} ${USERNAME} && \
    adduser -s /bin/ash -u ${USER_UID} -D -G ${USERNAME} ${USERNAME} && \
    echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME} && \
    chmod 0440 /etc/sudoers.d/${USERNAME} && \
    rm -rf /tmp/* /var/cache/apk/*

# For debug
# RUN apk add --no-cache bash vim

RUN rc-update add sshd && rc-status
RUN mkdir -p /run/openrc/ && touch /run/openrc/softlevel

# Enable Port Forwarding
RUN sed -i 's/^AllowTcpForwarding no/AllowTcpForwarding yes/g' /etc/ssh/sshd_config
RUN sed -i 's/^GatewayPorts no/GatewayPorts yes/g' /etc/ssh/sshd_config
RUN sed -i 's/^X11Forwarding no/X11Forwarding yes/g' /etc/ssh/sshd_config

# It will cause an ssh login error if user has no password
RUN echo "vscode:vscode" | chpasswd
# If need sudo
# RUN apk add --no-cache sudo
# RUN echo "ssh-user ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER vscode
RUN mkdir $HOME/.ssh
RUN ssh-keygen -m PEM -t rsa -b 1024 -C "vscode@example.com" -N "" -f $HOME/.ssh/id_rsa
RUN cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

USER root
# 秘密鍵をrootにコピー。鍵を作成したサーバとは別ホストで運用する場合、そちらに格納するため、
RUN cp /home/vscode/.ssh/id_rsa $HOME/vscode.pem
CMD ["/workspaces/wrapper_script.sh"]
#CMD /etc/init.d/sshd start && tail -f /dev/null

#ENTRYPOINT [ "/opt/tomcat/bin/catalina.sh", "run" ]