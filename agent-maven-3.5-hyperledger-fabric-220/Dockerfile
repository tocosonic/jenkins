FROM openshift/jenkins-slave-base-centos7:v3.11

MAINTAINER OpenShift Developer Services <jenkins+openshift-dev-services@redhat.com>

ENV MAVEN_VERSION=3.5 \
    BASH_ENV=/usr/local/bin/scl_enable \
    ENV=/usr/local/bin/scl_enable \
    PROMPT_COMMAND=". /usr/local/bin/scl_enable" \
    MAVEN_OPTS="-Duser.home=$HOME"

# Install Maven
RUN INSTALL_PKGS="java-11-openjdk-devel rh-maven35*" && \
    x86_EXTRA_RPMS=$(if [ "$(uname -m)" == "x86_64" ]; then echo -n java-11-openjdk-devel.i686; fi) && \
    yum install -y centos-release-scl-rh && \
    yum install -y --setopt=protected_multilib=false --enablerepo=centosplus $INSTALL_PKGS $x86_EXTRA_RPMS && \
    rpm -V java-11-openjdk-devel.x86_64 rh-maven35 $x86_EXTRA_RPMS && \
    yum clean all -y && \
    mkdir -p $HOME/.m2 

# Install shellcheck (required by SonarQube)
RUN yum install -y xz
RUN cd /tmp && export SC_VERSION=latest \
 && wget -qO- "https://github.com/koalaman/shellcheck/releases/download/${SC_VERSION?}/shellcheck-${SC_VERSION?}.linux.x86_64.tar.xz" | tar -xJv \
 && cp "shellcheck-${SC_VERSION}/shellcheck" /usr/bin/ \
 && cd -

# When bash is started non-interactively, to run a shell script, for example it
# looks for this variable and source the content of this file. This will enable
# the SCL for all scripts without need to do 'scl enable'.
ADD contrib/bin/* /usr/local/bin/
ADD ./contrib/settings.xml $HOME/.m2/

#set Java 11 as default
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.7.10-4.el7_8.x86_64
RUN alternatives --set java /usr/lib/jvm/java-11-openjdk-11.0.7.10-4.el7_8.x86_64/bin/java
#remove Java 1.8
RUN alternatives --remove java /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.i386/jre/bin/java
RUN alternatives --remove java /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/jre/bin/java

# Update oc to the most recent version
ENV OC_DOWNLOAD_URL=https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz
RUN wget $OC_DOWNLOAD_URL -O /tmp/oc.tgz
RUN tar xzvf /tmp/oc.tgz --strip-components=1 -C /usr/bin/
ENV OC_DOWNLOAD_URL=

RUN  yum install -y python36 python36-pip

# Pip has to point to pip3.6
RUN ln -s /usr/bin/pip3.6 /bin/pip
# Python has to point to python3.6 
RUN rm /usr/bin/python
RUN ln -s /usr/bin/python3.6 /usr/bin/python

RUN mkdir -p $HOME/.cache/pip

# Install Python packages
RUN pip install ruamel.yaml && \    
    pip install python-dotenv

# Download Hyperledger Fabric binaries
RUN mkdir -p $HOME/hlf-binaries && \ 
    cd $HOME/hlf-binaries && \ 
    curl https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh | bash -s -- 2.2.0 1.4.7 -d -s && \
    cd $HOME

# Add Binaries to PATH
ENV PATH="${HOME}/hlf-binaries/bin:${PATH}"

RUN chown -R 1001:0 $HOME && \
    chmod -R g+rw $HOME

USER 1001
