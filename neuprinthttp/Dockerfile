# ---- http Build ----
FROM golang:1.18 as httpbuilder
WORKDIR /opt
# fetch source from github tag
ARG NEUPRINT_TAG=master
RUN git clone --depth 1 --branch ${NEUPRINT_TAG} https://github.com/aplbrain/neuPrintHTTP.git
# WORKDIR /go/src/github.com/connectome-neuprint/neuPrintHTTP
# run go build
RUN go build .

# ---- explorer Build ----
FROM node:14 as explorerbuilder
RUN npm install -g npm@7.15.1
RUN apt-get -y update && apt-get -y install g++ build-essential
WORKDIR /opt
RUN git clone https://github.com/vishnubob/wait-for-it.git

WORKDIR /opt/certs
RUN openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout key.pem \
    -x509 -days 365 -out cert.pem \
    -subj /CN=\neuprint-dev.bossdb.org

WORKDIR /opt
ARG EXPLORER_TAG=master
RUN git clone --branch ${EXPLORER_TAG} --depth 1  https://github.com/aplbrain/neuPrintExplorer.git

WORKDIR /opt/neuPrintExplorer
RUN npm install --loglevel verbose
RUN npm run build

# ---- Release ----
FROM python:3.9-slim
MAINTAINER APL BossDB Team
LABEL maintainer="hannah.martinez@jhuapl.edu"
ARG NEUPRINT_TAG=master
LABEL neuprinthttp_version=${NEUPRINT_TAG}
ARG EXPLORER_TAG=master
LABEL neuprintexplorer_version=${EXPLORER_TAG}
RUN pip install --no-cache-dir pandas scipy scikit-learn
COPY --from=httpbuilder /opt/neuPrintHTTP/neuPrintHTTP /app/
COPY --from=httpbuilder /opt/neuPrintHTTP/canonical_celltype.py /app/
COPY --from=httpbuilder /opt/neuPrintHTTP/swaggerdocs /app/swaggerdocs/
COPY --from=httpbuilder /opt/neuPrintHTTP/authorized.json /app/auth/
COPY --from=httpbuilder /opt/neuPrintHTTP/sampleconfig.json /app/config/
RUN mv /app/config/sampleconfig.json /app/config/config.json

COPY --from=explorerbuilder /opt/neuPrintExplorer/build /app/www/neuprintexplorer
COPY --from=explorerbuilder /opt/certs /app/certs/
COPY --from=explorerbuilder /opt/wait-for-it/wait-for-it.sh /app/
WORKDIR /app
CMD bash /app/wait-for-it.sh neo4j:7474 --timeout=0 --strict -- ./neuPrintHTTP -port 11000 -proxy-port 443 -public_read /app/config/config.json
