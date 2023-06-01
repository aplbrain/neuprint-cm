# neuPrint Container Management
Docker containers and configuration files for deploying neuPrint web application with the DevOps concept of Infrastructure as Code (IaC).

### Setup environment

Place .env file with environment variables in directory of launching docker-compose.

```
vim .env
```
Add database password to initialize neo4j with, 
```
NEO4J_PASSWD=<password>.
```

### Place configuration files in the expected locations so the container can mount them.
- create directory to store neo4j files
```
mkdir -p /data/db/neo4j/neuprint
```
 - neuprinthttp config.json 
    - example found in neuprinthttp/config/config.json
 ```
 /opt/app/neuprinthttp/conf/config.json
 ```
 - neuprinthttp authorized.json
    - example found in neuprinthttp/auth/authorized.json
    - this is not used unless the "auth-file" key is present in config.json
 ```
 /opt/app/neuprinthttp/auth/authorized.json
 ```
 - nginx cert files
 ```
/etc/nginx/ssl/cert.pem
/etc/nginx/ssl/key.pem
``` 

### Build the Images

```bash
docker build --build-arg EXPLORER_TAG=<neuprintexplorer_repo_tag> --build-arg NEUPRINT_TAG=<neuprint_repo_tag> . -t <registry>/neuprinthttp:<version>
```
- if the repo_tags are not specified the master branch will be used.
- if the registry and version numbers do not match the ones specified in the docker-compose.yml, change them there as well.

### Push the Images to the Registry

```
docker push <registry>/neuprinthttp:<version>
```
### For multiplatform images
- combine the build and the push into one step

```
docker buildx build --platform linux/amd64,linux/arm64 --build-arg EXPLORER_TAG=<neuprintexplorer_repo_tag> --build-arg NEUPRINT_TAG=<neuprint_repo_tag> -t <registry>/neuprinthttp:<version> --push .
```
- the --platform argument specifies which platforms to build for
- the --push makes sure that the images are pushed to the registry with the correct manifest list.

### Deploy Container Stack
```
sudo docker-compose -f docker-compose.yml -f docker-compose-prod.yml up -d
```

### Testing the Stack in dev Environment

```
sudo docker-compose -f docker-compose.yml -f docker-compose.override.yml up -d
```
- an example of the override yaml can be found in the template directory. Check the mount locations to identify where to place your local configuration files.
