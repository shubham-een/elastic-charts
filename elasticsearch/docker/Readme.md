## Build Docker Custom ElasticSearch docker image

## Build

version=8.1.0
docker build --build-arg elasticsearch_version=$(version) -t uvdeployment/videosearch:elasticsearch_$(version)_cstm_v2 .

## Push
version=8.1.0
docker push uvdeployment/videosearch:elasticsearch_$(version)_cstm_v2


## Run ES docker-compose (1node)
docker-compose -f docker-compose.yaml up