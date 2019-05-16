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
    1. add an instance group
1. deploy CF with the new manifest
1. create ASG
1. create cups 
1. restage the app
