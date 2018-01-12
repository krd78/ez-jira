FROM openjdk:8-alpine

ENV CONF_DIR            /var/atlassian/jira
ENV INSTALL_DIR         /opt/atlassian/jira
ENV JIRA_VERSION        7.7.0

ENV JAVA_CACERTS        $JAVA_HOME/jre/lib/security/cacerts
ENV CERTIFICATE         $CONF_DIR/certificate

# Required tools
RUN apk --no-cache add curl xmlstarlet bash

RUN mkdir -p ${CONF_DIR} && \
    chmod -R 700 "${CONF_DIR}" && \
    chown daemon:daemon "${CONF_DIR}"

RUN mkdir -p "${INSTALL_DIR}" && \
    curl -Ls "https://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-software-${JIRA_VERSION}.tar.gz" \
    | tar -xz --directory "${INSTALL_DIR}" --strip-components=1 --no-same-owner

RUN mkdir -p "${INSTALL_DIR}/conf/Catalina"
RUN chown daemon:daemon "${JAVA_CACERTS}"

RUN chmod -R 700 "${INSTALL_DIR}" && chown -R daemon:daemon "${INSTALL_DIR}"

RUN sed --in-place "s/java version/openjdk version/g" "${INSTALL_DIR}/bin/check-java.sh"
RUN echo -e "\njira.home=$CONF_DIR" >> "${INSTALL_DIR}/atlassian-jira/WEB-INF/classes/jira-application.properties"

RUN xmlstarlet ed --inplace \
    --delete "Server/@debug" \
    --delete "Server/Service/Connector/@debug" \
    --delete "Server/Service/Connector/@useURIValidationHack" \
    --delete "Server/Service/Connector/@minProcessors" \
    --delete "Server/Service/Connector/@maxProcessors" \
    --delete "Server/Service/Engine/@debug" \
    --delete "Server/Service/Engine/Host/@debug" \
    --delete "Server/Service/Engine/Host/Context/@debug" \
    "${INSTALL_DIR}/conf/server.xml"

USER daemon:daemon
WORKDIR /var/atlassian/jira

CMD ["/opt/atlassian/jira/bin/catalina.sh", "run"]
