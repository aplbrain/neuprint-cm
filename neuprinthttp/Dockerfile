# ---- http Build ----
FROM golang:latest as httpbuilder
#ARG APP_TAG=master
RUN mkdir /build 
WORKDIR /build 
RUN go get github.com/connectome-neuprint/neuPrintHTTP

# ---- explorer Build ----
FROM node:latest as explorerbuilder
WORKDIR /opt
RUN git clone https://github.com/connectome-neuprint/neuPrintExplorer.git
RUN git clone https://github.com/vishnubob/wait-for-it.git
WORKDIR /opt/neuPrintExplorer
RUN npm install && \
    npm run build
RUN mkdir certs
WORKDIR /opt/certs
RUN openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout cert.key \
    -x509 -days 365 -out cert.crt \
    -subj /CN=\*.hhmi.org 

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
CMD bash /app/wait-for-it.sh neo4j:7474 --timeout=0 --strict -- ./neuPrintHTTP -p 443 -public_read /app/config/config.json
