# ---- http Build ----
FROM golang:1.15 as httpbuilder
WORKDIR /go/src/github.com/connectome-neuprint/
# fetch source from github tag
ARG NEUPRINT_TAG=master
RUN git clone --depth 1 --branch ${NEUPRINT_TAG} https://github.com/connectome-neuprint/neuPrintHTTP.git
WORKDIR /go/src/github.com/connectome-neuprint/neuPrintHTTP
# run go get
RUN go get -d -v ./
# run go build
RUN CGO_ENABLED=0 GOOS=linux go build .

# ---- explorer Build ----
FROM node:14 as explorerbuilder
RUN npm install -g npm@7.3.0
RUN apt-get -y update && apt-get -y install g++ build-essential
WORKDIR /opt
RUN git clone https://github.com/vishnubob/wait-for-it.git

WORKDIR /opt/certs
RUN openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout key.pem \
    -x509 -days 365 -out cert.pem \
    -subj /CN=\*.hhmi.org

WORKDIR /opt
ARG EXPLORER_TAG=master
RUN git clone --branch ${EXPLORER_TAG} --depth 1  https://github.com/connectome-neuprint/neuPrintExplorer.git

WORKDIR /opt/neuPrintExplorer
RUN npm ci --loglevel verbose
RUN npm run build

# ---- Release ----
FROM alpine:3.12.3
RUN apk add --no-cache bash
COPY --from=httpbuilder /go/src/github.com/connectome-neuprint/neuPrintHTTP/neuPrintHTTP /app/
COPY --from=httpbuilder /go/src/github.com/connectome-neuprint/neuPrintHTTP/swaggerdocs /app/swaggerdocs/
COPY --from=httpbuilder /go/src/github.com/connectome-neuprint/neuPrintHTTP/authorized.json /app/auth/
COPY --from=httpbuilder /go/src/github.com/connectome-neuprint/neuPrintHTTP/sampleconfig.json /app/config/
RUN mv /app/config/sampleconfig.json /app/config/config.json

COPY --from=explorerbuilder /opt/neuPrintExplorer/build /app/www/neuprintexplorer
COPY --from=explorerbuilder /opt/certs /app/certs/
COPY --from=explorerbuilder /opt/wait-for-it/wait-for-it.sh /app/
WORKDIR /app
CMD bash /app/wait-for-it.sh neo4j:7474 --timeout=0 --strict -- ./neuPrintHTTP -port 11000 -proxy-port 443 -public_read /app/config/config.json
