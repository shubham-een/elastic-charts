
## Add S3 Repository:
1. Create a docker image with aws s3 plugin installed
2. Use the docker image in es helm chart
3. Create a bucket in S3
4. Create an AWS IAM Role, IAM User, get access key and secret key 
5. SSH in to ALL the ES nodes
    - Add aws credentials to elasticsearch-keystore for default client
    echo '<aws_access_key>' | ./bin/elasticsearch-keystore add --stdin s3.client.default.access_key
    echo '<aws_secret_key>' | ./bin/elasticsearch-keystore add --stdin s3.client.default.secret_key

    - This should be done in every ES node
    - TODO: Find alternative as this is not persistent

6. Reload the cluster settings so keystore data will be taken [Important]
  POST _nodes/reload_secure_settings

7. Create S3 Repository
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
8.  Verify if the repository is connected
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

