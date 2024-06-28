ARG TAG=@sha256:77906da86b60585ce12215807090eb327e7386c8fafb5402369e421f44eff17e

FROM ubuntu${TAG} as builder

COPY requirements.txt /tmp
RUN apt update && apt install python3-pip python3-setuptools python3-dev gcc -y && \
     python3 -m pip wheel -w /tmp/wheel -r /tmp/requirements.txt

FROM ubuntu${TAG}

LABEL description="Ubuntu 24.04 Wazuh Docker Agent"

ARG AGENT_VERSION="4.7.3-1"
ENV JOIN_MANAGER_MASTER_HOST=""
ENV JOIN_MANAGER_WORKER_HOST=""
ENV VIRUS_TOTAL_KEY=""
ENV JOIN_MANAGER_PROTOCOL="https"
ENV JOIN_MANAGER_USER=""
ENV JOIN_MANAGER_PASSWORD=""
ENV JOIN_MANAGER_API_PORT="55000"
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV DEBIAN_FRONTEND=noninteractive
ENV PIP_ROOT_USER_ACTION=ignore
ENV TZ=Etc/UTC

COPY *.py /var/ossec/

WORKDIR /var/ossec/

COPY --from=builder /tmp/wheel /tmp/wheel
RUN apt update && apt install -y procps curl apt-transport-https gnupg2 inotify-tools python3-docker python3-setuptools python3-pip && \
  curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add - && \
  echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list && \
  apt update && apt install -y wazuh-agent=${AGENT_VERSION}  && \
  mkdir -p /usr/share/man/man1 && \
  apt install -y openjdk-11-jdk

COPY *.py /var/ossec/
WORKDIR /var/ossec/
COPY --from=builder /tmp/wheel /tmp/wheel

RUN pip3 install --no-index /tmp/wheel/*.whl && \
  chmod +x /var/ossec/deregister_agent.py && \
  chmod +x /var/ossec/register_agent.py && \
  apt-get clean autoclean && \
  apt-get autoremove -y && \
  rm -rf /var/lib/{apt,dpkg,cache,log}/ && \
  rm -rf  /tmp/* /var/tmp/* /var/log/* && \
  chown -R wazuh:wazuh /var/ossec/
EXPOSE 5000
ENTRYPOINT ["./register_agent.py"]
