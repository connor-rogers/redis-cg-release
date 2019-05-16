# redis-cg-release

to make this work 

1. clone the repo
`git clone git@github.com:connor-rogers/redis-cg-release.git`
1. cd into the repo
`cd redis-cg-release`
1. Create a release
`bosh create-release --force`
1. upload-release
`bosh upload-release --force`
1. edit the manifest
    1. add it to the release section
    ```yaml
    releases:
    - name: redis-cg
      version: latest
    ```
    1. add an instance group
    ```yaml
      instance_groups:
      - name: redis-master
        instances: 1
        azs:
        - z1
        jobs:
        - name: redis-server
          release: redis-cg
          properties:
            port: <port>
            password: <password>
        vm_type: small-highmem
        stemcell: default
        persistent_disk_type: 10GB
        networks:
        - name: default
    ```
1. deploy CF with the new manifest
`bosh -d cf deploy <path-to-manifest>`
1. create ASG
    - create ASG definition
    ```json
    [
      {
        "protocol": "tcp",
        "destination": "10.0.1.15/32",
        "ports": "6379",
        "description": "Exposes the Redis VM to the app"
      }
    ]
    ```
    - create ASG
    `cf create-security-group <asg-name> security.json`
    - bind ASG to org
    `cf bind-security-group <asg-name> <org-name> <space-name>`
1. create user provided service definition
`cf create-user-provided-service redis -p "{\"host\":\"<service_ip>\",\"port\":\"<port>\",\"password\":\"<password>\"}"`
1. restage the app
`cf restage <app_name>`
1. test the 
`curl -X PUT http://<route_to_app>/foo -d "data=bar"`
`curl -X GET http://<route_to_app>/foo` should equal to `bar`
