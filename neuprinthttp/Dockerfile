# ---- http Build ----
FROM golang:1.24.2-bullseye AS httpbuilder
WORKDIR /opt
# fetch source from github tag
ARG NEUPRINT_TAG=master
RUN git clone --depth 1 --branch ${NEUPRINT_TAG} https://github.com/aplbrain/neuPrintHTTP.git
WORKDIR /opt/neuPrintHTTP
# run go build
RUN CGO_ENABLED=0 GOOS=linux go build .

# ---- explorer Build ----
FROM node:22 AS explorerbuilder
RUN apt-get -y update && apt-get -y install g++ build-essential
WORKDIR /opt
RUN git clone https://github.com/vishnubob/wait-for-it.git

WORKDIR /opt/certs
RUN openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout key.pem \
    -x509 -days 365 -out cert.pem \
    -subj /CN=\neuprint.bossdb.org

WORKDIR /opt
ARG EXPLORER_TAG=master-aplbrain
RUN git clone --branch ${EXPLORER_TAG} --depth 1  https://github.com/aplbrain/neuPrintExplorer.git

WORKDIR /opt/neuPrintExplorer
RUN npm install --loglevel verbose --force
RUN npm run build

# ---- Release ----
FROM python:3.9.6-slim
MAINTAINER APL BossDB Team
LABEL maintainer="hannah.martinez@jhuapl.edu"
ARG NEUPRINT_TAG=master
LABEL neuprinthttp_version=${NEUPRINT_TAG}
ARG EXPLORER_TAG=master
LABEL neuprintexplorer_version=${EXPLORER_TAG}
RUN pip install --progress-bar off --no-cache-dir pandas scipy scikit-learn
COPY --from=httpbuilder /opt/neuPrintHTTP/neuPrintHTTP /app/
COPY --from=httpbuilder /opt/neuPrintHTTP/canonical_celltype.py /app/
COPY --from=httpbuilder /opt/neuPrintHTTP/swaggerdocs /app/swaggerdocs/
COPY --from=httpbuilder /opt/neuPrintHTTP/authorized.json /app/auth/
# COPY --from=httpbuilder /go/src/github.com/connectome-neuprint/neuPrintHTTP/config-examples/config-neo4j.json /app/config/
# RUN mv /app/config/config-neo4j.json /app/config/config.json
RUN mkdir /app/config && touch /app/config/config.json

COPY --from=explorerbuilder /opt/neuPrintExplorer/build /app/www/neuprintexplorer
COPY --from=explorerbuilder /opt/certs /app/certs/
COPY --from=explorerbuilder /opt/wait-for-it/wait-for-it.sh /app/
WORKDIR /app
CMD ["bash", "/app/wait-for-it.sh", "neo4j:7474", "--timeout=0", "--strict", "--", "./neuPrintHTTP", "-port", "11000", "-proxy-port", "443", "-public_read", "/app/config/config.json"]

