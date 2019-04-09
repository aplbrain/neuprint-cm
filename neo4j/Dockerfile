FROM neo4j:3.5

ARG NEO4J_DEFAULT_PASSWD=neo4j
ENV NEO4J_PASSWD=${NEO4J_DEFAULT_PASSWD}

RUN apk add --no-cache \
        curl

CMD bin/neo4j-admin set-initial-password ${NEO4J_PASSWD} || true && \
    bin/neo4j start && \
    tail -f logs/neo4j.log
