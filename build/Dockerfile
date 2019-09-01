FROM quay.io/operator-framework/ansible-operator:v0.10.0

USER root
RUN yum install -y python2-PyMySQL MySQL-python mariadb && pip install PyMySQL mysqlclient

RUN useradd -u ${USER_UID} ${USER_NAME}
USER ${USER_NAME}

COPY watches.yaml ${HOME}/watches.yaml

COPY roles/ ${HOME}/roles/
COPY playbooks/ ${HOME}/playbooks/
