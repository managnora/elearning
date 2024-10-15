FROM node:18.19.0

# bind your app to the gateway IP
ENV HOST=0.0.0.0

RUN yarn global add n

RUN n 18.19.0

COPY --chown=elearningdev . /var/www/elearning

# File mode/permissions trick
RUN groupadd --gid 1001 elearningdev \
    && useradd --uid 1001 --gid elearningdev --shell /bin/bash --create-home elearningdev \
    && chown -R elearningdev /var/www/elearning \
    && usermod -a -G www-data elearningdev

WORKDIR /var/www/elearning/front

USER elearningdev:elearningdev

COPY docker/frontend/startup.sh /home/elearningdev/

CMD ["/bin/bash","-c","~/startup.sh"]
