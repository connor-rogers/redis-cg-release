# redis-cg-release

To use this service:

1. deploy [redis-example-app](https://github.com/pivotal-cf/cf-redis-example-app)
    ```
     git clone git@github.com:pivotal-cf/cf-redis-example-app.git && \
       pushd cf-redis-example-app && \
       cf push <app_name> --no-start && \
       popd
    ```
1. clone the repo
    ```
    git clone git@github.com:connor-rogers/redis-cg-release.git
    ```
1. cd into the repo
    ```
    cd redis-cg-release
    ```
1. Create a release
    ```
    bosh create-release --force
    ```
1. upload-release
    ```
    bosh upload-release
    ```
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
    ```
    bosh -d cf deploy <path-to-manifest>
    ```
1. create ASG
    - create ASG definition
    ```json
    [
      {
        "protocol": "tcp",
        "destination": "<service_ip>/32",
        "ports": "<port>",
        "description": "Exposes the Redis VM to the app"
      }
    ]
    ```
    to get the service ip run 
    ```
    bosh vms | grep redis-master | awk '{ print $4 }'
    ```
    - create ASG
    ```
    cf create-security-group <asg-name> security.json
    ```
    - bind ASG to org
    ```
    cf bind-security-group <asg-name> <org-name> <space-name>
    ```
1. create user provided service definition
    ```
    cf create-user-provided-service redis -p '{"host":"<service_ip>","port":"<port>","password":"<password>"}'
    ```
1. start the app
    ```
    cf start <app_name>
    ```
1. test the app
    ```
    $ curl -X PUT http://<route_to_app>/foo -d "data=bar"
    $ curl -X GET http://<route_to_app>/foo
    bar
    ```
