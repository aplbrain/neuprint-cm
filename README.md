# neuPrint Container Management
Docker containers and configuration files for deploying neuPrint web application with the DevOps concept of Infrastructure as Code (IaC).

### Setup environment

Place .env file with environment variables in directory of launching docker-compose.

```
vi .env
```
Add database password to initialize neo4j with, NEO4J_PASSWD=<password>.

Place configuration files
  - config.json 
  - authorized.json
  - cert files

### Mount volumes 
      - /opt/app/neuprinthttp/conf/:/app/config
      - /opt/app/neuprinthttp/auth:/app/auth
      - /opt/app/neuprinthttp/certs/:/app/certs

### Deploy containers
sudo docker-compose -f docker-compose.yml -f docker-compose-prod.yml up -d
