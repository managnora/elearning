FROM node:18.19.0

ARG DOCKER_USER_NAME
ARG DOCKER_HOST_UID
ARG DOCKER_GROUP_NAME
ARG DOCKER_HOST_GID

ARG SOURCE_ROOT

# bind your app to the gateway IP
ENV HOST=0.0.0.0

# RUN yarn global add n

# RUN n 18.19.0

COPY --chown=${DOCKER_USER_NAME} . ${SOURCE_ROOT}

# File mode/permissions trick
RUN groupadd --gid ${DOCKER_HOST_GID} ${DOCKER_GROUP_NAME} \
    && useradd --uid ${DOCKER_HOST_UID} --gid ${DOCKER_GROUP_NAME} --shell /bin/bash --create-home ${DOCKER_USER_NAME} \
    && chown -R ${DOCKER_USER_NAME} ${SOURCE_ROOT} \
    && usermod -a -G www-data ${DOCKER_USER_NAME}

WORKDIR ${SOURCE_ROOT}/frontend

USER ${DOCKER_USER_NAME}:${DOCKER_GROUP_NAME}

COPY docker/frontend/startup.sh /home/${DOCKER_USER_NAME}/

CMD ["/bin/bash","-c","~/startup.sh"]
