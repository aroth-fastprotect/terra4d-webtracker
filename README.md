# Terra4D Webtracker setup on AWS

## Installation
* Get/create AWS account
* Get AWS access key from AWS console using AWS IAM service.
    * Create new group: docker-machine, with “AmazonEC2FullAccess” permissions
    * Create new user: webtracker, assign to group “docker-machine”
    * Goto user “webtracker”, tab “Security credentials”, Create Access key
    * Download access key and secret and store in a safe place
* Install/Download “Docker for Windows” or “Install Docker on Ubuntu” using the [official guide](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
* For the remaining instructions we are using an Ubuntu machine. The process is **not** identical on a Windows machine, but should be quite similar.
* Go to https://github.com/aroth-fastprotect/terra4d-webtracker
    * Either use “git clone” or download and unzip the ZIP Package from the GitHub page
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
    * Select the new instance `<machine_name>` and copy the `Public DNS` entry; e.g. ec2-15-135-214-16.eu-central-1.compute.amazonaws.com
* Go to the DNS management tool for your domain
    * Create a DNS-Record of type CNAME for "myhost.mydomain.com." and set the target to the `Public DNS` entry copied from the AWS console.
    * Check if the DNS record is working and can be correctly resolved.
    * Running `host myhost.mydomain.com.` on Linux should return for example:
        ```
        myhost.mydomain.com. is an alias for ec2-15-135-214-16.eu-central-1.compute.amazonaws.com
        ec2-15-135-214-16.eu-central-1.compute.amazonaws.com has address 15.135.214.16
        ```
* Start webtracker docker on AWS docker machine:
    * `docker-compose up -d`
    * This outputs the name of the newly created container; e.g. `terra4d-webtracker_webtracker_1`

## Check webtracker service container
* Run `eval $(docker-machine env <machine_name>)` in console
* Go to the “terra4d-webtracker” directory (with the docker-compose.yml file)
* Run `source aws_creds`
* Run `docker exec -it terra4d-webtracker_webtracker_1 /bin/bash` to get a SSH console session on the new machine

### Overall status
* Run `systemctl status` to get the overall system status.
* `systemctl` shows the list of all services and their status. If the `webtracker-first-start.service` service fails the SSL certificate for the configured
    hostname and domain cannot be verified by Let's Encrypt and the machine uses a self-signed certificate.

### Let's Encrypt issues
* Check the Let's Encrypt log file; run `cat /var/log/letsencrypt/letsencrypt.log`
    * The log file contains the data exchanged between the Let's Encrypt server and the docker container. Examine the log file and check
      the manual for Let's Encrypt or Certbot for guidance.
    * If certbot fails to register the docker on Let's Encrypt service the DNS name is not working and you get an error when accessing the machine via HTTPs.
    * After you corrected the issue, please repeat to run `systemctl restart webtracker-first-start.service`. This will repeat the registeration process. If the registeration was previously successful,
    the script refuses register again.

### Restart webtracker services
* Run `systemctl status` to get the overall system status.
* This should report system is running normally, if some service is not running use:
* `/usr/bin/restart-webtracker`

### Get version
* Visit the web page `https://<myhost.mydomain.com>/` returns a web page with the version of the webtracker service.
* Use `https://<myhost.mydomain.com>/version` to retrieve technical version information (as JSON).

## Update webtracker container
* Run `eval $(docker-machine env <machine_name>)` in console
* Go to the “terra4d-webtracker” directory (with the docker-compose.yml file)
* Run `source aws_creds`
* Run `docker-compose pull`
* Run `docker-compose restart`

## Restart the webtracker container
* Run `eval $(docker-machine env <machine_name>)` in console
* Go to the “terra4d-webtracker” directory (with the docker-compose.yml file)
* Run `source aws_creds`
* `docker-compose restart`

## Stop the webtracker container
* Run `eval $(docker-machine env <machine_name>)` in console
* Go to the “terra4d-webtracker” directory (with the docker-compose.yml file)
* Run `source aws_creds`
* Run `docker-compose down`

## Restart/Shutdown the docker machine
* Run `eval $(docker-machine env <machine_name>)` in console
* Go to the “terra4d-webtracker” directory (with the docker-compose.yml file)
* Run `source aws_creds`
* To restart the machine, run `docker-machine restart`
* To shutdown the machine, run `docker-machine stop`
* To start a previously stopped machine, run `docker-machine start`
* Restarting or stopping the complete machine results in loss of the IP address. After restarting the machine it gets a new IP
  address and therefore the DNS CNAME record must be updated.

