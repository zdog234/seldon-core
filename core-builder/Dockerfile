FROM openjdk:13.0.1-jdk-buster

ENV PYTHON_VERSION "3.6.10"

# install docker
RUN \
    apt-get update && \
    apt-get install -y \
         apt-transport-https \
         ca-certificates \
         curl \
         wget \
         gnupg2 \
         software-properties-common && \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | apt-key add - && \
    add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
       $(lsb_release -cs) \
       stable" && \
    apt-get update && \
    apt-get install -y docker-ce

# Installing source to image (s2i)
RUN wget https://github.com/openshift/source-to-image/releases/download/v1.1.14/source-to-image-v1.1.14-874754de-linux-386.tar.gz && \
    tar -zxvf source-to-image-v1.1.14-874754de-linux-386.tar.gz && \
    rm source-to-image-v1.1.14-874754de-linux-386.tar.gz && \
    mv ./s2i /bin && \
    mv ./sti /bin

# Installing KIND
RUN wget https://github.com/kubernetes-sigs/kind/releases/download/v0.6.1/kind-linux-amd64 && \
    chmod +x kind-linux-amd64 && \
    mv ./kind-linux-amd64 /bin/kind

# Installing Kubectl
RUN wget https://storage.googleapis.com/kubernetes-release/release/v1.16.2/bin/linux/amd64/kubectl && \
    chmod +x ./kubectl && \
    mv ./kubectl /bin

RUN \
    apt-get update -y && \
    apt-get install -y vim jq build-essential pandoc && \
    apt-get remove -y --auto-remove && \
    apt-get clean -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install maven
ENV MAVEN_VERSION=3.6.3
ENV USER_HOME_DIR="/root"
ENV SHA=c35a1803a6e70a126e80b2b3ae33eed961f83ed74d18fcd16909b2d44d7dada3203f1ffe726c17ef8dcca2dcaa9fca676987befeadc9b9f759967a8cb77181c0
ENV BASE_URL=https://apache.osuosl.org/maven/maven-3/${MAVEN_VERSION}/binaries
RUN mkdir -p /usr/share/maven /usr/share/maven/ref \
  && curl -fsSL -o /tmp/apache-maven.tar.gz ${BASE_URL}/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
  && echo "${SHA}  /tmp/apache-maven.tar.gz" | sha512sum -c - \
  && tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 \
  && rm -f /tmp/apache-maven.tar.gz \
  && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn
ENV MAVEN_HOME=/usr/share/maven
ENV MAVEN_CONFIG="$USER_HOME_DIR/.m2"

# compression and ssl libs required for installation python and dependencies (pandas)
RUN apt-get upgrade && \
        apt-get update && \
        apt-get install zlib1g -y && \
        apt-get install zlib1g-dev -y && \
        apt-get install libssl-dev -y && \
	apt-get install libbz2-dev -y && \
	apt-get install liblzma-dev -y

RUN curl -SLO https://www.python.org/ftp/python/${PYTHON_VERSION}/Python-${PYTHON_VERSION}.tgz \
    && tar xvf Python-${PYTHON_VERSION}.tgz \
    && cd Python-${PYTHON_VERSION} \
    && ./configure --prefix=/usr/local --with-openssl \
    && make \
    && make altinstall \
    && cd / \
    && rm -rf Python-${PYTHON_VERSION}* \
    && (cd /usr/local/bin && ln -svn python3.6 python)

# Ensure python3 and pip3 are default
RUN curl -sS https://bootstrap.pypa.io/get-pip.py >>setup.py && \
        python setup.py
RUN update-alternatives --install /usr/bin/python python /usr/local/bin/python3.6 1
RUN rm -rf /usr/local/bin/pip && \
        ln -s /usr/local/bin/pip3.6 /usr/local/bin/pip

# Upgrade and install some Python packages
RUN pip install --upgrade pip setuptools wheel
RUN pip install pyyaml grpcio grpcio-tools
RUN apt-get remove -y --auto-remove \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# INSTALL GO
ENV PATH /usr/local/go/bin:$PATH
RUN wget https://dl.google.com/go/go1.14.7.linux-amd64.tar.gz && \
        tar -zxvf go1.14.7.linux-amd64.tar.gz && \
        mv go/ /usr/local/go
RUN curl -sL https://go.kubebuilder.io/dl/2.3.0/linux/amd64 | tar -xz -C /tmp/ && \
        mv /tmp/kubebuilder_2.3.0_linux_amd64 /usr/local/kubebuilder/
ENV PATH /usr/local/kubebuilder/bin:$PATH

# DOWNLOAD HELM
RUN curl -L https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash -s -- --version v3.2.0

# DOWNLOAD KUSTOMIZE
RUN wget https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv4.1.2/kustomize_v4.1.2_linux_amd64.tar.gz && \
    	tar -xzf kustomize_v4.1.2_linux_amd64.tar.gz && \
        chmod +x kustomize && \
        mv kustomize /usr/local/bin/kustomize

# iptables fix
RUN \
    update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy && \
    update-alternatives --set iptables /usr/sbin/iptables-legacy

# install jupyter
RUN pip install jupyter

# Install grpcurl
RUN go get github.com/fullstorydev/grpcurl/... && \
    go install github.com/fullstorydev/grpcurl/cmd/grpcurl

ENV PATH="${PATH}:/root/go/bin"

# Notebooks python requirements
COPY dev_requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

# Ginkgo
RUN go get -u github.com/onsi/ginkgo/ginkgo

# Helm docs
RUN GO111MODULE=on go get github.com/norwoodj/helm-docs/cmd/helm-docs@f66fdbd6fe

WORKDIR /work

# Define default command.
CMD ["bash"]
