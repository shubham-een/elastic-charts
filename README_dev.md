 ## Deployment Steps
 1. Create namespace
    ```
    kubectl create namespace es-<ClusterName> 
    ex: kubectl create namespace es-beta
    ```
 2. Deploy Certificate manager (Only once for K8s cluster)
    ```
    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.1/cert-manager.yaml
    ```

 3. Create self signed certificate for SSL transport
     ```
     kubectl apply -f certs/certs.yaml -n es-<ClusterName>
     ex: kubectl apply -f certs/certs.yaml -n es-beta
     ```
 
 4. Deploy the Elastic Search
    ```
    helm install elasticsearch elasticsearch --values elasticsearch/values-<ClusterName>.yaml -n es-<ClusterName> 
    ex: helm install elasticsearch elasticsearch --values elasticsearch/values-beta.yaml --set nodeSelector.node-class=es-beta  -n es-beta
    ```

 4. Get password
    ```
    kubectl get secrets --namespace=es-<ClusterName>  es-<ClusterName>-master-credentials -ojsonpath='{.data.password}' | base64 -d
    ex: kubectl get secrets --namespace=es-beta es-beta-master-credentials -ojsonpath='{.data.password}' | base64 -d
    kubectl get secrets --namespace=elasticsearch-test elasticsearch-test-master-credentials -ojsonpath='{.data.password}' | base64 -d
    ```

 5. Update the password of kibana_system user 
    ```
    - ssh into an elasticsearch node
    - RUN:
      bin/elasticsearch-reset-password --user kibana_system
    - Note the password, use it in next step.
    ```

 6. Deploy the Kibana
    ```
    Set the password in kibana/values.yaml
    helm install kibana kibana --set elasticsearchHosts=<ES master endpoint>  --set service.nodePort=<Node Port> --set nodeSelector.node-class=<Node class> -n es-<ClusterName>
    ex: helm install kibana kibana --set elasticsearchHosts=http://es-beta-master:9200  --set service.nodePort=30014 --set nodeSelector.node-class=es-beta -n es-beta
    ```
