# Terra4D Webtracker setup on AWS

* Get/create AWS account
* Get AWS access key from AWS console using AWS IAM service.
    * Create new group: docker-machine, with “AmazonEC2FullAccess” permissions
    * Create new user: webtracker, assign to group “docker-machine”
    * Goto user “webtracker”, tab “Security credentials”, Create Access key
    * Download/Save access key and secret in a safe place
* Install/Download “Docker for Windows” or “Install Docker on Ubuntu” using the [official guide](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
* For the remaining instruction we are using an Ubuntu machine. The process is not identical on a Windows machine, but should be quite similar.
* Goto https://github.com/aroth-fastprotect/terra4d-webtracker
    * Either use “git clone” or download the ZIP Packge from the GitHub page
* Go to the “terra4d-webtracker” directory
* Open the text file named “aws_creds” and change the AWS credentials and AWS region (see AWS console):
    ```
    export DOCKER_AWS_ACCESS_KEY='<secret>'
    export DOCKER_AWS_SECRET_KEY='<secret>'
    export DOCKER_AWS_REGION='eu-central-1'
    ```
* Create a docker machine on EC2:
    * Open a console session
    * `source aws_creds`
    * `docker-machine create --driver amazonec2 --amazonec2-region "${DOCKER_AWS_REGION}" --amazonec2-access-key "${DOCKER_AWS_ACCESS_KEY}" --amazonec2-secret-key "${DOCKER_AWS_SECRET_KEY}" --amazonec2-instance-type "${DOCKER_AWS_INSTANCE_TYPE}" <machine_name>`
* Use newly created docker machine:
    * `eval $(docker-machine env <machine_name>)`
* Go to the AWS console and check the newly created instance in "EC2", "Instances"
    * Select the new instance "<machine_name>" and copy the "Public DNS" entry; e.g. ec2-15-135-214-16.eu-central-1.compute.amazonaws.com
* Go to the DNS management tool for your domain
    * Create a DNS-Record of type CNAME for "myhost.mydomain.com." and set the target to the "Public DNS" entry copied from the AWS console.
    * Check if the DNS record works and can be correctly resolved.
    * Run `host myhost.mydomain.com.`, which should return for example
        ```
myhost.mydomain.com. is an alias for ec2-15-135-214-16.eu-central-1.compute.amazonaws.com
ec2-15-135-214-16.eu-central-1.compute.amazonaws.com has address 15.135.214.16
        ```
* start webtracker docker on AWS docker machine:
    * `docker-compose up -d`
    * This outputs the name of the newly created container; e.g. `terra4d-webtracker_webtracker_1`
* Run `docker exec -it terra4d-webtracker_webtracker_1 /bin/bash` to get a SSH console session on the new machine
* Run `/root/first-start.sh`
    * This either gives a message like "myhost.mydomain.com has been correctly registered with Let's Encrypt." or results in an error message
    "Failed to register myhost.mydomain.com with Let's Encrypt."
    * In case of an failure most likely the DNS is not set up correctly. Please check that the DNS CNAME record is correct and can be successfully resolved. Sometimes DNS update might take some time to take effect.
    * After you corrected the issue, please repeat to run the above command.
* Check if everything is working fine:
    * `docker exec -it terra4d-webtracker_webtracker_1 /bin/bash`
    * `systemctl status`
        ▪ This should report system is running normally, if some service is not running use:
        ▪ `/usr/bin/restart-webtracker`

Restart the webtracker container:
* Run `eval $(docker-machine env <machine_name>)` in console
* Go to the “terra4d-webtracker” directory (with the docker-compose.yml file)
* `source aws_creds`
* `docker-compose down`
* `docker-compose up -d`

Restart/Shutdown the docker machine:
* Run `eval $(docker-machine env <machine_name>)` in console
* To restart the machine, run `docker-machine restart`
* To shutdown the machine, run `docker-machine stop`
* To start a previously stopped machine, run `docker-machine start`

