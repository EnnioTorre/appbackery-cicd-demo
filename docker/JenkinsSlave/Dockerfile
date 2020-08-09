FROM openshift/jenkins-agent-maven-35-centos7
USER root
RUN useradd -m jenkins -u 1001 -g 0
RUN  yum -y --setopt=tsflags=nodocs install socat wget git tmux && \
     curl -kL -sf https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm -o ./google-chrome-stable_current_x86_64.rpm && \
     yum -y install ./google-chrome-stable_current_x86_64.rpm && \
     yum -y clean all && \
     rm -rf google-chrome-stable_current_x86_64.rpm

USER 1001