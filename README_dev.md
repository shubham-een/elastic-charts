## Steps to setup basic ES cluster

  ```
  helm install -f elasticsearch/values-master.yaml esmaster elasticsearch
  helm install -f elasticsearch/values-datahot.yaml esdatahot elasticsearch
  helm install -f elasticsearch/values-datawarm.yaml esdatawarm elasticsearch
  helm install eskibana kibana
  ```