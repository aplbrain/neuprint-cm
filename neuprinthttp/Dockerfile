# ---- http Build ----
FROM golang:latest as httpbuilder
RUN mkdir /build
WORKDIR /build
RUN go get -tags nokafka github.com/connectome-neuprint/neuPrintHTTP

# ---- explorer Build ----
FROM node:10-stretch as explorerbuilder
RUN apt-get -y update && apt-get -y install g++ build-essential
WORKDIR /opt
RUN git clone https://github.com/vishnubob/wait-for-it.git

RUN mkdir certs
WORKDIR /opt/certs
RUN openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout cert.key \
    -x509 -days 365 -out cert.crt \
    -subj /CN=\*.hhmi.org

WORKDIR /opt
ARG APP_TAG=master
RUN git clone --branch ${APP_TAG} --depth 1  https://github.com/connectome-neuprint/neuPrintExplorer.git

WORKDIR /opt/neuPrintExplorer
RUN npm ci --loglevel verbose
RUN npm run build-public

# ---- Release ----
FROM alpine
RUN apk add --no-cache \
        libc6-compat \
        ca-certificates \
        openssl \
        bash
COPY --from=httpbuilder /go/bin/neuPrintHTTP /app/
COPY --from=httpbuilder /go/src/github.com/connectome-neuprint/neuPrintHTTP/swaggerdocs /app/swaggerdocs/
COPY --from=httpbuilder /go/src/github.com/connectome-neuprint/neuPrintHTTP/authorized.json /app/auth/
COPY --from=httpbuilder /go/src/github.com/connectome-neuprint/neuPrintHTTP/sampleconfig.json /app/config/
RUN mv /app/config/sampleconfig.json /app/config/config.json
COPY --from=explorerbuilder /opt/neuPrintExplorer/build /app/www/neuprintexplorer
COPY --from=explorerbuilder /opt/certs /app/certs/
COPY --from=explorerbuilder /opt/wait-for-it/wait-for-it.sh /app/
WORKDIR /app
CMD bash /app/wait-for-it.sh neo4j:7474 --timeout=0 --strict -- ./neuPrintHTTP -port 443 -public_read /app/config/config.json
