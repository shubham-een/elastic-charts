
## Add S3 Repository (using k8s-secret):
Prerequisite:
1. Create a bucket in S3
2. Create an AWS IAM Role, IAM User, get access key and secret key

Elasticsearch
3. Use ES 8.1.1 docker image which has s3-repo plugin installed 
4. Create k8s secret for s3-cred and bootstrap.password

    - kubectl create secret generic <s3-secret-name> -n <k8s-namespace> --from-literal=s3.client.default.access_key='<aws_access_key>' --from-literal=s3.client.default.secret_key='<aws_secret_key>'

    - kubectl create secret generic <es-secret-pwd-name> -n <k8s-namespace> --from-literal=bootstrap.password='<es-password>'


5. Use the created secret <s3-secret-name> and <es-secret-pwd-name> in the ES values.yaml file.
    
    Deploy ES Helm chart.
    Sample command[test]:

    ```
    helm upgrade --install -f elasticsearch/values-beta.yaml esstandalone elasticsearch -n elasticsearch-test --set image=docker.elastic.co/elasticsearch/elasticsearch --set imageTag=8.1.1  --set replicas=1 --set keystore[0].secretName=s3-creds --set keystore[1].secretName=es-password
    ```

6. Create S3 Repository
   -  with UI
        - Open Kibana, Go to Snapshot and Restore
        - Go to Repositories
        - Create S3 Repository
        - Enter:
            - repo name: '<es-s3-repo-name>'
            - client: "default"
            - bucket: '<aws-s3-bucket-name>'
            - basepath: ""
        - Submit
    - With API
        - PUT _snapshot/<es-s3-repo-name>
            {
                "type" : "s3",
                "settings" : {
                "bucket" : "<aws-s3-bucket-name>",
                "region":"<aws-bucket-region>"
                }
            }
7.  Verify if the repository is connected
    POST /_snapshot/<es-s3-repo-name>/_verify



## Create a snapshot
1. Create a Snapshot

    - With API
        PUT /_snapshot/es-test-snapshot-s3-repo/snapshot_3_test?wait_for_completion=false
        {
        "indices": "index-name1,index-name2",
        "ignore_unavailable": false,
        "metadata": {
            "taken_by": "",
            "taken_because": ""
            }
        }

2. List Snapshots
    API:
        - GET /_cat/snapshots?format=json


## Restore the Snapshot
1. Create the indices

    PUT /index-name1
    {
    "settings": {"number_of_replicas": 0}
    }

    PUT /index-name2
    {
    "settings": {"number_of_replicas": 0}
    }

2. Close the indices
    - POST /index-name1/_close
    - POST /index-name2/_close

3. Restore
    - With API
    
    POST /_snapshot/<es-s3-repo-name>/<snapshot_name>/_restore?wait_for_completion=<true/false>
        {
        "indices": "index-name1,index-name2",
        "ignore_unavailable": false,
        "include_global_state": false,
        "include_aliases": true
        }

    - With UI
        1. Open Kibana, snapshot and restore page (http://localhost:5601/app/management/data/snapshot_restore/snapshots)
        2. click on the restore button on Actions tab of the snapshot
        3. Select
            - Data Streams and indices to restore
            - Restore Aliases
            - Rest of the settings default
        4. Next
        5. Restore Snapshot

4. Check the status of Restore
    - UI : http://localhost:5601/app/management/data/snapshot_restore/restore_status

5. The closed indices will be open after restore automatically
