# Terra4D Webtracker setup on AWS

* Get/create AWS account
* Get AWS access key from AWS console using AWS IAM service.
    * Create new group: docker-machine, with “AmazonEC2FullAccess” permissions
    * Create new user: webtracker, assign to group “docker-machine”
    * Goto user “webtracker”, tab “Security credentials”, Create Access key
    * Download/Save access key and secret in a safe place
* Install/Download “Docker for Windows” or “Install Docker on Ubuntu” using the [official guide](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
* For the remaining instruction we are using an Ubuntu machine. The process is not identical on a Windows machine, but should be quite similar.
* Create a text file named “aws_creds” with the AWS details/credentials (see AWS console):
    ```
    DOCKER_AWS_USER='webtracker'
    DOCKER_AWS_ACCESS_KEY='<secret>'
    DOCKER_AWS_SECRET_KEY='<secret>'
    DOCKER_AWS_REGION='eu-central-1'
    ```
* Create a docker machine on EC2:
    * `source aws_creds`
    * `docker-machine create --driver amazonec2 --amazonec2-region "${DOCKER_AWS_REGION}" --amazonec2-access-key "${DOCKER_AWS_ACCESS_KEY}" --amazonec2-secret-key "${DOCKER_AWS_SECRET_KEY}" <machine_name>`
* Use newly created docker machine:
    * `eval $(docker-machine env <machine_name>)`
* Goto https://github.com/aroth-fastprotect/terra4d-webtracker
    * Either use “git clone” or download the ZIP Packge from the GitHub page
* Go to the “terra4d-webtracker” directory
* start webtracker docker on AWS docker machine:
    * `WT_HOST=myhost WT_DOMAIN=mydomain.com docker-compose up -d`
    * This outputs the name of the newly created container; e.g. `terra4d-webtracker_webtracker_1`
* Check if everything is working fine:
    * `docker exec -it terra4d-webtracker_webtracker_1 /bin/bash`
    * `systemctl status`
        ▪ This should report system is running normally, if some service is not running use:
        ▪ `/usr/bin/restart-webtracker`
