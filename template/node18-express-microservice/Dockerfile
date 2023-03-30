FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/openfaas/of-watchdog:0.9.11 as watchdog
FROM --platform=${TARGETPLATFORM:-linux/amd64} node:18-alpine as ship

ARG TARGETPLATFORM
ARG BUILDPLATFORM

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

RUN addgroup -S app && adduser -S -g app app

RUN apk --no-cache add ca-certificates

WORKDIR /root/

RUN mkdir -p /home/app

# Wrapper/boot-strapper
WORKDIR /home/app
COPY package.json ./

# Install template (microservice) packages first.
# This layer can then be used for each function based on this template.
RUN yarn --no-cache --frozen-lockfile

# Copy outer function handler
COPY index.js ./

WORKDIR /home/app/function

# Copy content of functions folder
COPY function/ ./

# Install additional packages needed by function
RUN yarn --no-cache --frozen-lockfile || :

# Set correct permissions to use non root user
WORKDIR /home/app/

# chmod for tmp is for a buildkit issue (@alexellis)
RUN chown app:app -R /home/app \
    && chmod 777 /tmp

USER app

ENV cgi_headers="true"
ENV fprocess="node index.js"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:3000"

ENV upstream_timeout="240m1s"
ENV exec_timeout="240m5s"
ENV read_timeout="240m10s"
ENV write_timeout="240m10s"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]

