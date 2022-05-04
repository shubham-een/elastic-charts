## Build Docker Custom ElasticSearch docker image

## Build
version=7.16.2
docker build --build-arg elasticsearch_version=$(version) -t uvdeployment/videosearch:elasticsearch_$(version)_cstm_v2 .


version=8.1.1
docker build --build-arg elasticsearch_version=$(version) -t uvdeployment/videosearch:elasticsearch_$(version)_cstm_v2 .

## Push
version=8.1.1
docker push uvdeployment/videosearch:elasticsearch_$(version)_cstm_v2
