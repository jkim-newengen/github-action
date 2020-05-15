# GitHub Self Hosted Runner

https://github.com/myoung34/docker-github-actions-runner

Super Token !!! 359dcaf5eb58fe43a1624d5247da1647f5ce5ec0


## dependencies

```
apt-get install libgtk2.0-0 libgtk-3-0 libnotify-dev libgconf-2-4 libnss3 libxss1 libasound2 libxtst6 xauth xvfb
```


## Docker

```bash
docker run -d --restart always --name github-runner \
  -e ACCESS_TOKEN="359dcaf5eb58fe43a1624d5247da1647f5ce5ec0" \
  -e REPO_URL="https://github.com/jkim-newengen/github-action" \
  -e RUNNER_NAME="github-action-runner" \
  -e RUNNER_WORKDIR="/tmp/github-action-0" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /tmp/github-action:/tmp/github-action \
  myoung34/github-runner:latest
```


```dockerfile
  
# hadolint ignore=DL3007
FROM myoung34/github-runner-base:latest
LABEL maintainer="myoung34@my.apsu.edu"

ARG GH_RUNNER_VERSION="2.262.1"
ARG TARGETPLATFORM

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /actions-runner
COPY install_actions.sh /actions-runner

RUN chmod +x /actions-runner/install_actions.sh \
  && /actions-runner/install_actions.sh ${GH_RUNNER_VERSION} ${TARGETPLATFORM} \
  && rm /actions-runner/install_actions.sh

COPY entrypoint.sh /
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```


```dockerfile
FROM ubuntu:eoan
LABEL maintainer="myoung34@my.apsu.edu"

ARG GIT_VERSION="2.26.2"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
ENV DEBIAN_FRONTEND=noninteractive
# hadolint ignore=DL3003
RUN apt-get update && \
  apt-get install -y --no-install-recommends \
    curl \
    tar \
    apt-transport-https \
    ca-certificates \
    sudo \
    gnupg-agent \
    software-properties-common \
    build-essential \
    zlib1g-dev \
    gettext \
    liblttng-ust0 \
    libcurl4-openssl-dev \
    inetutils-ping \
    jq \
  && rm -rf /var/lib/apt/lists/* \
  && c_rehash \
  && cd /tmp \
  && curl -sL https://www.kernel.org/pub/software/scm/git/git-${GIT_VERSION}.tar.gz -o git.tgz \
  && tar zxf git.tgz \
  && cd git-${GIT_VERSION} \
  && ./configure --prefix=/usr \
  && make \
  && make install \
  && cd / \
  && rm -rf /tmp/git.tgz /tmp/git-${GIT_VERSION}

RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - \
  && [[ $(lsb_release -cs) == "eoan" ]] && ( add-apt-repository "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu disco stable" ) || ( add-apt-repository "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" )\
  && apt-get update \
  && apt-get install -y docker-ce docker-ce-cli containerd.io --no-install-recommends \
  && rm -rf /var/lib/apt/lists/*

RUN curl -sL "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
  && chmod +x /usr/local/bin/docker-compose
```